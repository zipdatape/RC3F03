```mermaid
%% Visión general muy simple del sistema
flowchart LR
  U[Usuario inicia sincronización] --> S[Sistema de Integración]

  subgraph Fuentes Externas
    L[Licencias (PostgreSQL)\nEj: usuario: juan.perez]
    P[Proyectos (PostgreSQL)\nEj: PROY-1001]
    X[Datos Excel (PostgreSQL)\nEj: Hoja: Recursos]
    E[Equipos (MySQL)\nEj: Laptop: HP ProBook]
  end

  S --> L
  S --> P
  S --> X
  S --> E

  S --> V[Valida con código SAP\nEj: 12345]
  V --> D{¿Existe en Inventario?}
  D -- Sí --> ACT[Actualizar datos]
  D -- No --> INS[Crear registro]

  subgraph Inventario (Destino)
    INV[Licencias y Equipos]\n
  end

  ACT --> INV
  INS --> INV

  INV --> R[Resumen simple\nEj: 150 nuevas, 80 actualizadas]
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
  A[Datos leídos: usuario=juan.perez\nSAP=12345\nProyecto=PROY-1001\nEquipo=HP ProBook]
  A --> B{¿Tiene proyecto?}
  B -- Sí --> C{¿Coincide con Inventario?}
  B -- No --> D[Asignar proyecto por defecto\nEj: 51238]
  C -- Sí --> E[Actualizar estado/costo/tipo]
  C -- No --> F[Crear nuevo registro]
  D --> F
  E --> G[Registrar como "Actualizado"]
  F --> H[Registrar como "Nuevo"]
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
  A[Usuario sube Excel\nHoja: Recursos] --> B[Sistema procesa filas]
  B --> C{¿Fila válida?}
  C -- Sí --> D[Guardar en base de datos de Excel]
  C -- No --> E[Marcar fila con error]
  D --> F[Usar en sincronización de proyectos]
  E --> G[Mostrar errores al usuario]
```


