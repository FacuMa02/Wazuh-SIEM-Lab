# Bitácora Técnica de Laboratorio: Despliegue de SIEM Wazuh &amp; Respuesta Activa

**Proyecto:** Implementación de Laboratorio de Detección y Monitoreo de Seguridad  
**Plataforma SIEM:** Wazuh (Manager, Indexer, Dashboard)  
**Entorno:** Red Virtualizada Local (LAN Bridge) con Hipervisor Tipo 2  
**Autor:** Facundo Maidana

---

## 1\. Objetivos del Laboratorio

1. Desplegar un servidor centralizado de recolección y análisis de registros de seguridad (**Wazuh SIEM**) en un entorno Linux Server.
2. Configurar una arquitectura de red en modo puente (**Bridged Network**) para posibilitar visibilidad y comunicación bidireccional directa en la LAN local.
3. Enrolar múltiples endpoints con diversos sistemas operativos (Ubuntu Desktop, Windows 11 y equipo host físico) mediante agentes ligeros de telemetría.
4. Simular un vector de ataque de fuerza bruta sobre el servicio SSH para validar la ingesta y correlación de registros en `/var/log/auth.log`.
5. Implementar y validar un mecanismo de defensa automatizada mediante **Active Response** para el bloqueo dinámico de IPs atacantes.
6. Documentar de forma sistemática los incidentes técnicos y fallos de infraestructura encontrados durante el despliegue.

---

## 2\. Arquitectura de Red y Especificaciones del Entorno

### 2.1\. Nodos de la Infraestructura

* **Servidor Central SIEM (Wazuh Server):**  
  * **Sistema Operativo:** Ubuntu Server LTS
  * **Componentes:** Wazuh Manager, Wazuh Indexer, Wazuh Dashboard.
  * **Configuración de Red:** Adaptador en modo puente (Bridged) conectado a la interfaz física Ethernet del equipo anfitrión para visibilidad completa en la subred LAN.
* **Endpoints Monitoreados (Agentes Wazuh):**  
  * **Nodo 1:** Máquina Virtual Ubuntu Desktop (Simulación de estación de trabajo Linux).
  * **Nodo 2:** Máquina Virtual Windows 11 (Simulación de estación de trabajo corporativa con auditoría de eventos Windows).
  * **Nodo 3:** Laptop Host (Equipo físico conectado a la misma LAN para validación de telemetría híbrida).
* **Nodo Atacante / Generador de Tráfico:**  
  * Estación en la LAN ejecutando intentos de acceso automatizados por SSH hacia los nodos de la red.

<img width="1834" height="802" alt="image" src="https://github.com/user-attachments/assets/3d5a69d1-b56f-4644-b590-4a3d073ddfc5" />

*Modelo de arquitectura de red que se configuró para el proyecto.*

---

## 3\. Bitácora de Despliegue y Resolución de Incidentes (Troubleshooting)

### Incidencia 1: Conflicto de Puertos y Validación de Requisitos de Hardware

* **Descripción del Problema:** Durante el primer intento de despliegue de Wazuh, la instalación falló reportando colisiones con servicios preexistentes en los puertos requeridos (puertos estándar de indexación 9200/9300 y comunicación de agentes 1514/1515/55000), sumado a advertencias de recursos insuficientes.
* **Diagnóstico Técnico:** Se detectaron procesos activos en segundo plano ocupando puertos necesarios para los sockets de comunicación de Wazuh.
* **Resolución:**  
  1. Se inspeccionaron los puertos y procesos en escucha mediante herramientas de red (`ss -tulpn` / `netstat`).
  2. Se terminaron los servicios conflictivos y se depuraron los demonios residuales.
  3. Se ajustó la asignación de memoria RAM y núcleos de CPU en la configuración de la máquina virtual para cumplir los requerimientos de Wazuh Indexer.
 <img width="1855" height="409" alt="image" src="https://github.com/user-attachments/assets/f1939ffe-86b0-4696-871d-e1a4371a46fb" />



### Incidencia 2: Inestabilidad de Conectividad en Modo Puente sobre Wi-Fi

* **Descripción del Problema:** Al configurar la interfaz de red virtual en modo puente (Bridge) vinculada al adaptador inalámbrico (Wi-Fi), los paquetes de difusión y las solicitudes DHCP presentaban pérdidas intermitentes debido a restricciones del modo promiscuo en controladores Wi-Fi.
* **Diagnóstico Técnico:** La mayoría de las tarjetas de red inalámbricas y puntos de acceso no admiten múltiples direcciones MAC virtuales sobre una única asociación de radio de capa 2.
* **Resolución:**  
  1. Se conectó el equipo anfitrión a la red mediante interfaz física **Ethernet** (cableada).
  2. Se reconfiguró el adaptador de VirtualBox para enlazar directamente con la tarjeta de red Gigabit Ethernet física.
  3. Resultado: Asignación inmediata y estable de direcciones IP dentro del rango de la LAN local, permitiendo visibilidad total entre todos los nodos y el servidor.

<img width="1824" height="905" alt="image" src="https://github.com/user-attachments/assets/10a5cafe-f9c7-4cc5-a70a-bb3941b38221" />

### Incidencia 3: Despliegue del Dashboard y Acceso Administrativo

* **Descripción del Problema:** Fallas en la inicialización y acceso al Dashboard de Wazuh al intentar configuraciones remotas vía sesiones SSH no interactivas.
* **Diagnóstico Técnico:** Dependencias del servicio web del dashboard y certificados SSL autofirmados requerían reinicio limpio del servicio y verificación del estado del demonio `wazuh-dashboard`.
* **Resolución:**  
  1. Reinstalación controlada de la interfaz gráfica y reinicio del servicio (`systemctl restart wazuh-dashboard`).
  2. Verificación de logs en `/var/ossec/logs/ossec.log` y validación de escucha en el puerto HTTPS 443.
  3. Inicio de sesión exitoso en la consola web con las credenciales de administrador asignadas.
  * *Recomendación de Seguridad:* En entornos de producción, rotar inmediatamente la contraseña predeterminada e implementar certificados firmados por una Autoridad Certificadora (CA) de confianza.
<img width="830" height="639" alt="image" src="https://github.com/user-attachments/assets/cfa08299-0de4-4909-b502-fbb8c9f8a7a0" />



### Incidencia 4: Agotamiento de Espacio en Disco Virtual (`VERR_DISK_FULL`)

* **Descripción del Problema:** Tras un tiempo de funcionamiento del servidor, la máquina virtual de Wazuh se detuvo de manera crítica con el siguiente error de VirtualBox:  
&gt; *The I/O cache encountered an error while updating data in medium "ahci-0-0" (rc=VERR\_DISK\_FULL). Make sure there is enough free space on the disk and that the disk is working properly.*
* **Diagnóstico Técnico:** El proceso de indexación de Wazuh y los registros del sistema llenaron la capacidad asignada al disco virtual dinámico, impidiendo escrituras en el medio de almacenamiento y congelando el controlador AHCI.
* **Resolución:**  
  1. Reinstalación limpia del sistema operativo base (Ubuntu Server) con un tamaño de disco virtual sobredimensionado para absorber la ingesta de índices.
  2. Implementación de monitoreo de espacio en disco con `df -h` para controlar el crecimiento de los directorios de indexación en `/var/ossec/` y Elasticsearch/Indexer.
  3. Configuración de rotación de logs para evitar saturación de almacenamiento a mediano plazo.

---

## 4\. Despliegue y Enrolamiento de Agentes

1. **Procedimiento de Instalación:**  
  * En cada máquina cliente se ejecutó el instalador del agente Wazuh correspondiente a su arquitectura y sistema operativo.
  * Se apuntó la variable de configuración a la dirección IP LAN estática del servidor central Wazuh.
2. **Registro y Verificación:**  
  * Inicio de los servicios del agente (`systemctl start wazuh-agent` en Linux / inicio de servicio en Windows).
  * Verificación en el Dashboard: Comprobación de que los tres nodos (Ubuntu Desktop, Windows 11 y Host físico) reportaban estado **Active** con transmisión continua de métricas e integridad del sistema (FIM).

<img width="1497" height="934" alt="captura-instalacion-agentewazuh-windows" src="https://github.com/user-attachments/assets/2c94d38b-1772-45db-8d16-723a2fbdbf49" />
<img width="581" height="450" alt="captura-instalacion-agentewazuh-windows-2" src="https://github.com/user-attachments/assets/fc65b56e-7610-47c3-8689-6890c4d6321d" />
*Asistente de instalación de agente en Windows finalizado.*

<img width="1258" height="202" alt="image" src="https://github.com/user-attachments/assets/0f84d9e0-8a8a-4ec1-bf8e-3e42a5f70801" />
*Agente corriendo en uno de los clientes, proceso TOP en Linux.*

---

<img width="1873" height="969" alt="image" src="https://github.com/user-attachments/assets/4c165e25-81d7-4587-a509-d1a98d30edc5" />
<img width="1920" height="886" alt="dashboard-3-agentes-agregados" src="https://github.com/user-attachments/assets/a62ca490-7940-4c2e-8274-71047522d728" />

*Acceso a Dashboard y agentes agregados al Sistema*


## 5\. Simulación de Amenaza y Automatización de Respuesta Activa

### 5.1\. Vector de Ataque: Fuerza Bruta SSH

* Se generó tráfico de autenticación no autorizada de forma masiva contra el puerto 22 del servidor objetivo.
* Los intentos fallidos fueron registrados de inmediato en `/var/log/auth.log`:  
  * Eventos de contraseña fallida para usuarios existentes e inexistentes.
  * Ingesta automática por el módulo de decodificación y análisis de Wazuh.

### 5.2\. Correlación de Reglas y Detección

* Wazuh disparó alertas de nivel medio y alto basadas en sus reglas internas de autenticación:  
  * **Regla 5710:** Intento de acceso utilizando usuario no registrado.
  * **Regla 5716 / 5720:** Múltiples fallos consecutivos de autenticación SSH provenientes de una misma dirección IP origen.

<img width="1919" height="878" alt="alertas-mientras-ocurre-bruceforceattack" src="https://github.com/user-attachments/assets/b4bfb781-2288-4c80-b254-4c31f163c6f3" />

### 5.3\. Configuración de Active Response (Bloqueo Automático)

Para mitigar el ataque sin intervención manual de un operador, se enlazó la regla de detección con el script de mitigación en `ossec.conf`:

* **Acción ejecutada:** Script predeterminado `firewall-drop`.
* **Criterio de activación:** Superación del umbral de intentos fallidos en las reglas 5716 / 5720.
* **Tiempo de retención (*timeout*):** Bloqueo temporal por 600 segundos (10 minutos) en la tabla del firewall del sistema operativo anfitrión.
* **Resultado Obtenido:** La dirección IP atacante fue añadida automáticamente a las reglas de denegación del firewall local, interrumpiendo de forma instantánea la sesión de ataque y protegiendo el servicio SSH.

<img width="975" height="847" alt="ataque-detectado-detalles-aleta" src="https://github.com/user-attachments/assets/bbf96a5b-c393-49ba-ac0a-17b3d73c3426" />

*Detalles de la alerta generada tras el ataque*

---

## 6\. Conclusiones y Próximos Pasos

* **Validación Práctica:** El laboratorio demostró la viabilidad de implementar una arquitectura SIEM integral en un entorno controlado, transitando desde la resolución de problemas de red y almacenamiento hasta la automatización defensiva.
* **Habilidades Desarrolladas:** Administración de sistemas Linux, configuración de networking virtualizado (Bridge), gestión de almacenamiento en hipervisores, parseo de logs de auditoría y respuesta automatizada ante incidentes.
* **Evolución Futura:**  
  * Integración de una máquina vulnerable (**Metasploitable**) para monitorizar vectores de explotación web y de red.
  * Despliegue de **Sysmon** en la máquina Windows 11 para enriquecer la telemetría de creación de procesos y conexiones de red.
