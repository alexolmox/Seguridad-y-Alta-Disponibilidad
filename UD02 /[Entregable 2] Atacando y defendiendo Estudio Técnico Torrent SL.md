### Atacando y defendiendo Estudio Técnico Torrent SL

| # | Dato encontrado | Fuente | ¿Para qué le sirve al atacante? | Contramedida propuesta |
|---|---|---|---|---|
| 1 | Servidor con Ubuntu 8.04, Apache y vsftpd | 2 | Sistema sin soporte desde hace más de 10 años, con vulnerabilidades conocidas y sin parches. Es la vía más probable de entrada técnica. FTP además transmite credenciales en claro. | Migrar a una versión con soporte, aplicar actualizaciones, sustituir FTP por SFTP y valorar un hosting gestionado. |
| 2 | 3 puestos con Windows 7 | 2 | Sin soporte ni parches de seguridad. Cualquier malware recibido por correo tiene vía libre. | Actualizar a un sistema con soporte, antivirus/EDR y usuarios sin permisos de administrador. |
| 3 | Subdominios nas, pruebas, portal, mail, erp | 4 | Mapa de la superficie de ataque. El NAS expuesto a Internet es un objetivo directo. Los entornos de pruebas suelen tener contraseñas débiles y datos reales. | Quitar del DNS público lo innecesario, no exponer el NAS (acceso solo por VPN) y eliminar o proteger "pruebas". |
| 4 | NAS con las copias de seguridad | 2 | Quien lo comprometa puede robar datos y cifrarlos (ransomware), y destruir la capacidad de recuperación. | Regla 3-2-1 con una copia offline o inmutable, y NAS sin acceso desde Internet. |
| 5 | Nota adhesiva legible en el monitor | 5 | Muy probablemente una contraseña. La foto la expone al mundo entero. | Gestor de contraseñas, política de mesa limpia y revisar las fotos antes de publicarlas. |
| 6 | Marta teletrabaja con el portátil en el wifi de una cafetería | 5 | Permite ataques de wifi falso (evil twin) o espionaje del tráfico. Además revela cuándo y dónde está, y que están cerrando un proyecto importante. | VPN obligatoria fuera de la oficina, HTTPS/MFA y mejor usar datos móviles. |
| 7 | Móvil de Andrés y "urgencias técnicas" | 6 | Permite vishing o smishing haciéndose pasar por un cliente o proveedor, o suplantar a Andrés ante el resto. | No publicar móviles personales y protocolo de verificación de identidad en llamadas. |
| 8 | Nombres, cargos y formato de correo (inicial+apellido) | 1 | Permite deducir otras cuentas, hacer phishing personalizado y elegir a quién suplantar. | Formación en phishing, correos genéricos para contacto público y filtros anti-phishing. |
| 9 | Portátil como servidor de impresión y un practicante como administrador | 2 | Un equipo sin gestionar es un buen punto de apoyo para moverse por la red. Que todo recaiga en un becario con conocimientos básicos indica falta de control y una posible víctima de ingeniería social. | Quitar el portátil de la función de servidor, segmentar la red y documentar permisos. Evitar publicar detalles técnicos en las ofertas. |
| 10 | Oficina cerrada del 1 al 15 de agosto y nueva ubicación | 1 | Ventana para intrusión física o para colocar dispositivos sin testigos. También demora la detección de un incidente. | Alarma, control de acceso y guardar los equipos críticos bajo llave. |

### Plan de ataque
###### Elegir la ventana del 1 al 15 de agosto, con Marta sin correo y la oficina vacía.
###### Falsificar el correo de Marta para pedir a Lucía un pago o el cambio de IBAN de un proveedor, aprovechando que nadie puede verificarlo.
###### En paralelo, enviar un phishing a los tres socios con una página de login falsa del ERP/portal, o un adjunto dirigido a los equipos con Windows 7.
###### Entrar por el servidor Ubuntu y probar credenciales que aparezcan en la nota adhesiva o se reutilicen.
###### Cifrar o robar los datos y las copias del NAS para chantajear, o desviar los pagos.

### Las 3 contramedidas más importantes
###### 1. Autenticación de correo (SPF estricto, DKIM, DMARC) más verificación por segundo canal de pagos y cambios de IBAN. El fraude del CEO es el ataque más rentable y barato, y aquí hay dos fallos que se combinan (spoofing sin barreras y una persona que concentra los pagos). Es gratis o muy barato y corta el ataque principal. La verificación por otra vía protege incluso si el correo falso llega.
###### 2. Actualizar o retirar los sistemas sin soporte (Ubuntu 8.04, Windows 7) y cerrar lo expuesto (NAS, pruebas, FTP), con una copia de seguridad offline. Un sistema sin parches se compromete con herramientas públicas, y el NAS con las copias es lo que decide si un ransomware se convierte en catástrofe. Una copia offline es el último seguro.
###### 3. MFA en todo lo accesible (ERP, portal, correo, registrador) más gestor de contraseñas y formación. Casi todas las rutas del plan acaban en robar credenciales (phishing, nota adhesiva, correo personal del WHOIS). El MFA hace que una contraseña robada no baste, y la formación reduce los fallos humanos que ninguna tecnología evita.
