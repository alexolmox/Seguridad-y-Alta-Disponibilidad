# Actividad: PAM, MFA y privilegios root
**Autor:** Alejandro Olmos

---

## PARTE 1: Contraseñas robustas con PAM

### 1.1 ¿Por qué PAM y no tocar /etc/shadow directamente?

PAM (*Pluggable Authentication Modules*) es la capa que usa Linux para gestionar la autenticación de cualquier programa: `login`, `su`, `passwd`, `sshd`... Configurar la política de contraseñas en PAM significa que se aplica una sola vez y afecta a todos los programas que autentican usuarios, en vez de tener que vigilar cada aplicación por separado.

---

### 1.2 Instalar y activar pwquality

El módulo `pam_pwquality` es el que comprueba la calidad de una contraseña nueva antes de aceptarla.

```bash
$ sudo apt update
Linux@SAD-Ubuntu:~$ sudo apt update
[sudo] password for linux: 
Hit:1 http://es.archive.ubuntu.com/ubuntu noble InRelease
Get:2 http://es.archive.ubuntu.com/ubuntu noble-updates InRelease [126 kB]
Get:3 http://security.ubuntu.com/ubuntu noble-security InRelease [126 kB]
Get:4 http://es.archive.ubuntu.com/ubuntu noble-backports InRelease [126 kB]
Get:5 http://es.archive.ubuntu.com/ubuntu noble-updates/main amd64 Packages [1,408 kB]
Get:6 http://security.ubuntu.com/ubuntu noble-security/main amd64 Packages [1,153 kB]
Get:7 http://security.ubuntu.com/ubuntu noble-security/main Translation-en [232 kB]

$ sudo apt install libpam-pwquality
Linux@SAD-Ubuntu:~$ sudo apt install libpam-pwquality
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
libpam-pwquality is already the newest version (1.4.5-3build1).
libpam-pwquality set to manually installed.
The following packages were automatically installed and are no longer required:
  libfwupd2 linux-headers-7.0.0-31-generic linux-hwe-7.0-headers-7.0.0-31
  linux-hwe-7.0-tools-7.0.0-31 linux-image-7.0.0-31-generic
  linux-modules-7.0.0-31-generic linux-tools-7.0.0-31-generic
Use 'sudo apt autoremove' to remove them.
0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
linux@SAD-Ubuntu:~$ 
```

Se activa comprobando que esta línea existe en `/etc/pam.d/common-password`, justo antes de la línea de `pam_unix.so` (en Debian/Ubuntu suele instalarse ya activada; si no, añádela):

```text
password requisite pam_pwquality.so retry=3
```

```bash
linux@SAD-Ubuntu:~$ password requisite pam_pwquality.so retry=3
Command 'password' not found, did you mean:
  command 'Spassword' from snap 5password (0.3)
  command '1password' from snap 1password (8.11.14)
  command 'assword' from deb assword (0.12.2-1)
See 'snap info <snapname>' for additional versions.
linux@SAD-Ubuntu:~$ 
```

Qué se considera una contraseña «de calidad» se configura en `/etc/security/pwquality.conf`:

```ini
# GNU nano 7.2                  /etc/security/pwquality.conf
# Configuration for systemwide password quality limits
#
# Defaults:
# Number of characters in the new password that must not be present in the old password.
# difok = 1
#
# Minimum acceptable size for the new password (plus one if credits are not disabled which is the default).
# (See pam_cracklib manual.)
# Cannot be set to lower value than 6.
#
minlen = 12

# The maximum credit for having digits in the new password. If less than 0 it is the minimum number of digits in the new password.
dcredit = -1

# The maximum credit for having uppercase characters in the new password.
# If less than 0 it is the minimum number of uppercase characters in the new password.
ucredit = -1

# The maximum credit for having lowercase characters in the new password.
# If less than 0 it is the minimum number of lowercase characters in the new password.
lcredit = -1

# The maximum credit for having other characters in the new password.
# If less than 0 it is the minimum number of other characters in the new password.
ocredit = -1

# The minimum number of required classes of characters for the new password (digits, uppercase, lowercase, others).
#
minclass = 3

# The maximum number of allowed consecutive same characters in the new password.
# The check is disabled if the value is 0.
maxrepeat = 3

# Whether to check for the words from the cracklib dictionary.
# The check is enabled if the value is not 0.
#
dictcheck = 1

# Enforces pwquality checks on the root user password.
# Enabled if the option is present.
# enforce_for_root
```

---

### 1.3 Práctica guiada: aplicar la política

1. Edita `/etc/security/pwquality.conf` con `sudo` y fija los valores de la tabla anterior (descomenta la línea o añádela si no existe).
2. Comprueba que la línea `pam_pwquality.so` está presente en `/etc/pam.d/common-password`.
3. Como el usuario `hugo` (grupo operarios, creado ayer en la práctica de ACL), intenta cambiarle la contraseña por algo débil y observa el rechazo:

```bash
linux@SAD-Ubuntu:~$ sudo passwd hugo
New password: 
BAD PASSWORD: The password contains less than 1 uppercase letters
New password: 
```

4. Repite con una contraseña que cumpla la política y confirma que esta vez se acepta:

```bash
linux@SAD-Ubuntu:~$ sudo passwd hugo
New password: 
BAD PASSWORD: The password contains less than 1 uppercase letters
New password: 
Retype new password: 
passwd: password updated successfully
linux@SAD-Ubuntu:~$ 
```

5. Como `root`, intenta ponerle a `root` mismo una contraseña débil: sin `enforce_for_root` se aceptaría sin más. Actívala y repite la prueba:

```bash
Linux@SAD-Ubuntu:~$ sudo passwd root
[sudo] password for linux: 
New password: 
BAD PASSWORD: The password contains less than 1 uppercase letters
Retype new password: 
passwd: password updated successfully
Linux@SAD-Ubuntu:~$ 
```
