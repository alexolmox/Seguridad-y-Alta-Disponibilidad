# Ataques de suplantación de identidad

## 3. IP spoofing

### ¿Qué es?
###### Es una técnica que consiste en falsificar la dirección IP de origen en los paquetes de datos para hacer creer que provienen de una fuente confiable o de otro sistema.
<br>

### Cómo se lleva a cabo
###### 1. Todo paquete de red tiene una cabecera con la IP de origen y la de destino.
###### 2. El ataque modifica el campo de origen y pone otra IP (la de otro equipo, una inventada o la de la propia víctima).
###### 3. El destino recibe el paquete y cree que viene de esa IP falsa.
###### 4. Las respuestas se envían a la IP falsificada, no al atacante. Por eso, el spoofing suele usarse cuando el atacante no necesita recibir la respueta.
<br>

### Qué categoría(s) de amenaza compromete
###### Vulnera principalmente la Autenticidad, de forma derivada la Integridad y Disponibilidad
###### El enfoque es engañar al receptor, suplantando la identidad de otro emisor, permitiendo inyectar código bajo el pseudónimo de otro emisor considerado seguro, destruyendo por completo la garantía del origen, ergo, rompe la Autenticidad
<br>

### Ejemplo o caso real
###### Un caso famoso de las primeras veces que se aplicó esta técnica fue el de Kevin Mitnick en 1994. Este quería acceder ilegalmente al ordenador del experto en seguridad Tsutomu Shimomura. Para realizarlo, falsificó la dirección IP de una máquina de confianza que el sistema del atacado reconociera. Dado que el ataque parecía provenir de una dirección IP conocida y confiable, el sistema objetivo no solicitó una contraseña. Kevin no podría recibir respuestas, puesto que iban dirigidas a la máquina real, pero adivinó los códigos de confirmación para completar el protocolo de enlace y tener acceso unidireccional. Esto le permitió extraer datos y acaparar titulares, demostrando los riesgos de confiar únicamente en las direcciones IP.
<br>

### Medida de prevención
###### Se puede prevenir este ataque de varias formas, entre ellos: *Filtrado de paquetes:*Evalúa la cabecera de cada paquete IP (analizando aspectos como la dirección de origen, destino y puertos) para determinar si permite o bloquea el tráfico entrante o saliente según las reglas establecidas. Si un paquete incumple las reglas o muestra inconsistencia en sus datos, el firewall o dispositivo de red lo descarta. *Autenticación mediante infraestructura de clave pública:* Usa un cifrado asimétrico, una clave privada para cifrar y autenticar, y una pública para descifrar. Impide que terceros deduzcan la clave privada, lo que permite verificar con seguridad a usuarios y dispositivos frente a ataques de suplantación. *Supervisión de redes y Firewalls:* Detecta de forma temprana actividades sospechosas para mitigar daños, aunque la suplantación de IP intente ocultarlas. El firewall autentica direcciones IP y filtra el tráfico potencialmente malicioso para evitar accesos no autorizados. *Formación en materia de seguridad:*Enseñar a los usuarios a evitar trampas como enlaces sospechosos para mitigar los datos de la suplantación de IP.
<br>
### Fuente
###### https://www.keyfactor.com/es/blog/what-it-is-ip-spoofing-how-to-protect-against-it/
