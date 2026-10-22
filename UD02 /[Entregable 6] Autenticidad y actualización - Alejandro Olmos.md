# Entregable 6 · Autenticidad y actualización — Alejandro Olmos

## Parte A · ¿Está actualizado el sistema?

### A1. Anotad qué versión de sistema operativo y kernel tenéis

```console
linux@SAD-Ubuntu:~$ lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 24.04.3 LTS
Release:        24.04
Codename:       noble
linux@SAD-Ubuntu:~$ uname -r
7.0.0-31-generic
linux@SAD-Ubuntu:~$
```

### A2. Pedid al sistema la lista de paquetes disponibles y qué se puede actualizar

```console
linux@SAD-Ubuntu:~$ sudo apt update
[sudo] password for linux:
Ign:1 http://es.archive.ubuntu.com/ubuntu noble InRelease
Ign:2 http://security.ubuntu.com/ubuntu noble-security InRelease
Ign:3 http://es.archive.ubuntu.com/ubuntu noble-updates InRelease
```

```console
linux@SAD-Ubuntu:~$ apt list --upgradable
Listing... Done
alsa-ucm-conf/noble-updates 1.2.10-1ubuntu5.15 all [upgradable from: 1.2.10-1ubuntu5.7]
apparmor/noble-updates 4.0.1really4.0.1-0ubuntu0.24.04.8 amd64 [upgradable from: 4.0.1really4.0.1-0ubuntu0.24.04.4]
apport-core-dump-handler/noble-updates 2.28.3-0ubuntu0.1 all [upgradable from: 2.28.1-0ubuntu3.8]
apport-gtk/noble-updates 2.28.3-0ubuntu0.1 all [upgradable from: 2.28.1-0ubuntu3.8]
apport/noble-updates 2.28.3-0ubuntu0.1 all [upgradable from: 2.28.1-0ubuntu3.8]
base-files/noble-updates 13ubuntu10.5 amd64 [upgradable from: 13ubuntu10.3]
```

```console
256 packages can be upgraded. Run 'apt list --upgradable' to see them.
```

### A3. Simulad la actualización sin aplicarla

```console
linux@SAD-Ubuntu:~$ sudo apt-get -s upgrade | head -30
Reading package lists...
Building dependency tree...
Reading state information...
Calculating upgrade...
The following upgrades have been deferred due to phasing:
  apparmor gnome-shell gnome-shell-common libapparmor1 snapd
The following packages have been kept back:
  fwupd linux-generic-hwe-24.04 linux-headers-generic
  linux-headers-generic-hwe-24.04 linux-image-generic-hwe-24.04
The following packages will be upgraded:
  alsa-ucm-conf apport apport-core-dump-handler apport-gtk base-files
  bind9-dnsutils bind9-host bind9-libs bluez bluez-cups bluez-obexd bubblewrap
  cloud-init console-setup console-setup-linux dconf-cli
  dconf-gsettings-backend dconf-service dhcpcd-base dmidecode dnsmasq-base
  dracut-install gdb gdm3 ghostscript gir1.2-gdm-1.0 gir1.2-glib-2.0
  gir1.2-gtk-4.0 gir1.2-mutter-14 gir1.2-nm-1.0 gir1.2-polkit-1.0
  gir1.2-rsvg-2.0 gir1.2-soup-3.0 gnome-control-center
  gnome-control-center-data gnome-control-center-faces gnome-keyring
  gnome-keyring-pkcs11 gnome-remote-desktop gnome-settings-daemon
  gnome-settings-daemon-common gnome-shell-extension-desktop-icons-ng
  gstreamer1.0-alsa gstreamer1.0-gl gstreamer1.0-plugins-base
  gstreamer1.0-plugins-base-apps gstreamer1.0-plugins-good gstreamer1.0-x gvfs
  gvfs-backends gvfs-common gvfs-daemons gvfs-fuse gvfs-libs heif-gdk-pixbuf
  heif-thumbnailer initramfs-tools initramfs-tools-bin initramfs-tools-core
  iproute2 keyboard-configuration krb5-locales language-pack-en
  language-pack-en-base language-pack-gnome language-pack-gnome-en-base
  ldap-utils libaom3 libarchive13t64 libaudit-common libaudit1
  libauthen-sasl-perl libbluetooth3 libcurl3t64-gnutls libcurl4t64 libdconf1
  libde265-0 libegl-mesa0 libexpat1 libfprint-2-2 libfprint-2-tod1
  libfreerdp-server3-3 libfreerdp3-3 libfreetype6 libfwupd2 libgbm1 libgdm1
linux@SAD-Ubuntu:~$
```

- La opción `-s` es "simular". Fijaos en que dice qué instalaría, pero no toca nada.

### A4. ¿Y si el sistema ya no recibe parches?

Buscad en <https://endoflife.date> la fecha de fin de soporte de:

- La versión de Ubuntu/Debian de vuestro Torrent-Auditor.
- Ubuntu 8.04, la que tiene Torrent-Vulnerable (Metasploitable2).

Rellenad:

| Sistema | Fecha fin de soporte | ¿Recibe parches hoy? |
|---|---|---|
| Torrent-Auditor | Ends in 2 years and 7 months (31 May 2029) | Sí |
| Torrent-Vulnerable | Ended 13 years ago (09 May 2013) | No |

**Pregunta A:** En la oferta de empleo de Torrent (dossier de la UD02) decían que trabajaban con "Ubuntu 8.04". Aunque pusieran todas las contraseñas bien, ¿qué problema sigue sin tener solución?

- El sistema operativo dejó de recibir parches de seguridad por mucho tiempo, por lo que es extremadamente vulnerable a toda clase de ataques.

---

## Parte B · ¿De dónde vienen los programas instalados?

### B1. Mirad de qué repositorios se descarga vuestro sistema

```console
linux@SAD-Ubuntu:~$ cat /etc/apt/sources.list.d/ubuntu.sources
Types: deb
URIs: http://es.archive.ubuntu.com/ubuntu/
Suites: noble noble-updates noble-backports
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

Types: deb
URIs: http://security.ubuntu.com/ubuntu/
Suites: noble-security
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg
```

```console
linux@SAD-Ubuntu:~$ ls /etc/apt/sources.list.d/
ubuntu.sources  ubuntu.sources.curtin.orig
```

**Investiga el fichero donde están configurados los repositorios** y comenta el significado de cada una de las líneas y su contenido.

### B2. `apt` solo instala paquetes firmados por claves en las que confía. Mirad dónde están esas claves

```bash
ls /etc/apt/trusted.gpg.d/ /usr/share/keyrings/ 2>/dev/null
```

```console
linux@SAD-Ubuntu:~$ ls /etc/apt/trusted.gpg.d/ /usr/share/keyrings/ 2>/dev/null
/etc/apt/trusted.gpg.d/:
ubuntu-keyring-2012-cdimage.gpg  ubuntu-keyring-2018-archive.gpg

/usr/share/keyrings/:
ubuntu-archive-keyring.gpg           ubuntu-pro-esm-apps.gpg
ubuntu-archive-removed-keys.gpg      ubuntu-pro-esm-apps-legacy.gpg
ubuntu-cloudimage-keyring.gpg        ubuntu-pro-esm-infra.gpg
ubuntu-cloudimage-removed-keys.gpg   ubuntu-pro-esm-infra-legacy.gpg
ubuntu-master-keyring.gpg            ubuntu-pro-fips.gpg
ubuntu-pro-anbox-cloud.gpg           ubuntu-pro-fips-preview.gpg
ubuntu-pro-cc-eal.gpg                ubuntu-pro-realtime-kernel.gpg
ubuntu-pro-cis.gpg                   ubuntu-pro-ros.gpg
```

Nessus, por ejemplo, no viene de los repositorios de Ubuntu, lo descargasteis de Tenable. Si lo instalasteis con `dpkg -i`, **`apt` no comprobó ninguna firma**, la confianza la pusisteis vosotros.

**Reflexiona:** ¿qué pasaría si un atacante introduce una clave falsa en `/usr/share/keyrings/`? ¿y si intercepta o reemplaza la clave oficial de Ubuntu `ubuntu-archive-keyrings.gpg`?

- Si el atacante modifica o sustituye el fichero ya referenciado, puede firmar un repositorio falso para ese origen concreto.
- Si intercepta la clave oficial de Ubuntu, el daño es mucho mayor, porque esa clave es de confianza global, para cualquier repositorio.

### B3. Información de un paquete concreto. Elegid `openssh-server` o `curl`

```bash
apt policy curl
```

```console
linux@SAD-Ubuntu:~$ apt policy curl
curl:
  Installed: (none)
  Candidate: 8.5.0-2ubuntu10.16
  Version table:
     8.5.0-2ubuntu10.16 500
        500 http://security.ubuntu.com/ubuntu noble-security/main amd64 Packages
     8.5.0-2ubuntu10.15 500
        500 http://es.archive.ubuntu.com/ubuntu noble-updates/main amd64 Packages
     8.5.0-2ubuntu10 500
        500 http://es.archive.ubuntu.com/ubuntu noble/main amd64 Packages
```

Anotad: **versión instalada, versión candidata, repositorio de origen y el significado de la tabla de versiones.**

### B4. Buscad paquetes instalados que **no proceden de ningún repositorio** (instalados a mano)

```bash
apt list --installed 2>/dev/null | grep -i local
```

```console
linux@SAD-Ubuntu:~$ apt list --installed 2>/dev/null | grep -i local
krb5-locales/noble-security,now 1.20.1-6ubuntu2.7 all [installed,upgradable to: 1.20.1-6ubuntu2.10]
libencode-locale-perl/noble,now 1.05-3 all [installed,automatic]
liblocale-gettext-perl/noble,now 1.07-6ubuntu5 amd64 [installed,automatic]
locales/noble-updates,noble-security,now 2.39-0ubuntu8.9 all [installed,automatic]
nessus/now 10.12.4 amd64 [installed,local]
python-babel-localedata/noble,now 2.10.3-3build1 all [installed,automatic]
linux@SAD-Ubuntu:~$
```

Cada línea que salga merece una pregunta: ¿quién lo instaló, de dónde, y quién lo verificó?

| Paquete | ¿Quién lo instaló? | ¿De dónde viene? | ¿Quién lo verificó? |
|---|---|---|---|
| `krb5-locales` | Alguien lo pidió explícitamente (no lleva `automatic`): tú o el instalador | `noble-security` | APT, contra `ubuntu-archive-keyring` |
| `libencode-locale-perl` | Dependencia (`automatic`) | `noble` | APT |
| `liblocale-gettext-perl` | Dependencia | `noble` | APT |
| `locales` | Dependencia | `noble-updates` y `noble-security` (la misma versión en ambos) | APT |
| `python-babel-localedata` | Dependencia | `noble` | APT |
| **`nessus`** | Manual, con `dpkg -i` | **Ningún repositorio** | **Nadie a través de APT** |

### B5. Comprobad si algún fichero de un paquete ha cambiado desde que se instaló

```bash
sudo dpkg -V
```

```console
linux@SAD-Ubuntu:~$ sudo dpkg -V
[sudo] password for linux:
??5??????   /opt/nessus/etc/nessus/fipsmodule.cnf
missing     /opt/nessus/var/nessus/tmp/nessusd
missing     /opt/nessus/var/nessus/tmp/nessusd.service
??5??????   /opt/nessus/lib/nessus/plugins/known_CA.inc
??5??????   /usr/lib/systemd/system/cloud-init.service
```

- Si no sale nada, todo coincide con lo instalado.
- Si salen líneas, la primera columna indica qué cambió (`5` = el hash MD5 es distinto). Los ficheros de configuración (`c`) cambian de forma normal. **Un binario (`/usr/bin/...`) modificado es una alarma.**

**Pregunta B:** `dpkg -V` ¿detectaría un rootkit que además hubiera modificado la base de datos de dpkg? ¿Por qué conviene comparar con una copia guardada **fuera** del sistema? (Pista: recordad la fase 4 de la intrusión.)

- No. `dpkg -V` solo comprueba que el sistema es coherente consigo mismo, no que sea legítimo.

---

## Parte C · Verificar una descarga: hash y firma

Descargaremos un programa pequeño y real, **GNU Hello**, junto a su firma y la lista de claves de GNU.

### C1. Descarga

```bash
mkdir -p ~/verif && cd ~/verif
wget https://ftp.gnu.org/gnu/hello/hello-2.12.1.tar.gz
wget https://ftp.gnu.org/gnu/hello/hello-2.12.1.tar.gz.sig
wget https://ftp.gnu.org/gnu/gnu-keyring.gpg
```

```console
linux@SAD-Ubuntu:~$ mkdir -p ~/verif && cd ~/verif
linux@SAD-Ubuntu:~/verif$ wget https://ftp.gnu.org/gnu/hello/hello-2.12.1.tar.gz
--2026-10-22 14:51:52--  https://ftp.gnu.org/gnu/hello/hello-2.12.1.tar.gz
Resolving ftp.gnu.org (ftp.gnu.org)... 209.51.188.20, 2001:470:142:3::b
Connecting to ftp.gnu.org (ftp.gnu.org)|209.51.188.20|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1033297 (1009K) [application/x-gzip]
Saving to: 'hello-2.12.1.tar.gz'

hello-2.12.1.tar.gz   100%[========================>]   1009K   894KB/s    in 1.1s

2026-10-22 14:51:53 (894 KB/s) - 'hello-2.12.1.tar.gz' saved [1033297/1033297]

linux@SAD-Ubuntu:~/verif$ wget https://ftp.gnu.org/gnu/hello/hello-2.12.1.tar.gz.sig
--2026-10-22 14:52:02--  https://ftp.gnu.org/gnu/hello/hello-2.12.1.tar.gz.sig
Resolving ftp.gnu.org (ftp.gnu.org)... 209.51.188.20, 2001:470:142:3::b
Connecting to ftp.gnu.org (ftp.gnu.org)|209.51.188.20|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 488 [application/pgp-signature]
Saving to: 'hello-2.12.1.tar.gz.sig'

hello-2.12.1.tar.gz.s 100%[========================>]     488  --.-KB/s    in 0s

2026-10-22 14:52:02 (319 MB/s) - 'hello-2.12.1.tar.gz.sig' saved [488/488]

linux@SAD-Ubuntu:~/verif$ wget https://ftp.gnu.org/gnu/gnu-keyring.gpg
--2026-10-22 14:52:28--  https://ftp.gnu.org/gnu/gnu-keyring.gpg
Resolving ftp.gnu.org (ftp.gnu.org)... 209.51.188.20, 2001:470:142:3::b
Connecting to ftp.gnu.org (ftp.gnu.org)|209.51.188.20|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 3698125 (3.5M)
Saving to: 'gnu-keyring.gpg'

gnu-keyring.gpg       100%[========================>]   3.53M   921KB/s    in 4.4s

2026-10-22 14:52:33 (825 KB/s) - 'gnu-keyring.gpg' saved [3698125/3698125]
```

### C2. Calculad el hash

```bash
sha256sum hello-2.12.1.tar.gz
```

```console
linux@SAD-Ubuntu:~/verif$ sha256sum hello-2.12.1.tar.gz
8d99142afd92576f30b0cd7cb42a8dc6809998bc5d607d88761f512e26c7db20  hello-2.12.1.tar.gz
```

Comparadlo con vuestro compañero de al lado. **¿Sale igual?** Debería.

### C3. Verificad la **firma** usando la lista de claves de GNU

```bash
gpg --no-default-keyring --keyring ./gnu-keyring.gpg --verify hello-2.12.1.tar.gz.sig hello-2.12.1.tar.gz
```

```console
linux@SAD-Ubuntu:~/verif$ gpg --no-default-keyring --keyring ./gnu-keyring.gpg --verify hello-2.12.1.tar.gz.sig hello-2.12.1.tar.gz
gpg: Signature made Sun 29 May 2022 11:05:00 PM UTC
gpg:                using RSA key 24093F016FFE8602EF449BB84C8EF3DA3FD37230
gpg: Good signature from "Reuben Thomas <rrt@sc3d.org>" [unknown]
gpg:                 aka "keybase.io/rrt <rrt@keybase.io>" [unknown]
gpg: WARNING: This key is not certified with a trusted signature!
gpg:          There is no indication that the signature belongs to the owner.
Primary key fingerprint: 2409 3F01 6FFE 8602 EF44  9BB8 4C8E F3DA 3FD3 7230
```

Deberíais ver `Good signature from ...`. También saldrá un aviso `WARNING: This key is not certified with a trusted signature`. **Es normal**, significa que vosotros no habéis dicho personalmente que esa clave sea de confianza. Anotad:

- Nombre de quien firmó.
- Fecha de la firma.
- Identificador de la clave.

### C4. El experimento: ¿qué pasa si alguien toca el fichero?

Haced una copia y añadid un carácter:

```bash
cp hello-2.12.1.tar.gz hello-manipulado.tar.gz
echo "x" >> hello-manipulado.tar.gz
sha256sum hello-2.12.1.tar.gz hello-manipulado.tar.gz
cp hello-2.12.1.tar.gz.sig hello-manipulado.tar.gz.sig
gpg --no-default-keyring --keyring ./gnu-keyring.gpg --verify hello-manipulado.tar.gz.sig hello-manipulado.tar.gz
```

```console
linux@SAD-Ubuntu:~/verif$ gpg --no-default-keyring --keyring ./gnu-keyring.gpg --verify hello-2.12.1.tar.gz.sig hello-2.12.1.tar.gz
gpg: Signature made Sun 29 May 2022 11:05:00 PM UTC
gpg:                using RSA key 24093F016FFE8602EF449BB84C8EF3DA3FD37230
gpg: Good signature from "Reuben Thomas <rrt@sc3d.org>" [unknown]
gpg:                 aka "keybase.io/rrt <rrt@keybase.io>" [unknown]
gpg: WARNING: This key is not certified with a trusted signature!
gpg:          There is no indication that the signature belongs to the owner.
Primary key fingerprint: 2409 3F01 6FFE 8602 EF44  9BB8 4C8E F3DA 3FD3 7230
linux@SAD-Ubuntu:~/verif$ cp hello-2.12.1.tar.gz hello-manipulado.tar.gz
linux@SAD-Ubuntu:~/verif$ echo "x" >> hello-manipulado.tar.gz
linux@SAD-Ubuntu:~/verif$ cp hello-2.12.1.tar.gz.sig hello-manipulado.tar.gz.sig
linux@SAD-Ubuntu:~/verif$ gpg --no-default-keyring --keyring ./gnu-keyring.gpg --veriff hello-manipulado.tar.gz.sig hello-manipulado.tar.gz
gpg: invalid option "--veriffy"
linux@SAD-Ubuntu:~/verif$ gpg --no-default-keyring --keyring ./gnu-keyring.gpg --verify hello-manipulado.tar.gz.sig hello-manipulado.tar.gz
gpg: Signature made Sun 29 May 2022 11:05:00 PM UTC
gpg:                using RSA key 24093F016FFE8602EF449BB84C8EF3DA3FD37230
gpg: BAD signature from "Reuben Thomas <rrt@sc3d.org>" [unknown]
linux@SAD-Ubuntu:~/verif$
```

Anotad: **¿cómo cambia el hash? ¿Qué dice ahora gpg?** (Debe decir `BAD signature`.)

Esto es lo que habría detectado el caso de vsftpd 2.3.4: una firma que no encaja.

### C5. Pregunta C: completad

| Situación | ¿Basta con el hash? | ¿Hace falta la firma? ¿Por qué? |
|---|---|---|
| El fichero se corrompió al descargarlo | **Sí** | No. Un fallo de transmisión es un error accidental, y el hash lo detecta porque el resultado no coincide con el publicado. No hay adversario, así que no se necesita autenticidad. |
| Un atacante cambió el fichero **y** el hash publicado en la misma web | **No** | Sí. El atacante controla ambos, así que el hash coincide con el fichero falso y la comprobación pasa. La firma GPG se hace con una clave privada que el atacante no tiene: no puede regenerarla, y la verificación falla. |
| Descargas desde un espejo que no es de confianza | **Depende de dónde se obtenga el hash** | Sí. El hash solo vale si se saca de un canal independiente y de confianza (la web oficial por HTTPS). Si el espejo publica el hash junto al fichero, se vuelve al caso anterior. La firma permite descargar de cualquier sitio: no importa quién sirva el fichero, solo si la firma es válida con la clave oficial. |

---

## Parte D · Política de actualizaciones para Torrent

Torrent tiene tres socios y nadie se encarga del mantenimiento. Redactad una política de máximo 6 líneas que responda:

1. ¿Quién se encarga de las actualizaciones y con qué frecuencia?
2. ¿Se actualiza todo automáticamente o primero se prueba? (Pensad en el equipo del servidor de impresión y en el ERP.)
3. ¿De dónde se puede instalar software y quién lo autoriza?
4. ¿Qué se hace con el equipo que ya no recibe parches (EOL)?

### Política de mantenimiento de software de Torrent

1. **Responsable y frecuencia:** Se lo encargaría a un socio designado (con un suplente) que revise e instale las actualizaciones el primer martes de cada mes.
2. **Pruebas:** Los equipos normales y el servidor de impresión se actualizan automáticamente; el ERP nunca se actualiza sin probar antes en una máquina virtual, con copia de seguridad previa y fuera del horario laboral.
3. **Origen del software:** Solo se puede instalarlo desde los repositorios oficiales del sistema o del fabricante, con firma o hash verificado.
4. **Autorización y registro:** Cualquier software nuevo necesita la aprobación de al menos dos socios y se anota en un inventario.
5. **Equipos EOL:** Se inventarían las fechas de fin de soporte; seis meses antes se planifica la migración, y el equipo que deje de recibir parches se sustituye o, si debe seguir temporalmente, se aísla de la red y de internet.
6. **Revisión:** Cada trimestre se comprueba que se cumple lo anterior.

Para ver la opción automática en Ubuntu/Debian (**solo mirad, no hace falta activarla**):

```bash
apt policy unattended-upgrades
cat /etc/apt/apt.conf.d/20auto-upgrades 2>/dev/null
```

```console
linux@SAD-Ubuntu:~/verif$ apt policy unattended-upgrades
unattended-upgrades:
  Installed: 2.9.1+nmu4ubuntu1
  Candidate: 2.9.1+nmu4ubuntu1
  Version table:
 *** 2.9.1+nmu4ubuntu1 500
        500 http://es.archive.ubuntu.com/ubuntu noble/main amd64 Packages
        100 /var/lib/dpkg/status
linux@SAD-Ubuntu:~/verif$ cat /etc/apt/apt.conf.d/20auto-upgrades 2>/dev/null
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Unattended-Upgrade "1";
linux@SAD-Ubuntu:~/verif$
```

**Debate:** actualizar automáticamente reduce el riesgo de que Torrent esté desactualizado, pero una actualización puede romper el ERP un viernes por la tarde. ¿Qué ganáis y qué perdéis con cada opción?

| | Automático | Manual con pruebas |
|---|---|---|
| **Ganáis** | No depende de que alguien se acuerde. | Control del momento, prueba previa y plan de vuelta atrás. El ERP no cambia sin que se sepa. |
| **Perdéis** | El control del momento. Un parche puede romper una dependencia del ERP o reiniciar un servicio en mal momento, y el fallo se diagnostica después. | Tiempo de exposición. Y depende de personas: con tres socios y nadie a cargo, la actualización "pendiente de probar" acaba sin hacerse nunca. |

**Windows:** los mismos conceptos existen allí con otros nombres. `winver` (versión), Windows Update (parches), `Get-FileHash fichero` (hash) y `Get-AuthenticodeSignature fichero` (firma de un ejecutable).
