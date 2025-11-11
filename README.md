# Topologías de Red

## Topología 1: Infraestructura con Equipos del Proveedor

```mermaid
graph TB
    subgraph "Empresa"
        SwitchTP["Switch TP-Link<br/>(Empresa)"]
        Equipo1["Equipo 1"]
        Equipo2["Equipo 2"]
        Equipo3["Equipo 3"]
    end
    
    subgraph "Proveedor"
        RouterProv["Router<br/>(Proveedor)"]
        SwitchAruba["Switch Aruba<br/>(Proveedor)"]
        AP1["AP 1<br/>(Proveedor)"]
        AP2["AP 2<br/>(Proveedor)"]
        AP3["AP 3<br/>(Proveedor)"]
    end
    
    Internet["Internet"]
    
    Equipo1 --> SwitchTP
    Equipo2 --> SwitchTP
    Equipo3 --> SwitchTP
    SwitchTP --> RouterProv
    RouterProv --> SwitchAruba
    SwitchAruba --> AP1
    SwitchAruba --> AP2
    SwitchAruba --> AP3
    RouterProv --> Internet
    
    style SwitchTP fill:#90EE90
    style RouterProv fill:#FFB6C1
    style SwitchAruba fill:#FFB6C1
    style AP1 fill:#FFB6C1
    style AP2 fill:#FFB6C1
    style AP3 fill:#FFB6C1
    style Equipo1 fill:#87CEEB
    style Equipo2 fill:#87CEEB
    style Equipo3 fill:#87CEEB
```

**Leyenda:**
- 🟢 Verde: Equipos de la Empresa
- 🔴 Rosa: Equipos del Proveedor
- 🔵 Azul: Equipos de Usuario

**Descripción:**
- La empresa solo posee un **Switch TP-Link** que conecta los equipos internos
- El **Router** y el **Switch Aruba** pertenecen al proveedor
- Los **APs (Access Points)** también son del proveedor y están conectados al Switch Aruba
- El Router del proveedor proporciona la conexión a Internet

---

## Topología 2: Infraestructura con Router Propio de la Empresa

```mermaid
graph TB
    subgraph "Empresa"
        RouterEmp["Router Propio<br/>(Empresa)"]
        SwitchTP["Switch TP-Link<br/>(Empresa)"]
        Equipo1["Equipo 1"]
        Equipo2["Equipo 2"]
        Equipo3["Equipo 3"]
    end
    
    subgraph "Proveedor"
        RouterProv["Router Proveedor<br/>(Proveedor)"]
        SwitchAruba["Switch Aruba<br/>(Proveedor)"]
        AP1["AP 1<br/>(Proveedor)"]
        AP2["AP 2<br/>(Proveedor)"]
        AP3["AP 3<br/>(Proveedor)"]
    end
    
    Internet["Internet"]
    
    Equipo1 --> SwitchTP
    Equipo2 --> SwitchTP
    Equipo3 --> SwitchTP
    SwitchTP --> RouterEmp
    RouterEmp --> RouterProv
    RouterProv --> SwitchAruba
    SwitchAruba --> AP1
    SwitchAruba --> AP2
    SwitchAruba --> AP3
    RouterProv --> Internet
    
    style RouterEmp fill:#90EE90
    style SwitchTP fill:#90EE90
    style RouterProv fill:#FFB6C1
    style SwitchAruba fill:#FFB6C1
    style AP1 fill:#FFB6C1
    style AP2 fill:#FFB6C1
    style AP3 fill:#FFB6C1
    style Equipo1 fill:#87CEEB
    style Equipo2 fill:#87CEEB
    style Equipo3 fill:#87CEEB
```

**Leyenda:**
- 🟢 Verde: Equipos de la Empresa
- 🔴 Rosa: Equipos del Proveedor
- 🔵 Azul: Equipos de Usuario

**Descripción:**
- La empresa posee su propio **Router** y un **Switch TP-Link**
- El Router de la empresa se conecta al **Router del Proveedor** para obtener acceso a Internet
- El **Switch Aruba** y los **APs** siguen siendo del proveedor
- Esta configuración permite a la empresa tener mayor control sobre su red interna antes de salir a Internet

---

## Comparación de Topologías

| Aspecto | Topología 1 | Topología 2 |
|---------|-------------|-------------|
| Router Empresa | ❌ No | ✅ Sí |
| Control de Red | Limitado | Mayor control |
| Gestión de Tráfico | Depende del proveedor | Control interno |
| Costo | Menor | Mayor (router propio) |
| Seguridad | Básica | Mejor (firewall propio) |

