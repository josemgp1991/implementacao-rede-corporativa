## 🌐 Diagrama de la Arquitectura de Red

Este diagrama representa la topología lógica del laboratorio virtual corporativo, detallando el aislamiento de las redes locales (LAN y DMZ) a través del firewall pfSense y la salida segura hacia Internet (WAN).

```mermaid
graph TD
    %% Definición de Nodos de Red
    Internet((Internet / WAN)) <-->|IP por DHCP Automático| pfSense[Firewall pfSense]
    
    subgraph Red_Corporativa_Interna [Entorno Virtual Seguro]
        pfSense <-->|Interfaz LAN / VLAN 10| SW_LAN[Switch Virtual LAN]
        pfSense <-->|Interfaz DMZ / VLAN 20| SW_DMZ[Switch Virtual DMZ]
        
        %% Segmento LAN
        subgraph Segmento_LAN [LAN - Gestión e Infraestructura]
            SW_LAN <--> WinServer[Windows Server 2022<br>• Controlador de Dominio AD<br>• Servidor DNS / DHCP]
            SW_LAN <--> WinClient[Cliente Windows 10/11<br>• Estación de Trabajo]
        end
        
        %% Segmento DMZ
        subgraph Segmento_DMZ [DMZ - Servicios Públicos]
            SW_DMZ <--> UbuntuSrv[Ubuntu Server 24.04/26.04 LTS<br>• Entorno Full-Stack / Web]
        end
    end

    %% Estilos Visuales
    style pfSense fill:#f96,stroke:#333,stroke-width:2px
    style WinServer fill:#9cf,stroke:#333,stroke-width:1px
    style UbuntuSrv fill:#fc9,stroke:#333,stroke-width:1px
    style Internet fill:#ccc,stroke:#333,stroke-width:1px
```

### 🛠️ ¿Cómo se adapta esto a tu documentación empresarial?

1. **Formato Profesional Nativo:** Al usar ````mermaid` dentro del `README.md`, GitHub lo convertirá en un gráfico interactivo muy limpio sin necesidad de que subas imágenes pesadas.
2. **Alineación con tu configuración:** Refleja con precisión tu entorno (pfSense como core, Windows Server 2022 gestionando la LAN, y tu máquina de desarrollo web en Ubuntu).
3. **Privacidad y Simulación:** Mantiene el enfoque de seguridad corporativa al no exponer datos reales de tu red física doméstica.

<FollowUp>
¿Quieres que agreguemos en el texto del `README.md` la **tabla detallada con las IPs falsas/simuladas** y los rangos de las subredes (VLANs) para que coincida exactamente con lo que tienes configurado en tu pfSense?
</FollowUp>
