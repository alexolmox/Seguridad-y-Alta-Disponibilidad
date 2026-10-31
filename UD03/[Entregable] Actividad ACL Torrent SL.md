# Configuración de ACL en Linux
**Autor:** Alejandro Olmos

---

## Tarea 1: Preparar el entorno

### 1. Comprueba que el soporte de ACL está disponible en tu sistema
```console
linux@SAD-Ubuntu:~$ dpkg -s acl
Package: acl
Status: install ok installed
Priority: optional
Section: utils
Installed-Size: 192
Maintainer: Ubuntu Developers <ubuntu-devel-discuss@lists.ubuntu.com>
Architecture: amd64
Multi-Arch: foreign
Version: 2.3.2-1build1.1
Depends: libacl1 (= 2.3.2-1build1.1), libc6 (>= 2.38)
Description: access control list utilities
This package contains the getfacl and setfacl utilities needed for
manipulating access control lists. It also contains the chacl IRIX
compatible utility.
Homepage: https://savannah.nongnu.org/projects/acl/
Original-Maintainer: Guillem Jover <guillem@debian.org>
```

### 2. Crea los tres grupos y los nueve usuarios del enunciado, asignando a cada usuario su grupo correspondiente
```bash
linux@SAD-Ubuntu:~$ sudo groupadd operarios
linux@SAD-Ubuntu:~$ sudo groupadd ingenieria
linux@SAD-Ubuntu:~$ sudo groupadd supervisores

linux@SAD-Ubuntu:~$ sudo useradd -m -g operarios alberto
linux@SAD-Ubuntu:~$ sudo useradd -m -g operarios oscar
linux@SAD-Ubuntu:~$ sudo useradd -m -g operarios hugo

linux@SAD-Ubuntu:~$ sudo useradd -m -g ingenieria nestor
linux@SAD-Ubuntu:~$ sudo useradd -m -g ingenieria victor
linux@SAD-Ubuntu:~$ sudo useradd -m -g ingenieria lucia

linux@SAD-Ubuntu:~$ sudo useradd -m -g supervisores carlos
linux@SAD-Ubuntu:~$ sudo useradd -m -g supervisores mary
linux@SAD-Ubuntu:~$ sudo useradd -m -g supervisores john
```

### 3. Crea la estructura de ficheros
Un fichero (o script) ejecutable llamado `hmi_planta`, y una carpeta `planos/` con los ficheros `autoclave.dwg`, `caldera.dwg` y `destilacion.dwg`.

```bash
linux@SAD-Ubuntu:~$ touch hmi_planta
linux@SAD-Ubuntu:~$ chmod +x hmi_planta
linux@SAD-Ubuntu:~$ mkdir planos
linux@SAD-Ubuntu:~$ touch planos/autoclave.dwg planos/caldera.dwg planos/destilacion.dwg
```

---

## Tarea 2: Permisos base (propietario / grupo / resto)

### 4. Decide qué usuario y qué grupo deben ser propietarios de `hmi_planta` y de `planos/`
```bash
linux@SAD-Ubuntu:~$ sudo chown root:ingenieria hmi_planta
linux@SAD-Ubuntu:~$ sudo chown root:ingenieria planos
linux@SAD-Ubuntu:~$ sudo chown root:ingenieria planos/*.dwg
```

### 5. Aplica con `chmod` unos permisos de partida restrictivos
Permisos de partida (por ejemplo `750` o `700`), de forma que el resto del acceso tenga que concederse explícitamente mediante ACL.

```bash
linux@SAD-Ubuntu:~$ sudo chmod 750 hmi_planta
linux@SAD-Ubuntu:~$ sudo chmod 750 planos
linux@SAD-Ubuntu:~$ sudo chmod 640 planos/*.dwg
```

---

## Tarea 3: Configurar las ACL con `setfacl`

Sintaxis básica de referencia:
* `$ setfacl -m g:grupo:rwx recurso` *(ACL de acceso para un grupo)*
* `$ setfacl -m u:usuario:rwx recurso` *(ACL de acceso para un usuario concreto)*
* `$ setfacl -R -m g:grupo:rwx planos/` *(aplicar de forma recursiva a una carpeta)*
* `$ setfacl -d -m g:grupo:rwx planos/` *(ACL por defecto para los ficheros nuevos)*

### 6. Aplica las ACL necesarias sobre `hmi_planta`
```bash
linux@SAD-Ubuntu:~$ sudo setfacl -m g:operarios:--x hmi_planta
linux@SAD-Ubuntu:~$ sudo setfacl -m g:ingenieria:rwx hmi_planta
```

### 7. Aplica las ACL necesarias sobre la carpeta `planos/` y sobre los ficheros que contiene
```bash
linux@SAD-Ubuntu:~$ sudo setfacl -m g:ingenieria:rwx planos/
linux@SAD-Ubuntu:~$ sudo setfacl -R -m g:ingenieria:rwx planos/
linux@SAD-Ubuntu:~$ sudo setfacl -R -m g:supervisores:r-x planos/
linux@SAD-Ubuntu:~$ sudo setfacl -R -m g:operarios:r-- planos/
linux@SAD-Ubuntu:~$ sudo setfacl -m u:alberto:r-- planos/autoclave.dwg
linux@SAD-Ubuntu:~$ sudo setfacl -m u:oscar:r-- planos/caldera.dwg
```

### 8. ACL por defecto (`setfacl -d`) para que los ficheros nuevos hereden automáticamente los permisos
```bash
linux@SAD-Ubuntu:~$ sudo setfacl -d -m g:ingenieria:rwx planos/
linux@SAD-Ubuntu:~$ sudo setfacl -d -m g:supervisores:r-x planos/
linux@SAD-Ubuntu:~$ sudo setfacl -d -m g:operarios:r-- planos/
```

---

## Verificación con `getfacl`

Comprobar las ACL de `hmi_planta` y `planos/`:

```console
linux@SAD-Ubuntu:~$ sudo getfacl hmi_planta
# file: hmi_planta
# owner: root
# group: ingenieria
user::rwx
group::r-x
group:operarios:--x
group:ingenieria:rwx
mask::rwx
other::---

linux@SAD-Ubuntu:~$ sudo getfacl -R planos/
# file: planos/
# owner: root
# group: ingenieria
user::rwx
group::r-x
group:operarios:r--
group:ingenieria:rwx
group:supervisores:r-x
mask::rwx
other::---
default:user::rwx
default:group::r-x
default:group:operarios:r--
default:group:ingenieria:rwx
default:group:supervisores:r-x
default:mask::rwx
default:other::---

# file: planos//destilacion.dwg
# owner: root
# group: ingenieria
user::rw-
group::rw-
group:operarios:r--
group:ingenieria:rwx
group:supervisores:r-x
mask::rwx
other::r--

# file: planos//autoclave.dwg
# owner: root
# group: ingenieria
user::rw-
user:alberto:r--
group::rw-
group:operarios:r--
group:ingenieria:rwx
group:supervisores:r-x
mask::rwx
other::r--

# file: planos//caldera.dwg
# owner: root
# group: ingenieria
user::rw-
user:oscar:r--
group::rw-
group:operarios:r--
group:ingenieria:rwx
group:supervisores:r-x
mask::rwx
other::r--
```

---

## 5. Entrega

* Un script (`.sh`) con todos los comandos ejecutados (`grupos`, `usuarios`, `chmod` y `setfacl`), comentado.
* Capturas de pantalla de la salida de `getfacl` para `hmi_planta` y para `planos/`.
* Las respuestas a las preguntas de la sección 6.

---

## 6. Preguntas de reflexión

### 1. ¿Por qué no bastan los permisos clásicos de Unix (usuario / grupo / resto) para implementar el esquema del diagrama? Pon un ejemplo concreto de este caso.
Porque, por ejemplo, sobre el recurso `hmi_planta`, el grupo `ingenieria` necesita permisos totales (`rwx`) y el grupo `operarios` necesita permisos de ejecución (`--x`). Si usáramos solo los permisos clásicos, el grupo del fichero solo podría tener uno de los dos conjuntos, resultando imposible otorgar permisos distintos a ambos grupos sin interferir con el "resto".

### 2. ¿Qué es la máscara (mask) de una ACL y cómo afecta a los permisos efectivos de un usuario o grupo, aunque su entrada de ACL indique más permisos?
La máscara de una ACL actúa como un límite superior o filtro para los permisos efectivos de todos los grupos y usuarios específicos (excepto el propietario del usuario y el "resto").

### 3. ¿Qué diferencia hay entre una ACL de acceso y una ACL por defecto? ¿Cuándo se aplica cada una?
* **ACL de acceso:** Define los permisos concretos que se aplican de manera inmediata sobre un fichero o directorio existente en ese preciso momento.
* **ACL por defecto (`-d`):** Solo se puede aplicar sobre directorios. No afecta a los permisos del directorio en sí, sino que actúa como una "plantilla" hereditaria: todos los subdirectorios y ficheros que se creen dentro de ese directorio en el futuro heredarán automáticamente estas reglas como sus ACLs de acceso iniciales.

### 4. Si mañana se incorpora un nuevo operario, ¿qué pasos tendrías que repetir para darle exactamente el mismo acceso que al resto de operarios?
Crear el usuario nuevo y asignarlo directamente al grupo `operarios` al darlo de alta en el sistema (por ejemplo, mediante `useradd -g operarios nuevousuario` o modificándolo con `usermod -g operarios nuevousuario`). Al pertenecer al grupo, heredará de forma automática todas las ACLs asociadas al grupo `operarios` tanto en `hmi_planta` como en la carpeta `planos/`.
