# Diagrama de Flujo - Sistema de Auto-Sincronización de Usuarios (Mermaid)

## Flujo Principal del Sistema

```mermaid
flowchart TD
    A[🚀 Inicio Auto-Sync] --> B[📊 Consulta GLPI Database]
    B --> C{¿Equipos nuevos encontrados?}
    C -->|No| D[✅ Sincronización completada<br/>Sin cambios]
    C -->|Sí| E[📋 Procesar cada equipo]
    
    E --> F[🔍 Extraer datos del equipo]
    F --> G{¿Datos válidos?}
    G -->|No| H[⚠️ Equipo omitido<br/>Datos incompletos]
    G -->|Sí| I[🌐 Consultar Gesinfra API]
    
    I --> J{¿Usuario encontrado en Gesinfra?}
    J -->|No| K[⚠️ Usuario no encontrado<br/>en Gesinfra]
    J -->|Sí| L[🔄 Transformar datos Gesinfra]
    
    L --> M[📦 Preparar payload para inventario]
    M --> N[🚀 Registrar en Inventory API]
    N --> O{¿Registro exitoso?}
    O -->|No| P[❌ Error en registro<br/>de inventario]
    O -->|Sí| Q[✅ Usuario registrado<br/>exitosamente]
    
    H --> R[📊 Generar reporte final]
    K --> R
    P --> R
    Q --> R
    R --> S[📧 Enviar notificación]
    S --> T[🏁 Fin del proceso]
    D --> T
    
    style A fill:#e1f5fe
    style T fill:#e8f5e8
    style Q fill:#e8f5e8
    style H fill:#fff3e0
    style K fill:#fff3e0
    style P fill:#ffebee
    style D fill:#f3e5f5
```

## Dependencias Críticas del GLPI

```mermaid
graph TD
    A[📋 Equipo en GLPI] --> B{¿numero_usuario_alternativo existe?}
    B -->|No| C[❌ CRÍTICO<br/>Equipo no se procesa]
    B -->|Sí| D{¿estado válido?}
    
    D -->|No| E[❌ CRÍTICO<br/>Estado incorrecto]
    D -->|Sí| F{¿fecha_creacion válida?}
    
    F -->|No| G[❌ CRÍTICO<br/>Fecha incorrecta]
    F -->|Sí| H{¿usuario tiene nombre?}
    
    H -->|No| I[⚠️ IMPORTANTE<br/>Usuario sin nombre]
    H -->|Sí| J{¿modelo especificado?}
    
    J -->|No| K[⚠️ IMPORTANTE<br/>Modelo faltante]
    J -->|Sí| L[✅ Equipo válido<br/>Listo para procesar]
    
    C --> M[📊 Reporte de errores]
    E --> M
    G --> M
    I --> M
    K --> M
    L --> N[🔄 Continuar con Gesinfra]
    
    style A fill:#e3f2fd
    style L fill:#e8f5e8
    style C fill:#ffebee
    style E fill:#ffebee
    style G fill:#ffebee
    style I fill:#fff3e0
    style K fill:#fff3e0
    style M fill:#fce4ec
    style N fill:#e1f5fe
```

## Estados Válidos en GLPI

```mermaid
graph LR
    A[📊 Estados en GLPI] --> B[✅ En uso]
    A --> C[✅ Asignado]
    A --> D[❌ En almacén]
    A --> E[❌ Fuera de servicio]
    A --> F[❌ Pendiente]
    
    B --> G[🔄 Se procesa]
    C --> G
    D --> H[⏭️ Se omite]
    E --> H
    F --> H
    
    style B fill:#e8f5e8
    style C fill:#e8f5e8
    style D fill:#ffebee
    style E fill:#ffebee
    style F fill:#ffebee
    style G fill:#e1f5fe
    style H fill:#f5f5f5
```

## Flujo de APIs

```mermaid
sequenceDiagram
    participant GLPI as 📊 GLPI Database
    participant API as 🔄 Auto-Sync API
    participant Gesinfra as 🌐 Gesinfra API
    participant Inventory as 📦 Inventory API
    participant App as 💾 App Licencias
    
    API->>GLPI: 1. Consulta equipos nuevos
    GLPI-->>API: 2. Lista de equipos
    
    loop Para cada equipo
        API->>API: 3. Validar datos GLPI
        API->>Gesinfra: 4. POST /api/users/search
        Note over API,Gesinfra: Body: {"username": "50800998"}
        Gesinfra-->>API: 5. Datos de licenciamiento
        
        API->>API: 6. Transformar datos
        API->>Inventory: 7. POST /api/v1/licenses/bulk
        Note over API,Inventory: Body: {"licenses": [...], "codigo_proyecto": "51238", "autoDebit": true}
        Inventory-->>API: 8. Confirmación de registro
        
        API->>App: 9. Actualizar base de datos
    end
    
    API->>API: 10. Generar reporte final
```

## Estructura de Datos

```mermaid
graph TD
    A[📋 Datos GLPI] --> B[🔍 Extracción]
    B --> C[📊 Datos Base]
    
    C --> D[codigo_sap: numero_usuario_alternativo]
    C --> E[usuario: nombre completo]
    C --> F[equipo: modelo, ubicacion, tipo]
    C --> G[fechas: creacion, modificacion]
    
    H[🌐 Consulta Gesinfra] --> I[🔄 Transformación]
    I --> J[📦 Datos Gesinfra]
    
    J --> K[username: usuario corto]
    J --> L[tipo_licencia: O365_M365F3]
    J --> M[estado: activo]
    J --> N[correo: email del usuario]
    
    C --> O[📦 Payload Final]
    J --> O
    
    O --> P[codigo_sap: 50800998]
    O --> Q[nombre_licencia: O365_M365F3]
    O --> R[usuario_asignado: ambard]
    O --> S[tipo_licencia: O365_M365F3]
    O --> T[estado: activa]
    O --> U[codigo_proyecto: 51238]
    O --> V[codigo_lote: 2]
    O --> W[observaciones: Auto-registro desde GLPI]
    
    style A fill:#e3f2fd
    style H fill:#e8f5e8
    style O fill:#fff3e0
    style P fill:#f3e5f5
    style Q fill:#f3e5f5
    style R fill:#f3e5f5
    style S fill:#f3e5f5
    style T fill:#f3e5f5
    style U fill:#f3e5f5
    style V fill:#f3e5f5
    style W fill:#f3e5f5
```

## Manejo de Errores

```mermaid
flowchart TD
    A[🚨 Error Detectado] --> B{¿Tipo de error?}
    
    B -->|GLPI| C[📊 Error en GLPI]
    B -->|Gesinfra| D[🌐 Error en Gesinfra]
    B -->|Inventario| E[📦 Error en Inventario]
    
    C --> F[❌ numero_usuario_alternativo vacío]
    C --> G[❌ Estado incorrecto]
    C --> H[❌ Fecha incorrecta]
    C --> I[❌ Usuario sin nombre]
    
    D --> J[❌ Código SAP no encontrado]
    D --> K[❌ Token inválido]
    D --> L[❌ Timeout de conexión]
    D --> M[❌ Respuesta malformada]
    
    E --> N[❌ Código proyecto inválido]
    E --> O[❌ Código lote inválido]
    E --> P[❌ Token inválido]
    E --> Q[❌ Estructura incorrecta]
    
    F --> R[📝 Log de error]
    G --> R
    H --> R
    I --> R
    J --> R
    K --> R
    L --> R
    M --> R
    N --> R
    O --> R
    P --> R
    Q --> R
    
    R --> S[⚠️ Continuar con siguiente equipo]
    S --> T[📊 Reporte final con errores]
    
    style A fill:#ffebee
    style C fill:#ffebee
    style D fill:#ffebee
    style E fill:#ffebee
    style F fill:#ffcdd2
    style G fill:#ffcdd2
    style H fill:#ffcdd2
    style I fill:#ffcdd2
    style J fill:#ffcdd2
    style K fill:#ffcdd2
    style L fill:#ffcdd2
    style M fill:#ffcdd2
    style N fill:#ffcdd2
    style O fill:#ffcdd2
    style P fill:#ffcdd2
    style Q fill:#ffcdd2
    style R fill:#fff3e0
    style S fill:#e8f5e8
    style T fill:#e1f5fe
```

## Métricas de Calidad

```mermaid
pie title Distribución de Errores en Auto-Sync
    "Código SAP faltante" : 40
    "Estado incorrecto" : 25
    "Fecha incorrecta" : 20
    "Usuario sin nombre" : 15
```

## Arquitectura del Sistema

```mermaid
graph TB
    subgraph "🖥️ Frontend"
        A[📱 User Auto-Sync Component]
        B[⚙️ Settings Component]
    end
    
    subgraph "🔄 API Layer"
        C[🔄 /api/users/auto-sync]
        D[🌐 /api/gesinfra/search]
        E[⚙️ /api/api-validation]
    end
    
    subgraph "💾 Database Layer"
        F[📊 GLPI Database]
        G[💾 App Database]
        H[⚙️ Configuration DB]
    end
    
    subgraph "🌐 External APIs"
        I[🌐 Gesinfra API]
        J[📦 Inventory API]
    end
    
    A --> C
    B --> E
    C --> F
    C --> I
    C --> J
    D --> I
    E --> I
    C --> G
    C --> H
    
    style A fill:#e3f2fd
    style B fill:#e3f2fd
    style C fill:#e8f5e8
    style D fill:#e8f5e8
    style E fill:#e8f5e8
    style F fill:#fff3e0
    style G fill:#fff3e0
    style H fill:#fff3e0
    style I fill:#f3e5f5
    style J fill:#f3e5f5
```

## Proceso de Validación

```mermaid
flowchart TD
    A[📋 Equipo recibido] --> B[🔍 Validar campos obligatorios]
    B --> C{¿numero_usuario_alternativo?}
    C -->|No| D[❌ Error: Código SAP faltante]
    C -->|Sí| E{¿estado válido?}
    
    E -->|No| F[❌ Error: Estado incorrecto]
    E -->|Sí| G{¿fecha_creacion?}
    
    G -->|No| H[❌ Error: Fecha faltante]
    G -->|Sí| I{¿usuario?}
    
    I -->|No| J[⚠️ Advertencia: Usuario sin nombre]
    I -->|Sí| K{¿modelo?}
    
    K -->|No| L[⚠️ Advertencia: Modelo faltante]
    K -->|Sí| M[✅ Validación exitosa]
    
    D --> N[📝 Log de error]
    F --> N
    H --> N
    J --> O[📝 Log de advertencia]
    L --> O
    M --> P[🔄 Continuar procesamiento]
    
    N --> Q[⏭️ Omitir equipo]
    O --> P
    P --> R[🌐 Consultar Gesinfra]
    
    style A fill:#e3f2fd
    style M fill:#e8f5e8
    style D fill:#ffebee
    style F fill:#ffebee
    style H fill:#ffebee
    style J fill:#fff3e0
    style L fill:#fff3e0
    style N fill:#fce4ec
    style O fill:#fff8e1
    style P fill:#e1f5fe
    style Q fill:#f5f5f5
    style R fill:#e8f5e8
```

## Flujo de Configuración

```mermaid
flowchart TD
    A[⚙️ Configuración del Sistema] --> B[🔧 Variables de Entorno]
    B --> C[🌐 Gesinfra API]
    B --> D[📦 Inventory API]
    B --> E[📊 GLPI Database]
    
    C --> F[URL: https://gesinfra.example.com]
    C --> G[Token: Bearer token]
    C --> H[Timeout: 30 segundos]
    
    D --> I[URL: https://inventory.example.com]
    D --> J[Token: Bearer token]
    D --> K[Endpoint: /api/v1/licenses/bulk]
    
    E --> L[Host: localhost]
    E --> M[Port: 3306]
    E --> N[Database: glpi]
    E --> O[User: glpi_user]
    E --> P[Password: glpi_password]
    
    F --> Q[✅ Configuración completa]
    G --> Q
    H --> Q
    I --> Q
    J --> Q
    K --> Q
    L --> Q
    M --> Q
    N --> Q
    O --> Q
    P --> Q
    
    style A fill:#e3f2fd
    style C fill:#e8f5e8
    style D fill:#e8f5e8
    style E fill:#e8f5e8
    style Q fill:#e8f5e8
```

## Resumen de Dependencias

```mermaid
mindmap
  root((Sistema Auto-Sync))
    GLPI Database
      numero_usuario_alternativo
        CRÍTICO
        Código SAP
      estado
        CRÍTICO
        En uso o Asignado
      fecha_creacion
        CRÍTICO
        Fecha válida
      usuario
        IMPORTANTE
        Nombre completo
      modelo
        IMPORTANTE
        Modelo del equipo
    Gesinfra API
      Código SAP válido
      Token de autenticación
      Username disponible
      Tipo de licencia
    Inventory API
      Código proyecto válido
      Código lote válido
      Token de autenticación
      Estructura correcta
    App Licencias
      Base de datos
      Tabla app_licencias
      Campos requeridos
```

---

**Nota**: Estos diagramas Mermaid pueden ser renderizados en cualquier editor que soporte Mermaid (como GitHub, GitLab, o editores online) para visualizar el flujo completo del sistema de auto-sincronización.
