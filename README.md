# Laboratorio de Infraestructura de Red de Alta Disponibilidad (HA) y Segmentación VLAN con pfSense y Windows Server 2022

Este repositorio contiene la documentación y registros de configuración de un entorno de red empresarial en un laboratorio virtualizado (**Oracle VirtualBox**). El diseño implementa redundancia crítica a nivel de firewall de borde, segmentación lógica de tráfico mediante VLANs y servicios centralizados de identidad e infraestructura.

## 📌 Arquitectura del Laboratorio

La topología está compuesta por tres nodos principales interconectados mediante conmutadores virtuales internos con modo promiscuo habilitado para soportar el protocolo CARP.

1. **`pfSense-Principal` (Master):** Firewall primario del clúster HA.
2. **`pfSense-Respaldo` (Backup):** Firewall secundario redundante.
3. **`Windows Server 2022`:** Servidor centralizado de servicios (Active Directory, DNS, DHCP).

---

## 🗺️ Matriz de Direccionamiento y Segmentación (VLANs)

El puerto físico `le1` de ambos firewalls actúa como un enlace troncal (*Trunk*) para transportar los siguientes segmentos:

| Red / VLAN | Propósito | IP Física (Master) | IP Física (Backup) | IP Virtual (CARP / Gateway) | Rango DHCP (Windows Server) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **LAN / VLAN 10** | Infraestructura y Servidores | `172.16.10.2 /24` | `172.16.10.3 /24` | `172.16.10.1 /24` | *Estático (Servidores)* |
| **VLAN 20** | Aplicaciones Internas | `172.16.20.2 /24` | `172.16.20.3 /24` | `172.16.20.1 /24` | *Reservado para despliegues* |
| **VLAN 30** | Zona Desmilitarizada (DMZ) | `172.16.30.2 /24` | `172.16.30.3 /24` | `172.16.30.1 /24` | *Estático* |
| **VLAN 40** | VIP (Contabilidad / RH) | `172.16.40.2 /24` | `172.16.40.3 /24` | `172.16.40.1 /24` | `172.16.40.50` - `172.16.40.250` |
| **VLAN 50** | General (Admin / Ventas) | `172.16.50.2 /24` | `172.16.50.3 /24` | `172.16.50.1 /24` | `172.16.50.50` - `172.16.50.250` |
| **PFSYNC** | Enlace de Sincronización HA | `172.16.100.1 /29`| `172.16.100.2 /29`| *No Aplica* | *No Aplica* |

---

## 🛠️ Configuraciones Implementadas

### 1. Clúster de Alta Disponibilidad (HA)
* **pfsync (Sincronización de Estados):** Configurado a través de una red interna dedicada (`vlan_pfsync`) en la tarjeta física `le2` para compartir de forma instantánea las tablas de conexiones activas entre ambos nodos.
* **XMLRPC Sync (Réplica de Configuración):** Sincronización unidireccional activada desde el nodo *Principal* hacia el nodo *Respaldo* (`172.16.100.2`), clonando de forma automática reglas de firewall, NAT, alias y subinterfaces.
* **Protocolo CARP:** Despliegue de 5 Direcciones IP Virtuales (VIPs) que comparten una dirección MAC virtual, actuando como las puertas de enlace redundantes para todos los clientes y servidores de la red.

### 2. Servicios Core (Windows Server 2022)
* **Active Directory Domain Services (AD DS):** Inicializado el dominio interno de la organización.
* **Servidor DNS:** Configurado como la autoridad máxima interna (`172.16.10.10`). Se removieron los bucles lógicos con las IPs del firewall y se establecieron reenviadores (*Forwarders*) directos a DNS públicos (`8.8.8.8` y `1.1.1.1`) para garantizar la navegación externa del dominio.
* **Servidor DHCP:** Creación y activación de ámbitos exclusivos para las redes de usuarios finales (**VLAN 40** y **VLAN 50**), inyectando el Gateway CARP (`.1`) correspondiente a cada segmento.

### 3. Enrutamiento e Interconectividad entre Segmentos
* **DHCP Relay:** Activado en pfSense sobre las interfaces de clientes (**VLAN40** y **VLAN50**), utilizando como servidor de destino la IP `172.16.10.10` en la red nativa de infraestructura para cruzar las peticiones de broadcast por encima de los límites de capa 3.
* **Control de Procesamiento HA en Servicios:** El DHCP Relay está amarrado al estado CARP de la VIP `172.16.10.1`, asegurando que el reenviador de paquetes DHCP solo se ejecute en el equipo que ostente el rol de `MASTER`.

---

## 🧪 Pruebas de Caldera Realizadas Exitosamente

### Prueba de Conmutación por Error (Failover Test)
Para validar el diseño tolerante a fallos, se ejecutó un ping continuo desde el Windows Server hacia internet (`ping 8.8.8.8 -t`) y se forzó un apagado de emergencia (*Power Off*) en el nodo `pfSense-Principal`.

* **Resultado:** El nodo `pfSense-Respaldo` detectó la caída en menos de 1 segundo, migrando instantáneamente todas las interfaces virtuales CARP de estado `BACKUP` a `MASTER`. 
* **Impacto en la Red:** Pérdida insignificante de un único paquete de datos (`Request timed out`), restableciéndose el tráfico de forma automática a través del nodo secundario sin caída de servicios ni interrupción de la interfaz web de gestión.

---

## 🚀 Próximos Pasos (Hoja de Ruta del Proyecto)

Para completar el despliegue del entorno empresarial, las siguientes fases guiarán la continuidad de la caminata tecnológica:

1. **Despliegue y Validación de Hosts Clientes:**
   * Incorporación de estaciones de trabajo virtuales (Windows 10 / Linux) conectadas al switch troncal.
   * Validación del etiquetado VLAN en los clientes y comprobación de la correcta asignación de direccionamiento automático IP y DNS vía **DHCP Relay** hacia el Windows Server.
2. **Políticas de Aislamiento y Control de Tráfico (Firewall Hardening):**
   * Configuración de reglas estrictas en pfSense para romper el enrutamiento predeterminado entre segmentos (*Inter-VLAN Routing*).
   * Restricción total de acceso de la red de usuarios generales (**VLAN 50**) hacia la zona financiera (**VLAN 40**) y el segmento de infraestructura (**VLAN 10**).
3. **Control de Acceso a Servicios Core:**
   * Apertura exclusiva de puertos críticos para la operación de Active Directory (Kerberos, LDAP, DNS, SMB) en las VLANs de clientes, bloqueando cualquier otra comunicación innecesaria hacia el Windows Server.
4. **Pruebas de Stress de Redundancia:**
   * Validación del failover CARP mientras los clientes finales consumen tráfico interactivo, asegurando un entorno tolerante a fallos de nivel empresarial.
