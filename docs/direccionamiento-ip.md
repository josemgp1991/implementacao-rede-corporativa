# Esquema de Direccionamiento IP - Laboratorio HA pfSense

Este documento detalla la distribución de subredes y direccionamiento lógico utilizado para simular el entorno corporativo de alta disponibilidad.

## Matriz de Direccionamiento

| Interfaz / VLAN | Segmento | IP pfSense Principal | IP pfSense Respaldo | VIP CARP (Gateway) | Notas / Hosts Fijos |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **pfsync** (HA) | `172.16.100.0/29` | `172.16.100.1/29` | `172.16.100.2/29` | N/A | Sincronización de estados |
| **VLAN 10** (Infra) | `172.16.10.0/24` | `172.16.10.2/24` | `172.16.10.3/24` | `172.16.10.1/24` | AD DS / DNS Server: `172.16.10.10` |
| **VLAN 20** (Apps) | `172.16.20.0/24` | `172.16.20.2/24` | `172.16.20.3/24` | `172.16.20.1/24` | Web/Mail Server: `172.16.20.10` |
| **VLAN 30** (DMZ)  | `172.16.30.0/24` | `172.16.30.2/24` | `172.16.30.3/24` | `172.16.30.1/24` | Linux Dev Sandbox: `172.16.30.10` |
| **VLAN 40** (VIP)  | `172.16.40.0/24` | `172.16.40.2/24` | `172.16.40.3/24` | `172.16.40.1/24` | Contabilidad y RH (Pool DHCP) |
| **VLAN 50** (Gral) | `172.16.50.0/24` | `172.16.50.2/24` | `172.16.50.3/24` | `172.16.50.1/24` | Admin, Logística, Ventas (Pool DHCP) |

## Reglas de Asignación Importantes
1. Todos los clientes finales (VLAN 40 y 50) deben apuntar a la **VIP CARP** como su puerta de enlace predeterminada.
2. El servidor de la VLAN 10 resolverá las consultas DNS internas para toda la organización.