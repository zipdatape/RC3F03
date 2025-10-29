```mermaid
%% Visión general muy simple del sistema
flowchart LR
  U[Usuario inicia sincronización] --> S[Sistema de Integración]

  subgraph Fuentes Externas
    L[Licencias (PostgreSQL)<br/>Ej: usuario: juan.perez]
    P[Proyectos (PostgreSQL)<br/>Ej: PROY-1001]
    X[Datos Excel (PostgreSQL)<br/>Ej: Hoja: Recursos]
    E[Equipos (MySQL)<br/>Ej: Laptop: HP ProBook]
  end

  S --> L
  S --> P
  S --> X
  S --> E

  S --> V[Valida con código SAP<br/>Ej: 12345]
  V --> D{¿Existe en Inventario?}
  D -- Sí --> ACT[Actualizar datos]
  D -- No --> INS[Crear registro]

  subgraph Inventario (Destino)
    INV[Licencias y Equipos]
  end

  ACT --> INV
  INS --> INV

  INV --> R[Resumen simple<br/>Ej: 150 nuevas, 80 actualizadas]
```

```mermaid
%% Secuencia resumida con datos de ejemplo
sequenceDiagram
  participant U as Usuario
  participant S as Sistema
  participant L as Licencias (PG)
  participant P as Proyectos (PG)
  participant E as Equipos (MySQL)
  participant I as Inventario

  U->>S: Iniciar sincronización
  S->>L: Buscar usuarios (ej: juan.perez)
  S->>P: Obtener proyecto de SAP 12345
  S->>E: Consultar laptop del usuario
  S->>S: Validar estado y tipo
  alt Registro ya existe
    S->>I: Actualizar licencia/equipo
  else No existe
    S->>I: Crear licencia/equipo
  end
  I-->>S: Resumen (ok/errores)
  S-->>U: Mostrar resultados
```

```mermaid
%% Decisiones principales (no técnico)
flowchart TB
  A[Datos leídos: usuario=juan.perez<br/>SAP=12345<br/>Proyecto=PROY-1001<br/>Equipo=HP ProBook]
  A --> B{¿Tiene proyecto?}
  B -- Sí --> C{¿Coincide con Inventario?}
  B -- No --> D[Asignar proyecto por defecto<br/>Ej: 51238]
  C -- Sí --> E[Actualizar estado/costo/tipo]
  C -- No --> F[Crear nuevo registro]
  D --> F
  E --> G[Registrar como Actualizado]
  F --> H[Registrar como Nuevo]
```

```mermaid
%% Experiencia del usuario (journey)
journey
  title Experiencia del usuario no técnico
  section Antes
    Prepara archivo o pulsa "Sincronizar": 3: Usuario
  section Durante
    El sistema lee datos externos: 4: Sistema
    Valida por código SAP y reglas simples: 4: Sistema
    Actualiza o crea en Inventario: 4: Sistema
  section Después
    Muestra resumen claro (nuevos/actualizados/errores): 5: Usuario
```

```mermaid
%% Resultados de ejemplo (pie)
pie showData
  title Resultado de una sincronización (ejemplo)
  "Licencias nuevas" : 120
  "Licencias actualizadas" : 70
  "Laptops nuevas" : 45
  "Laptops actualizadas" : 30
  "Errores" : 5
```

```mermaid
%% Frecuencia y tiempos (gantt)
gantt
  title Frecuencias típicas de sincronización (ejemplo)
  dateFormat  HH:mm
  axisFormat  %H:%M
  section Diario
  Licencias (rápido)        :done,    09:00, 30m
  Laptops (medio)           :active,  10:00, 45m
  section Semanal
  Proyectos (revisión)      :         11:00, 1h
  Monitores (por lotes)     :         14:00, 45m
```

```mermaid
%% Estados simplificados de datos
stateDiagram-v2
  [*] --> Nuevo
  Nuevo --> Validado: Reglas SAP ok
  Validado --> EnInventario: Se creó
  EnInventario --> Actualizado: Cambios detectados
  EnInventario --> SinCambios: Igual que origen
  Validado --> Error: Datos incompletos
  Error --> [*]
  Actualizado --> [*]
  SinCambios --> [*]
```

```mermaid
%% Flujo de carga de Excel (resumen)
flowchart LR
  A[Usuario sube Excel<br/>Hoja: Recursos] --> B[Sistema procesa filas]
  B --> C{¿Fila válida?}
  C -- Sí --> D[Guardar en base de datos de Excel]
  C -- No --> E[Marcar fila con error]
  D --> F[Usar en sincronización de proyectos]
  E --> G[Mostrar errores al usuario]
```

```mermaid
%% Sincronizaciones: Licencias, Laptops y Proyectos (resumen no técnico)
flowchart LR
  subgraph Fuentes
    L[Licencias (PostgreSQL)<br/>Usuarios y estados]
    EQ[Equipos (MySQL)<br/>Laptops por usuario]
    PR[Proyectos (PostgreSQL)<br/>Tabla proyectos]
    XL[Excel (PostgreSQL)<br/>Recursos importados]
    CFG[Configuración (MySQL)<br/>sync_configuration]
    LT[Tablas de tipos y precios<br/>license_types / equipment_types]
  end

  S[Sistema de Integración] --> L
  S --> EQ
  S --> PR
  S --> XL
  S --> CFG
  S --> LT

  L & EQ & PR & XL & CFG & LT --> V[Validar por código SAP<br/>Reglas simples de negocio]

  V --> MAP[Mapear estado/tipo<br/>Calcular precio]
  MAP --> INV{¿Existe en Inventario?}
  INV -- Sí --> UPD[Actualizar datos]
  INV -- No --> INS[Crear nuevo]

  subgraph Destino
    I1[Inventario - Licencias]
    I2[Inventario - Laptops]
  end

  UPD --> I1 & I2
  INS --> I1 & I2
```

```mermaid
%% Secuencia con detalle de validaciones y precios (ejemplos)
sequenceDiagram
  participant U as Usuario (inicia)
  participant S as Sistema
  participant L as Licencias (PG)
  participant EQ as Equipos (MySQL)
  participant P as Proyectos/Excel (PG)
  participant T as Tipos/Precios
  participant I as Inventario

  U->>S: Sincronizar Licencias y Laptops
  Note over S: SAP del usuario = 12345

  S->>L: Obtener estado y tipo de licencia
  S->>EQ: Obtener laptop del usuario
  S->>P: Buscar proyecto(s) por SAP
  S->>T: Leer reglas de precio/Tipo

  S->>S: Validar SAP y completar faltantes
  S->>S: Mapear estado (ej.: Activo -> activa)
  S->>S: Calcular precio (ej.: tipo_licencia A = 25 USD)

  alt Registro ya existe en Inventario
    S->>I: Actualizar estado, tipo, costo, proyecto
  else No existe
    S->>I: Crear licencia y/o laptop
  end

  I-->>S: Resumen (nuevos/actualizados/errores)
  S-->>U: Mostrar resultados
```

```mermaid
%% Precios y reglas en términos simples
flowchart TB
  A[Entrada: tipo y características] --> B{¿Es licencia?}
  B -- Sí --> L1[Buscar en license_types<br/>Precio por tipo]
  B -- No --> C{¿Es laptop?}
  C -- Sí --> E1[Aplicar equipment_types<br/>Reglas por procesador/memoria]
  C -- No --> X[Otros casos]
  L1 --> P$[Precio final]
  E1 --> P$
  X --> P$
```


