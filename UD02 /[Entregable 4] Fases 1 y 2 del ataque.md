# [Entregable 4] Fases 1 y 2 del ataque

## Fase 2: exploración y contramedidas

### Medición - ANTES

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

### Procesos y comandos

```text
msfadmin@metasploitable:~$ sudo netstat -tulpn
[sudo] password for msfadmin:
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:45344           0.0.0.0:*               LISTEN      4601/rmiregistry
tcp        0      0 0.0.0.0:512             0.0.0.0:*               LISTEN      4465/xinetd
tcp        0      0 0.0.0.0:2049            0.0.0.0:*               LISTEN      -
tcp        0      0 0.0.0.0:514             0.0.0.0:*               LISTEN      4465/xinetd
tcp        0      0 0.0.0.0:8009            0.0.0.0:*               LISTEN      4564/jsvc
tcp        0      0 0.0.0.0:6697            0.0.0.0:*               LISTEN      4613/unrealircd
tcp        0      0 0.0.0.0:3306            0.0.0.0:*               LISTEN      4206/mysqld
tcp        0      0 0.0.0.0:1099            0.0.0.0:*               LISTEN      4601/rmiregistry
tcp        0      0 0.0.0.0:6667            0.0.0.0:*               LISTEN      4613/unrealircd
tcp        0      0 0.0.0.0:139             0.0.0.0:*               LISTEN      4449/smbd
```

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

```text
msfadmin@metasploitable:~$ grep -v '^#' /etc/inetd.conf
telnet          stream  tcp     nowait  telnetd /usr/sbin/tcpd  /usr/sbin/in.telnetd
tftp            dgram   udp     wait    nobody  /usr/sbin/tcpd  /usr/sbin/in.tftpd /srv/tftp
shell           stream  tcp     nowait  root    /usr/sbin/tcpd  /usr/sbin/in.rshd
exec            stream  tcp     nowait  root    /usr/sbin/tcpd  /usr/sbin/in.rexecd
ingreslock stream tcp nowait root /bin/bash bash -i
```

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
/etc/xinetd.d/vsftpd:   disable                 = no
...
```

```text
msfadmin@metasploitable:~$ sudo grep -rn -w 22 /etc/inetd.com /etc/xinetd.conf /etc/xinetd.conf /etc/xinetd.d 2>/dev/null
/etc/xinetd.d/daytime:3:# then prints it out in a format like this: "Wed Nov 13 22:30:27 EST 2002".
```

```text
linux@SAD-Ubuntu:~$ sudo nmap -sV 512 192.168.1.20
[sudo] password for linux:
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

### Serivicio 1

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

(después, hacer `sudo reboot`)

### Servicio 2

```text
msfadmin@metasploitable:~$ sudo grep -n -w login \
> /etc/inetd.conf /etc/xinetd.conf \
> /etc/xinetd.d/* 2>/dev/null
/etc/inetd.conf:6:#login          stream  tcp     nowait  root    /usr/sbin/tcpd  /usr/sbin/in.rlogind
msfadmin@metasploitable:~$
```

(Está en inetd.conf)

```text
msfadmin@metasploitable:~$ sudo cp /etc/inetd.conf /root/inetd.conf.bak
msfadmin@metasploitable:~$ sudo sed -i 's/^login/#login/' /etc/inetd.conf
msfadmin@metasploitable:~$ sudo /etc/init.d/xinetd reload
 * Reloading internet superserver configuration xinetd                    [ OK ]
msfadmin@metasploitable:~$
```

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

### Medición - DESPUÉS

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
(Los servicios Proftpd y Apache Jserv no están en la lista)

## Medición DESPUÉS

| Aspecto | Antes | Contramedida | Después |
|---|---|---|---|
| Puertos abiertos | 22 | Apagado de ProFTPD (2121) | 21 |
| Servicios/versiones visibles | 22 servicios con versión: vsftpd 2.3.4, OpenSSH 4.7p1, Linux telnetd, Postfix, BIND 9.4.2, Apache 2.2.8, rpcbind, Samba 3.X-4.X (x2), rexecd, rshd, grmiregistry, root shell (1524), NFS, ProFTPD 1.3.1, MySQL 5.0.51a, PostgreSQL 8.3.x, VNC 3.3, X11, UnrealIRCd, Apache Jserv, Tomcat 1.1 | Servicio ProFTPD detenido | 21 servicios, los mismos menos ProFTPD 1.3.1 |
| Respuesta al ping | Sí responde | Ninguna todavía | Sí responde |

## Preguntas finales

### 1. ¿Qué contramedida ha reducido más lo que ve el atacante? ¿Por qué?

Cerrar o filtrar puertos (firewall con política de denegación por defecto) y desactivar servicios innecesarios. Actúan en la primera fase del reconocimiento: si un puerto aparece como `filtered` o no existe, el atacante no obtiene versión, banner ni servicio contra el que probar exploits. Se reduce la superficie de ataque entera, no solo la información sobre ella.

### 2. Si solo ocultáis un banner pero el servicio sigue activo, ¿la vulnerabilidad sigue ahí? Razonad la respuesta.

La vulnerabilidad seguiría ahí, porque el banner es información sobre el servicio, no parte de su lógica. La vulnerabilidad está en el código del software.

### 3. Torrent-Vulnerable usa un sistema operativo sin soporte desde hace años. ¿Puede una contramedida de estas sustituir a actualizarlo? ¿Qué haríais en una empresa real?

- No, no lo sustituye. Un SO sin soporte acumula vulnerabilidades conocidas que nunca tendrán parche. Las contramedidas reducen la exposición o la probabilidad de explotación, pero no eliminan el fallo.
- En una empresa real, migraría y actualizaría a un SO con soporte.

### 4. De todas las contramedidas de las partes A y B, ¿cuáles evitan que el atacante encuentre información y cuáles solo ayudan a detectar que lo están intentando?

*(Aún no las hemos aplicado todas, pero pensad en cuáles serían de cada tipo.)*

**Evitan que el atacante encuentre información:**

- Desactivar servicios innecesarios
- Ocultar banners
- Restringir por IP de origen
- Firewall (UFW)

**Solo ayudan a detectar:**

- Logging (rsyslog, journald)
- Auditoría del sistema (auditd)

