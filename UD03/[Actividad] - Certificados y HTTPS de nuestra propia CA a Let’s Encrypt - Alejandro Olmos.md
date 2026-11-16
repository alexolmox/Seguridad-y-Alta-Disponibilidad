# Certificados y HTTPS: de nuestra propia CA a Let's Encrypt

## 0.1 Antes de empezar: qué necesitas

- Una máquina virtual Linux de servidor (Debian o Ubuntu Server) con usuario con permisos de sudo. Puedes usar Torrent-Auditor.
- Salida a Internet desde la VM, para instalar paquetes y, en la Parte 2, hablar con Let's Encrypt y DuckDNS.
- Poder llegar desde el navegador del equipo a la VM.

Averigua qué IP usa tu equipo para llegar a la VM y compruébala. A partir de ahora la llamaremos `IP_SERVIDOR` (cámbiala por la tuya en todos los comandos):

```console
linux@SAD-Ubuntu:~$ ip -br -4 a
lo               UNKNOWN        127.0.0.1/8
enp0s3           UP             10.0.2.15/24
enp0s8           UP             192.168.56.102/24
linux@SAD-Ubuntu:~$ ping -c 2 1.1.1.1
PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.
```

```console
linux@SAD-Ubuntu:~$ ping -c 2 10.0.2.15
PING 10.0.2.15 (10.0.2.15) 56(84) bytes of data.
64 bytes from 10.0.2.15: icmp_seq=1 ttl=64 time=3.07 ms
64 bytes from 10.0.2.15: icmp_seq=2 ttl=64 time=0.971 ms

--- 10.0.2.15 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1012ms
rtt min/avg/max/mdev = 0.971/2.019/3.067/1.048 ms
```

```console
linux@SAD-Ubuntu:~$ dig debian.org

;; ANSWER SECTION:
debian.org.     160     IN      A       151.101.66.132
debian.org.     160     IN      A       151.101.2.132
debian.org.     160     IN      A       151.101.130.132
debian.org.     160     IN      A       151.101.194.132

;; Query time: 37 msec
;; SERVER: 127.0.0.53#53(127.0.0.53) (UDP)
;; WHEN: Fri Nov 06 18:19:08 UTC 2026
;; MSG SIZE  rcvd: 103
```

---

## 1.2 Paso a paso

### Paso 0 — Instalar Apache y comprobar que funciona (todavía sin cifrar)

Instalamos el servidor web, la herramienta `openssl` con la que crearemos todo, `curl` para hacer pruebas y `cron` (lo necesitaremos en la Parte 2).

```console
linux@SAD-Ubuntu:~$ sudo apt update
linux@SAD-Ubuntu:~$ sudo apt install -y apache2 openssl curl cron
linux@SAD-Ubuntu:~$ systemctl status apache2 --no-pager
● apache2.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/apache2.service; enabled; preset: enabled)
     Active: active (running) since Fri 2026-11-06 18:38:53 UTC; 25s ago
linux@SAD-Ubuntu:~$ sudo ss -tlnp | grep ':80'
LISTEN 0   511   *:80   *:*   users:(("apache2",pid=15835,fd=4),("apache2",pid=15833,fd=4),("apache2",pid=15831,fd=4))
```

### Paso 1 — Crear una carpeta de trabajo para la CA

Separamos todo lo de la CA en su propia carpeta, lejos de la configuración de Apache. La subcarpeta `private` contendrá claves privadas, así que solo tu usuario puede entrar en ella (permisos 700).

```console
linux@SAD-Ubuntu:~$ mkdir -p ~/ca/private ~/ca/certs
linux@SAD-Ubuntu:~$ chmod 700 ~/ca/private
linux@SAD-Ubuntu:~$ cd ~/ca
linux@SAD-Ubuntu:~/ca$ ls -ld private
drwx------ 2 linux linux 4096 Nov  6 18:42 private
```

### Paso 2 — Generar la clave privada de la CA

`genrsa` crea una clave privada RSA; 4096 bits es una longitud generosa porque esta clave va a vivir mucho tiempo. La opción `-aes256` cifra el fichero con una contraseña (passphrase): si alguien copia `ca.key`, no puede usarla sin esa frase. Cuidado: si la olvidas no se puede recuperar.

```console
linux@SAD-Ubuntu:~/ca$ openssl genrsa -aes256 -out private/ca.key 4096
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
```

**✎ Anota en tu informe:** ¿por qué conviene proteger con contraseña la clave privada de una CA aunque esté en un directorio con permisos restrictivos?

> Porque los permisos a nivel de sistema operativo son vulnerables ante múltiples ataques. Si la clave se almacena en texto plano, la seguridad de toda la infraestructura de clave pública dependería de un único factor de protección.

### Paso 3 — Crear el certificado raíz de la CA (autofirmado)

Ahora la CA se presenta al mundo: crea su propio certificado y lo firma con su propia clave. Eso se llama certificado autofirmado: el emisor y el sujeto son la misma entidad. Es lo normal en una CA raíz, porque no hay nadie por encima que la avale.

```console
linux@SAD-Ubuntu:~/ca$ openssl req -x509 -new -key private/ca.key -sha256 -days 3650 -out certs/ca.crt -subj "/C=ES/O=Torrent S.L./CN=Torrent S.L. CA Raiz" -addext "basicConstraints=critical,CA:TRUE" -addext "keyUsage=critical,keyCertSign,cRLSign"
Enter pass phrase for private/ca.key:
```

### Paso 4 — Inspeccionar el certificado que acabas de crear

Un certificado es un documento legible. Comprueba que contiene lo que esperas.

```console
linux@SAD-Ubuntu:~/ca$ openssl x509 -in certs/ca.crt -noout -subject -issuer -dates
subject=C = ES, O = Torrent S.L., CN = Torrent S.L. CA Raiz
issuer=C = ES, O = Torrent S.L., CN = Torrent S.L. CA Raiz
linux@SAD-Ubuntu:~/ca$ openssl x509 -in certs/ca.crt -noout -text | grep -A1 'Basic Constraints'
            X509v3 Basic Constraints: critical
                CA:TRUE
```

```console
linux@SAD-Ubuntu:~/ca$ openssl x509 -in certs/ca.crt -noout -dates
notBefore=Nov  9 18:41:30 2026 GMT
notAfter=Nov  6 18:41:30 2036 GMT
```

**✎ Anota en tu informe:** ¿Por qué coinciden Subject e Issuer? ¿Hasta qué fecha es válida tu CA?

> - Porque es un certificado autofirmado (self-signed), como corresponde a una CA raíz. El Subject indica a quién pertenece el certificado y el Issuer indica quién lo firmó.
> - Su fecha de caducidad es el 6 de noviembre de 2036.

### Paso 5 — Clave privada del servidor y petición de certificado (CSR)

El servidor necesita su propio par de claves. A diferencia de la CA, esta clave va sin contraseña, porque Apache tiene que poder arrancar solo sin que nadie teclee nada. Lo compensamos protegiendo el fichero con permisos. Después creamos la CSR: es la petición que se le envía a la CA diciendo «aquí tienes mi clave pública y mi identidad, fírmame un certificado». La CSR nunca contiene la clave privada.

```console
linux@SAD-Ubuntu:~/ca$ openssl genrsa -out private/srv.key 2048
linux@SAD-Ubuntu:~/ca$ openssl req -new -key private/srv.key -out srv.csr -subj "/C=ES/O=Torrent S.L./CN=hmi.torrent.local"
```

### Paso 6 — Indicar para qué nombres e IP es válido el certificado (SAN)

Los navegadores modernos ya no miran el nombre (CN) del certificado: comprueban el campo Subject Alternative Name (SAN). Como vas a entrar por IP, esa IP tiene que figurar ahí. Crea el fichero de extensiones y sustituye `IP_SERVIDOR` por la tuya:

```console
linux@SAD-Ubuntu:~/ca$ nano srv.net
```

Contenido de `srv.net`:

```ini
subjectAltName = DNS:hmi.torrent.local, IP:192.168.56.102
basicConstraints = CA:FALSE
keyUsage = digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth
```

### Paso 7 — La CA firma el certificado del servidor

Este es el momento clave: la CA comprueba la petición y la firma con su clave privada. A partir de aquí, cualquiera que confíe en la CA puede confiar en el servidor. La opción `-CAcreateserial` asigna un número de serie único al certificado.

```console
linux@SAD-Ubuntu:~/ca$ openssl x509 -req -in srv.csr -CA certs/ca.crt -CAkey private/ca.key -CAcreateserial -out certs/srv.crt -days 365 -sha256
Certificate request self-signature ok
subject=C = ES, O = Torrent S.L., CN = hmi.torrent.local
Enter pass phrase for private/ca.key:
```

**✎ Anota en tu informe:** ¿Por qué el certificado del servidor dura 365 días y el de la CA 10 años?

> El certificado de la CA tiene una validez de 10 años porque sustituirlo exige redistribuirlo a todos los clientes, su clave privada se usa poco y está mejor protegida, y debe ser válido más tiempo que los certificados que firma. El certificado del servidor dura 365 días porque su clave está más expuesta, y una vida corta limita el periodo de riesgo si se compromete, obliga a renovar y actualizar la información periódicamente, y es sencillo de reemplazar sin afectar a otros servicios.

### Paso 8 — Verificar la cadena de confianza

`openssl` puede comprobar que el certificado del servidor está firmado por la CA indicada.

```console
linux@SAD-Ubuntu:~/ca$ openssl verify -CAfile certs/ca.crt certs/srv.crt
certs/srv.crt: OK
linux@SAD-Ubuntu:~/ca$ openssl x509 -in certs/srv.crt -noout -subject -issuer -ext subjectAltName
subject=C = ES, O = Torrent S.L., CN = hmi.torrent.local
issuer=C = ES, O = Torrent S.L., CN = Torrent S.L. CA Raiz
No extensions in certificate
```
