# Informe de auditoría de vulnerabilidades 

## 1. Contexto y alcance
El análisis se centra en identificar servicios expuestos, configuraciones obsoletas o inseguras y vulnerabilidades conocidas que puedan ser explotadas por un atacante interno o externo para comprometer la confidencialidad, integridad o disponibilidad de MetaSploitable2.

## 2. Metodología
He realizado un escaneo básico (Basic Network Scan) en Nessus con la dirección IP de MetaSploitable2. Después del escaneo, Nessus me ha ofrecido una lista con todas las vulnerabilidades de la máquina.

## 3. Resumen de resultados
(Cuántas vulnerabilidades por severidad — puedes usar una tabla o una lista) - Pondré 3 ejemplos (de las vulnerabilidades que han salido después del escaneo):
| Vulnerabilidad | Severidad / CVSS | Origen | Descripción |
| :--- | :--- | :--- | :--- |
| Canonical Ubuntu Linux SEoL | Crítica / 10.0 | Configuración / Gestión de obsolescencia | El servidor está ejecutando **Ubuntu Linux 8.04**, una versión cuyo soporte de seguridad finalizó en mayo de 2013. Al estar en estado *Security End of Life* (SEoL), Canonical ya no publica parches ni actualizaciones de seguridad para este sistema. |
| Samba Badlock Vulnerability | High / 7.5 | Diseño de protocolos / Implementación de software | Un atacante mediante un ataque Man-in-the-Middle (MitM) puede interceptar el tráfico de red y forzar la degradación del nivel de autenticación (*downgrade*), lo que le permite ejecutar llamadas arbitrarias en la red Samba, modificar datos sensibles de seguridad en Active Directory o desactivar servicios críticos. |
| SSL Version 2 and 3 Protocol Detection | Critical / 9.8 | Configuración de seguridad / Criptografía obsoleta | El servidor está usando sistemas de seguridad antiguos y defectuosos (SSL 2.0 y SSL 3.0) para proteger la información que envía y recibe. Un atacante en la misma red podría interceptar la conexión, romper esa protección antigua y leer o modificar los datos privados (como contraseñas o información confidencial) que viajan entre los usuarios y el servidor. |

## 4. Conclusión
Valoración global:
Desde una perspectiva estricta de ciberseguridad, no se debería firmar un contrato de mantenimiento en el estado actual de la máquina si el compromiso implica asumir responsabilidad sobre la seguridad, disponibilidad o confidencialidad del sistema sin margen de modificación. El equipo presenta un nivel de riesgo inaceptable.
