# Experimento — Fases 1 y 2 de ataque (Entregable 4)

> Documento que combina el enunciado de la práctica con las capturas de pantalla del alumno, transcritas como texto. Debajo de cada bloque de código se explica el significado de los comandos utilizados.
>
> **Datos del entorno (deducidos de las capturas):**
> - Torrent-Auditor (Ubuntu): prompt `linux@SAD-Ubuntu`
> - Torrent-Vulnerable (Metasploitable): prompt `msfadmin@metasploitable`, IP **192.168.1.20**

---

## Parte A — Fase 1: reconocimiento

### A1. `whois` y `nslookup` desde dos máquinas (≈15 min)

Este punto lo hemos trabajado sobre un escenario ficticio debido a las limitaciones de red que nos impone la GVA. En casa podéis probar a hacerlo sobre dominios reales.

**Respuesta:** no hay capturas ni anotaciones de este apartado en el documento original.

---

## Parte B — Fase 2: exploración y contramedidas

### B1. Preparación

- Arrancad Torrent-Auditor y Torrent-Vulnerable.
- Antes de tocar nada, haced una instantánea de Torrent-Vulnerable (en VirtualBox: menú *Máquina → Tomar instantánea*). Vais a modificarla y luego la necesitaremos tal como estaba.
- Anotad la IP de Torrent-Vulnerable (`ip a` dentro de ella).
- Desde Torrent-Auditor, comprobad que la veis: `ping -c 3 <IP>`.

**Respuesta:** no hay capturas de la instantánea, de `ip a` ni del `ping`. Por las capturas posteriores, la IP de Torrent-Vulnerable es `192.168.1.20`.

---

### B2. Medición ANTES

En Torrent-Auditor:

```bash
sudo nmap -sV <IP> -oN antes.txt
```

Apuntad:
- Cuántos puertos abiertos hay.
- Qué servicios y versiones aparecen.
- Si el ping responde.

**Respuesta (captura "ANTES"):**

```text
linux@SAD-Ubuntu:~$ sudo nmap -sV 192.168.1.20 -oN antes.txt
[sudo] password for linux:
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-10-09 11:23 UTC
Nmap scan report for 192.168.1.20
Host is up (0.00043s latency).
Not shown: 978 closed tcp ports (reset)
PORT      STATE SERVICE     VERSION
21/tcp    open  ftp         vsftpd 2.3.4
22/tcp    open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
23/tcp    open  telnet      Linux telnetd
25/tcp    open  smtp        Postfix smtpd
53/tcp    open  domain      ISC BIND 9.4.2
80/tcp    open  http        Apache httpd 2.2.8 ((Ubuntu) DAV/2)
111/tcp   open  rpcbind     2 (RPC #100000)
139/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
512/tcp   open  exec        netkit-rsh rexecd
514/tcp   open  shell       Netkit rshd
1099/tcp  open  java-rmi    GNU Classpath grmiregistry
1524/tcp  open  bindshell   Metasploitable root shell
2049/tcp  open  nfs         2-4 (RPC #100003)
2121/tcp  open  ftp         ProFTPD 1.3.1
3306/tcp  open  mysql       MySQL 5.0.51a-3ubuntu5
5432/tcp  open  postgresql  PostgreSQL DB 8.3.0 - 8.3.7
5900/tcp  open  vnc         VNC (protocol 3.3)
6000/tcp  open  X11         (access denied)
6667/tcp  open  irc         UnrealIRCd
8009/tcp  open  ajp13       Apache Jserv (Protocol v1.3)
8180/tcp  open  http        Apache Tomcat/Coyote JSP engine 1.1
MAC Address: 08:00:27:4D:FD:0D (Oracle VirtualBox virtual NIC)
Service Info: Hosts:  metasploitable.localdomain, irc.Metasploitable.LAN; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 22.31 seconds
```

**Significado de los comandos:**

- `sudo` → ejecuta el comando con privilegios de administrador (nmap los necesita para ciertos tipos de escaneo).
- `nmap` → herramienta de exploración de red que descubre equipos y puertos abiertos.
- `-sV` → *Service Version detection*: no se fía del número de puerto; se conecta a cada puerto abierto y analiza la respuesta para identificar el servicio y su versión.
- `192.168.1.20` → IP del objetivo (Torrent-Vulnerable).
- `-oN antes.txt` → guarda el resultado en formato "normal" (legible) en el fichero `antes.txt`, para compararlo después con `diff`.

**Anotaciones deducidas de la captura:**

- **Puertos abiertos:** 22 (los otros 978 de los 1000 escaneados están cerrados).
- **Servicios y versiones:** los de la tabla anterior (vsftpd 2.3.4, OpenSSH 4.7p1, Apache 2.2.8, Samba 3.X–4.X, ProFTPD 1.3.1, MySQL 5.0.51a, PostgreSQL 8.3.x, UnrealIRCd, Tomcat, etc.).
- **Respuesta al ping:** `Host is up` indica que el equipo responde, pero no hay captura del `ping -c 3` en el documento original.

---

### B3. Contramedidas (en Torrent-Vulnerable, con sudo)

Usuario `msfadmin`, contraseña `msfadmin`.

**Suposición de trabajo:** este servidor solo tiene que ofrecer servicio web (puerto 80) y administración remota por SSH (puerto 22). Todo lo demás sobra.

#### Contramedida 1 — Inventariar y apagar servicios innecesarios

**1. Listad lo que escucha:** `sudo netstat -tulpn`

**Respuesta (captura "Procesos y comandos"; la captura original aparece recortada, solo se ven estas líneas):**

```text
msfadmin@metasploitable:~$ sudo netstat -tulpn
[sudo] password for msfadmin:
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address       Foreign Address     State       PID/Program name
tcp        0      0 0.0.0.0:45344       0.0.0.0:*           LISTEN      4601/rmiregistry
tcp        0      0 0.0.0.0:512         0.0.0.0:*           LISTEN      4465/xinetd
tcp        0      0 0.0.0.0:2049        0.0.0.0:*           LISTEN      -
tcp        0      0 0.0.0.0:514         0.0.0.0:*           LISTEN      4465/xinetd
tcp        0      0 0.0.0.0:8009        0.0.0.0:*           LISTEN      4564/jsvc
tcp        0      0 0.0.0.0:6697        0.0.0.0:*           LISTEN      4613/unrealircd
tcp        0      0 0.0.0.0:3306        0.0.0.0:*           LISTEN      4206/mysqld
tcp        0      0 0.0.0.0:1099        0.0.0.0:*           LISTEN      4601/rmiregistry
tcp        0      0 0.0.0.0:6667        0.0.0.0:*           LISTEN      4613/unrealircd
tcp        0      0 0.0.0.0:139         0.0.0.0:*           LISTEN      4449/smbd
```

**Significado de los comandos:**

- `netstat` → muestra conexiones de red y puertos en escucha.
- `-t` → incluye sockets TCP.
- `-u` → incluye sockets UDP.
- `-l` → muestra solo los sockets en escucha (*listening*).
- `-p` → muestra el PID y el nombre del programa propietario de cada socket.
- `-n` → muestra direcciones y puertos en formato numérico, sin resolver nombres.
- `0.0.0.0:<puerto>` → el servicio escucha en todas las interfaces de red.
- Detalle importante: los puertos **512 y 514** aparecen a nombre de **`xinetd`** (el "portero" que arranca servicios a demanda), no del servicio real.

---

**¿Cómo sé qué servicio hay detrás de un número de puerto?** No os fiéis de una sola fuente: no todas las máquinas traen los mismos ficheros completos. Tres caminos, de más a menos fiable:

**1. Preguntárselo al servicio.** Desde la Torrent-Auditor: `sudo nmap -sV -p 512 IP` (cambiad el 512 por el puerto que miréis). Con `-sV`, nmap no se fía del número: se conecta, analiza cómo responde y lo compara con su propia base de datos, así que funciona aunque la máquina que escaneáis no tenga esa información. Es lo mismo que ya hicisteis en la medición "antes".

**Respuesta (a): consulta del puerto 22**

```text
linux@SAD-Ubuntu:~$ sudo nmap -sV -p 22 192.168.1.20
[sudo] password for linux:
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-10-09 15:28 UTC
Nmap scan report for 192.168.1.20
Host is up (0.00070s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
MAC Address: 08:00:27:4D:FD:0D (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 0.57 seconds
```

**Significado de los comandos:**

- `nmap -sV` → detección de servicio y versión.
- `-p 22` → limita el escaneo únicamente al puerto 22.
- Resultado: el puerto 22 es **SSH**, programa **OpenSSH 4.7p1**.

**Respuesta (b): consulta del puerto 512**

```text
linux@SAD-Ubuntu:~$ sudo nmap -sV 512 192.168.1.20
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-10-11 09:06 UTC
Nmap scan report for 192.168.1.20
Host is up (0.00044s latency).
Not shown: 978 closed tcp ports (reset)
PORT      STATE SERVICE     VERSION
21/tcp    open  ftp         vsftpd 2.3.4
22/tcp    open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
23/tcp    open  telnet      Linux telnetd
25/tcp    open  smtp        Postfix smtpd
53/tcp    open  domain      ISC BIND 9.4.2
80/tcp    open  http        Apache httpd 2.2.8 ((Ubuntu) DAV/2)
111/tcp   open  rpcbind     2 (RPC #100000)
139/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
512/tcp   open  exec        netkit-rsh rexecd
514/tcp   open  shell       Netkit rshd
1099/tcp  open  java-rmi    GNU Classpath grmiregistry
1524/tcp  open  bindshell   Metasploitable root shell
2049/tcp  open  nfs         2-4 (RPC #100003)
2121/tcp  open  ftp         ProFTPD 1.3.1
3306/tcp  open  mysql       MySQL 5.0.51a-3ubuntu5
5432/tcp  open  postgresql  PostgreSQL DB 8.3.0 - 8.3.7
5900/tcp  open  vnc         VNC (protocol 3.3)
6000/tcp  open  X11         (access denied)
6667/tcp  open  irc         UnrealIRCd
8009/tcp  open  ajp13       Apache Jserv (Protocol v1.3)
8180/tcp  open  http        Apache Tomcat/Coyote JSP engine 1.1
MAC Address: 08:00:27:4D:FD:0D (Oracle VirtualBox virtual NIC)
Service Info: Hosts:  metasploitable.localdomain, irc.Metasploitable.LAN; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
```

**Significado de los comandos:**

- `sudo nmap -sV 512 192.168.1.20` → en este comando **falta la opción `-p`** delante del 512, por lo que nmap no limita el escaneo al puerto 512 y escanea los 1000 puertos habituales de la IP. La forma correcta es `sudo nmap -sV -p 512 192.168.1.20`.
- Aun así, la salida sirve para identificar el puerto: **512/tcp → `exec` → `netkit-rsh rexecd`**.

---

**2. Mirar lo que sirve el portero** (cuando el dueño del puerto es `inetd` o `xinetd`). No hace falta partir del puerto: listad todo lo que sirve y buscad ahí.

- `grep -v '^#' /etc/inetd.conf` (las líneas activas; la primera palabra es el servicio y la última, el programa).
- `grep -H disable /etc/xinetd.d/*` y, dentro de cada fichero, las líneas `service` y `server =`.
- Y por si el puerto está escrito a mano: `sudo grep -rn -w 512 /etc/inetd.conf /etc/xinetd.conf /etc/xinetd.d 2>/dev/null`

**Respuesta (a): líneas activas de `inetd.conf`**

```text
msfadmin@metasploitable:~$ grep -v '^#' /etc/inetd.conf
telnet          stream  tcp     nowait  telnetd /usr/sbin/tcpd  /usr/sbin/in.telnetd
tftp            dgram   udp     wait    nobody  /usr/sbin/tcpd  /usr/sbin/in.tftpd /srv/tftp
shell           stream  tcp     nowait  root    /usr/sbin/tcpd  /usr/sbin/in.rshd
exec            stream  tcp     nowait  root    /usr/sbin/tcpd  /usr/sbin/in.rexecd
ingreslock stream tcp nowait root /bin/bash bash -i
```

**Significado de los comandos:**

- `grep` → busca líneas que coincidan con un patrón en un fichero.
- `-v` → invierte la búsqueda: muestra las líneas que **no** coinciden.
- `'^#'` → expresión regular: líneas que empiezan por `#` (comentarios).
- Conjunto: muestra solo las líneas **activas** (no comentadas) de `/etc/inetd.conf`.
- Columnas de cada línea: servicio · tipo de socket · protocolo · modo de espera · usuario · programa (`tcpd`, el envoltorio) · programa final.
- Observación: `ingreslock` ejecuta directamente `/bin/bash -i` como root, es decir, **una shell de root abierta a la red** (es la puerta trasera que nmap detecta como `bindshell` en el puerto 1524).

**Respuesta (b): estado de los servicios de `xinetd.d`**

```text
msfadmin@metasploitable:~$ grep -H disable /etc/xinetd.d/*
/etc/xinetd.d/chargen:  disable         = yes
/etc/xinetd.d/chargen:  disable         = yes
/etc/xinetd.d/daytime:  disable         = yes
/etc/xinetd.d/daytime:  disable         = yes
/etc/xinetd.d/discard:  disable         = yes
/etc/xinetd.d/discard:  disable         = yes
/etc/xinetd.d/echo:     disable         = yes
/etc/xinetd.d/echo:     disable         = yes
/etc/xinetd.d/time:     disable         = yes
/etc/xinetd.d/time:     disable         = yes
/etc/xinetd.d/vsftpd:   disable         = no
```

**Significado de los comandos:**

- `grep -H disable /etc/xinetd.d/*` → busca la palabra `disable` en todos los ficheros de `/etc/xinetd.d/`.
- `-H` → muestra el nombre del fichero delante de cada línea encontrada.
- `/etc/xinetd.d/*` → todos los ficheros de configuración de servicios de `xinetd`.
- `disable = yes` → el servicio está **desactivado**; `disable = no` → el servicio está **activo** (aquí, `vsftpd`).

**Respuesta (c): búsqueda de un puerto escrito "a mano"**

```text
msfadmin@metasploitable:~$ sudo grep -rn -w 22 /etc/inetd.com /etc/xinetd.conf /etc/xinetd.d 2>/dev/null
/etc/xinetd.d/daytime:3:# then prints it out in a format like this: "Wed Nov 13 22:30:27 EST 2002".
```

**Significado de los comandos:**

- `grep -rn -w 22 ...` → busca recursivamente (`-r`) la palabra completa (`-w`) `22`, mostrando el número de línea (`-n`).
- `2>/dev/null` → descarta los mensajes de error (por ejemplo, ficheros que no existen).
- Resultado: la única coincidencia es un **falso positivo** (el "22" de la hora `22:30:27` en un comentario de `daytime`), así que el puerto 22 no está definido en `inetd`/`xinetd`; lo sirve un proceso propio (OpenSSH).
- Nota: en el comando aparece `/etc/inetd.com` en lugar de `/etc/inetd.conf`. Al estar `2>/dev/null`, el error por fichero inexistente quedó oculto. La ruta correcta es `/etc/inetd.conf`.

---

**3. La agenda del sistema.** `grep -w 512/tcp /etc/services` empareja números con nombres. Es cómoda cuando existe la entrada, pero puede no estar (o estar recortada o modificada): si no sale nada, eso no significa que el puerto no sea de nadie. Seguid por el camino 1 o el 2.

**Respuesta:** no hay captura de este camino en el documento original.

---

**Cuidado:** el número de puerto es solo una costumbre. Nada impide que un servicio use un puerto que no le corresponde, y así se esconden muchas puertas traseras. Una agenda os dice lo que suele haber; `nmap -sV` os dice lo que hay.

**El nombre no es el programa.** Aunque sepáis que el 512 es `exec`, falta llegar hasta el programa concreto. La cadena completa:

| Eslabón | Cómo averiguarlo |
|---|---|
| Puerto → nombre | `nmap -sV` (o `/etc/services`, si tiene la entrada) |
| Nombre → programa | En la configuración del portero: en `inetd.conf`, la última columna; en `xinetd`, la línea `server =` |
| Programa → paquete | `dpkg -S /ruta/del/programa` |

**Respuesta (cadena aplicada a los puertos 512 y 514, según las capturas):**

| Puerto | Nombre (nmap -sV) | Programa (`inetd.conf`) |
|---|---|---|
| 512/tcp | `exec` (netkit-rsh rexecd) | `/usr/sbin/in.rexecd` |
| 514/tcp | `shell` (Netkit rshd) | `/usr/sbin/in.rshd` |

El paso "programa → paquete" (`dpkg -S`) no aparece en el documento original.

---

**Haced una tabla:** puerto, servicio, ¿es necesario según la suposición de trabajo?, decisión.

**Respuesta:** el documento original no incluye esta tabla completa. Con la información de las capturas, solo se pueden rellenar los dos servicios tratados:

| Puerto | Servicio | ¿Necesario? (solo 80 y 22) | Decisión |
|---|---|---|---|
| 2121 | ProFTPD 1.3.1 | No | Parar y quitar del arranque (Servicio 1) |
| 513 | login (`in.rlogind`) | No | Comentar en `inetd.conf` (Servicio 2) |
| *(resto de puertos)* | — | — | *Sin documentar* |

**Ojo — servicios "a demanda".** Normalmente un servicio (como Apache) es un proceso que arranca con el sistema y se queda esperando conexiones. Pero algunos no: los gestiona un programa "portero" (`inetd` o `xinetd`) que vigila los puertos de varios servicios a la vez y solo arranca el servicio cuando alguien se conecta, y lo cierra al terminar. Por eso un puerto puede aparecer abierto en nmap sin que veáis ningún proceso del servicio con `ps`, y en `netstat` figurará a nombre de `inetd` o `xinetd`. Para saber qué hay detrás de uno de esos puertos, mirad su archivo de configuración (`/etc/inetd.conf` o la carpeta `/etc/xinetd.d/`). Esto no es lo mismo que los scripts de `/etc/init.d/`, que arrancan los servicios permanentes.

**Apagad los servicios que sobran.** El profesor os dará el método para cada uno (unos se paran con su script de `/etc/init.d/`, otros se desactivan en un archivo de configuración).

##### Servicio 1 — ProFTPD (puerto 2121), método: script de `/etc/init.d/`

```text
msfadmin@metasploitable:~$ sudo /etc/init.d/proftpd stop
 * Stopping ftp server proftpd
msfadmin@metasploitable:~$ sudo netstat -tulpn | grep 2121
msfadmin@metasploitable:~$ sudo update-rc.d -f proftpd remove
 Removing any system startup links for /etc/init.d/proftpd ...
   /etc/rc0.d/K50proftpd
   /etc/rc1.d/K50proftpd
   /etc/rc2.d/S50proftpd
   /etc/rc3.d/S50proftpd
   /etc/rc4.d/S50proftpd
   /etc/rc5.d/S50proftpd
   /etc/rc6.d/K50proftpd
```

**Significado de los comandos:**

- `sudo /etc/init.d/proftpd stop` → ejecuta el script del servicio con el argumento `stop`: detiene ProFTPD en ese momento.
- `sudo netstat -tulpn | grep 2121` → lista los puertos en escucha y filtra (`|` + `grep`) por el 2121. **No devuelve nada**: el puerto ya no está en escucha.
- `sudo update-rc.d -f proftpd remove` → elimina los enlaces de arranque del servicio en los niveles de ejecución (`/etc/rcN.d/`), para que **no vuelva a arrancar** al reiniciar. `-f` fuerza la operación aunque el script siga existiendo. Los enlaces `K50…` eran de parada (*Kill*) y los `S50…` de arranque (*Start*).

Comprobación desde Torrent-Auditor:

```text
linux@SAD-Ubuntu:~$ sudo nmap -sV -p 2121 192.168.1.20
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-10-13 14:44 UTC
Nmap scan report for 192.168.1.20
Host is up (0.0020s latency).

PORT     STATE  SERVICE     VERSION
2121/tcp closed ccproxy-ftp
MAC Address: 08:00:27:4D:FD:0D (Oracle VirtualBox virtual NIC)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 0.54 seconds
```

**Significado de los comandos:**

- `sudo nmap -sV -p 2121 192.168.1.20` → vuelve a escanear solo el puerto 2121 para verificar el cambio.
- Resultado: `closed` → el puerto ya está cerrado. El nombre `ccproxy-ftp` es solo la etiqueta del puerto según la base de datos de nmap (la "agenda"); no hay ningún servicio detrás.

Nota del alumno: *(después, hacer `sudo reboot`)*

- `sudo reboot` → reinicia el sistema para comprobar que el servicio no arranca de nuevo.

##### Servicio 2 — `login` (rlogind, puerto 513), método: archivo de configuración de `inetd`

```text
msfadmin@metasploitable:~$ sudo grep -n -w login \
> /etc/inetd.conf /etc/xinetd.conf \
> /etc/xinetd.d/* 2>/dev/null
/etc/inetd.conf:6:#login          stream  tcp     nowait  root    /usr/sbin/tcpd  /usr/sbin/in.rlogind
msfadmin@metasploitable:~$
```

(Nota del alumno: *Está en `inetd.conf`*)

**Significado de los comandos:**

- `sudo grep -n -w login ... ` → busca la palabra completa `login` (`-w`) mostrando el número de línea (`-n`).
- `\` al final de línea → continúa el comando en la línea siguiente (por eso aparece `>`).
- `/etc/inetd.conf /etc/xinetd.conf /etc/xinetd.d/*` → los tres sitios donde puede estar definido un servicio gestionado por el portero.
- `2>/dev/null` → oculta los errores.
- Resultado: el servicio `login` está en la **línea 6** de `/etc/inetd.conf`, y su programa es `/usr/sbin/in.rlogind`.

```text
msfadmin@metasploitable:~$ sudo cp /etc/inetd.conf /root/inetd.conf.bak
msfadmin@metasploitable:~$ sudo sed -i 's/^login/#login/' /etc/inetd.conf
msfadmin@metasploitable:~$ sudo /etc/init.d/xinetd reload
 * Reloading internet superserver configuration xinetd                   [ OK ]
msfadmin@metasploitable:~$
```

**Significado de los comandos:**

- `sudo cp /etc/inetd.conf /root/inetd.conf.bak` → hace una **copia de seguridad** del fichero de configuración antes de tocarlo.
- `sudo sed -i 's/^login/#login/' /etc/inetd.conf` → edita el fichero "en el sitio" (`-i`): sustituye (`s/…/…/`) las líneas que empiezan por `login` (`^login`) por `#login`, es decir, **las comenta** y las desactiva.
- `sudo /etc/init.d/xinetd reload` → pide al portero que **recargue su configuración** sin reiniciar, para que aplique el cambio.

Comprobación desde Torrent-Auditor:

```text
linux@SAD-Ubuntu:~$ sudo nmap -sV -p 513 192.168.1.20
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-10-16 16:15 UTC
Nmap scan report for 192.168.1.20
Host is up (0.0028s latency).

PORT    STATE  SERVICE VERSION
513/tcp closed login
MAC Address: 08:00:27:4D:FD:0D (Oracle VirtualBox virtual NIC)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 0.39 seconds
linux@SAD-Ubuntu:~$
```

**Significado de los comandos:**

- `sudo nmap -sV -p 513 192.168.1.20` → escanea solo el puerto 513 (el de `login`/rlogin).
- Resultado: `closed` → el puerto está cerrado.

---

#### Contramedida 2 — Bloquear el ping (obligatoria)

```bash
sudo iptables -A INPUT -p icmp --icmp-type echo-request -j DROP
```

Comprobad desde Torrent-Auditor: `ping -c 3 <IP>`.

**Respuesta:** el documento original no incluye capturas de esta contramedida.

**Ampliación (si os da tiempo):**

- **DROP frente a REJECT.** Bloquead un puerto de dos formas distintas y mirad cómo lo describe nmap cada vez (borrad la regla anterior con `-D` antes de probar la otra):
  `sudo iptables -A INPUT -p tcp --dport 23 -j DROP`
  `sudo iptables -A INPUT -p tcp --dport 23 -j REJECT --reject-with tcp-reset`
- **Registrar los escaneos.** `sudo iptables -I INPUT 1 -p tcp --syn -m limit --limit 5/second -j LOG --log-prefix "POSIBLE-ESCANEO: "`. Después de un escaneo desde Torrent-Auditor, mirad el registro: `dmesg | grep POSIBLE-ESCANEO | tail`
- **Ocultar banners.** El profesor os indicará cómo cambiar el mensaje de versión de Apache o de vsftpd. Después repetid `nmap -sV` sobre ese puerto.

**Respuesta:** el documento original no incluye capturas de la ampliación.

---

### B4. Medición DESPUÉS

En Torrent-Auditor:

```bash
sudo nmap -sV <IP> -oN despues.txt
diff antes.txt despues.txt
```

Rellenad la tabla:

| Aspecto | Antes | Contramedida aplicada | Después |
|---|---|---|---|
| Puertos abiertos | | | |
| Servicios/versiones visibles | | | |
| Respuesta al ping | | | |

**Respuesta (captura "DESPUÉS"):**

```text
linux@SAD-Ubuntu:~$ sudo nmap -sV 192.168.1.20 -oN despues.txt
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-10-16 16:16 UTC
Nmap scan report for 192.168.1.20
Host is up (0.00071s latency).
Not shown: 979 closed tcp ports (reset)
PORT      STATE SERVICE     VERSION
21/tcp    open  ftp         vsftpd 2.3.4
22/tcp    open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
23/tcp    open  telnet      Linux telnetd
25/tcp    open  smtp        Postfix smtpd
53/tcp    open  domain      ISC BIND 9.4.2
80/tcp    open  http        Apache httpd 2.2.8 ((Ubuntu) DAV/2)
111/tcp   open  rpcbind     2 (RPC #100000)
139/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
512/tcp   open  exec        netkit-rsh rexecd
514/tcp   open  shell       Netkit rshd
1099/tcp  open  java-rmi    GNU Classpath grmiregistry
1524/tcp  open  bindshell   Metasploitable root shell
2049/tcp  open  nfs         2-4 (RPC #100003)
3306/tcp  open  mysql       MySQL 5.0.51a-3ubuntu5
5432/tcp  open  postgresql  PostgreSQL DB 8.3.0 - 8.3.7
5900/tcp  open  vnc         VNC (protocol 3.3)
6000/tcp  open  X11         (access denied)
6667/tcp  open  irc         UnrealIRCd
8009/tcp  open  ajp13       Apache Jserv (Protocol v1.3)
8180/tcp  open  http        Apache Tomcat/Coyote JSP engine 1.1
MAC Address: 08:00:27:4D:FD:0D (Oracle VirtualBox virtual NIC)
Service Info: Hosts:  metasploitable.localdomain, irc.Metasploitable.LAN; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 21.96 seconds
linux@SAD-Ubuntu:~$
```

Nota del alumno: *(Los servicios Proftpd y Apache Jserv no están en la lista)*

**Significado de los comandos:**

- `sudo nmap -sV 192.168.1.20 -oN despues.txt` → repite exactamente el escaneo de la medición "antes", guardando el resultado en `despues.txt`.
- `diff antes.txt despues.txt` → compara línea a línea ambos ficheros y muestra las diferencias (no hay captura en el documento original).

**Tabla rellenada con los datos de las capturas:**

| Aspecto | Antes | Contramedida aplicada | Después |
|---|---|---|---|
| Puertos abiertos | 22 | Parar ProFTPD (2121) y quitarlo del arranque | 21 (desaparece el 2121) |
| Servicios/versiones visibles | vsftpd 2.3.4, OpenSSH 4.7p1, Apache 2.2.8, Samba 3.X–4.X, ProFTPD 1.3.1, MySQL 5.0.51a, PostgreSQL 8.3.x, UnrealIRCd, Tomcat 1.1, etc. | Igual que arriba; el servicio `login` (513) ya estaba cerrado | Los mismos salvo ProFTPD 1.3.1 |
| Respuesta al ping | *(no documentado)* | *(no documentada; la regla de `iptables` no aparece)* | *(no documentado)* |

---

### B5. Preguntas finales

1. ¿Qué contramedida ha reducido más lo que ve el atacante? ¿Por qué?
2. Si solo ocultáis un banner pero el servicio sigue activo, ¿la vulnerabilidad sigue ahí? Razonad la respuesta.
3. Torrent-Vulnerable usa un sistema operativo sin soporte desde hace años. ¿Puede una contramedida de estas sustituir a actualizarlo? ¿Qué haríais en una empresa real?
4. De todas las contramedidas de las partes A y B, ¿cuáles evitan que el atacante encuentre información y cuáles solo ayudan a detectar que lo están intentando? (Aún no las hemos aplicado todas, pero pensad en cuáles serían de cada tipo.)

**Respuesta:** el documento original no incluye respuestas a estas preguntas.

---

## Observaciones sobre la revisión del documento

Al combinar ambos documentos aparecen algunos puntos que conviene revisar antes de entregar:

1. **Apache Jserv sigue en la lista "DESPUÉS".** La nota del alumno dice que ProFTPD y Apache Jserv ya no aparecen, pero la captura sigue mostrando `8009/tcp open ajp13 Apache Jserv (Protocol v1.3)`. Solo ha desaparecido el puerto 2121 (ProFTPD), y no hay captura del apagado de Jserv (en `netstat` el 8009 pertenece a `jsvc`).
2. **El puerto 513 (`login`) no estaba abierto en el "ANTES".** En `inetd.conf` la línea `#login` ya aparecía comentada al buscarla con `grep`, así que el `sed` no cambió nada. El número de puertos abiertos solo bajó de 22 a 21 por ProFTPD.
3. **Comando `nmap -sV 512 …` sin `-p`.** Escaneó todos los puertos por defecto en lugar de solo el 512.
4. **Ruta mal escrita:** `/etc/inetd.com` en lugar de `/etc/inetd.conf` en el `grep -rn -w 22`.
5. **Faltan evidencias** de: A1, B1 (instantánea, `ip a`, `ping`), ping antes/después, Contramedida 2 y ampliación, `diff antes.txt despues.txt`, la tabla completa de puertos/decisiones y las preguntas B5.
6. **Servicios todavía expuestos** (relevante para la tabla de decisiones): con la suposición de trabajo (solo 80 y 22), siguen abiertos, entre otros, 21, 23, 25, 53, 111, 139, 445, 512, 514, 1099, 1524, 2049, 3306, 5432, 5900, 6000, 6667, 8009 y 8180.
