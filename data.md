# 📊 Diagrama de Flujo - Módulo de Inventario de Materiales

## 🎯 Presentación del Sistema

---

## Slide 1: Vista General del Módulo

```mermaid
graph TB
    Start([Módulo de Inventario<br/>de Materiales]) --> GM[Gestión de Materiales]
    Start --> MS[Movimientos de Stock]
    Start --> SM[Solicitudes de Materiales]
    Start --> DM[Devoluciones de Materiales]
    
    GM --> CRUD[CRUD de Materiales]
    GM --> Stock[Control de Stock]
    GM --> Status[Estados: Disponible/Bajo/Sin Stock]
    
    MS --> Entrada[Entrada de Materiales]
    MS --> Salida[Salida de Materiales]
    MS --> Movimientos[Registro de Movimientos]
    
    SM --> CrearSolicitud[Crear Solicitud]
    SM --> Aprobar[Aprobar/Rechazar]
    SM --> Entregar[Completar Entrega]
    
    DM --> CrearDevolucion[Crear Devolución]
    DM --> AprobarDevolucion[Aprobar Devolución]
    DM --> AceptarDevolucion[Aceptar Devolución]
    
    style Start fill:#2563eb,stroke:#1e40af,stroke-width:3px,color:#fff
    style GM fill:#10b981,stroke:#059669,stroke-width:2px,color:#fff
    style MS fill:#f59e0b,stroke:#d97706,stroke-width:2px,color:#fff
    style SM fill:#8b5cf6,stroke:#7c3aed,stroke-width:2px,color:#fff
    style DM fill:#ef4444,stroke:#dc2626,stroke-width:2px,color:#fff
```

---

## Slide 2: Gestión de Materiales (CRUD)

```mermaid
flowchart TD
    Start([Gestión de Materiales]) --> Crear[Crear Material]
    Start --> Editar[Editar Material]
    Start --> Eliminar[Eliminar Material]
    Start --> Consultar[Consultar Materiales]
    
    Crear --> FormCrear[Formulario de Creación]
    FormCrear --> ValidarCrear{Validar Datos}
    ValidarCrear -->|Válido| GuardarCrear[Guardar en BD]
    ValidarCrear -->|Inválido| ErrorCrear[Mostrar Error]
    GuardarCrear --> NotificarCrear[Notificación de Éxito]
    
    Editar --> Seleccionar[Seleccionar Material]
    Seleccionar --> FormEditar[Formulario de Edición]
    FormEditar --> ValidarEditar{Validar Datos}
    ValidarEditar -->|Válido| GuardarEditar[Actualizar en BD]
    ValidarEditar -->|Inválido| ErrorEditar[Mostrar Error]
    GuardarEditar --> NotificarEditar[Notificación de Éxito]
    
    Eliminar --> Confirmar{Confirmar Eliminación}
    Confirmar -->|Sí| VerificarStock{Stock = 0?}
    VerificarStock -->|Sí| EliminarBD[Eliminar de BD]
    VerificarStock -->|No| ErrorStock[Error: Stock debe ser 0]
    Confirmar -->|No| Cancelar[Cancelar]
    EliminarBD --> NotificarEliminar[Notificación de Éxito]
    
    Consultar --> Filtros[Aplicar Filtros]
    Filtros --> Buscar[Buscar por Nombre/Código]
    Filtros --> Categoria[Filtrar por Categoría]
    Filtros --> Estado[Filtrar por Estado]
    Buscar --> Lista[Listar Resultados]
    Categoria --> Lista
    Estado --> Lista
    
    style Start fill:#2563eb,stroke:#1e40af,stroke-width:3px,color:#fff
    style Crear fill:#10b981,stroke:#059669,stroke-width:2px,color:#fff
    style Editar fill:#3b82f6,stroke:#2563eb,stroke-width:2px,color:#fff
    style Eliminar fill:#ef4444,stroke:#dc2626,stroke-width:2px,color:#fff
    style Consultar fill:#8b5cf6,stroke:#7c3aed,stroke-width:2px,color:#fff
```

---

## Slide 3: Movimientos de Stock

```mermaid
flowchart TD
    Start([Movimiento de Stock]) --> Tipo{Tipo de Movimiento}
    
    Tipo -->|Entrada| Entrada[Entrada de Materiales]
    Tipo -->|Salida| Salida[Salida de Materiales]
    
    Entrada --> FormEntrada[Formulario de Entrada]
    FormEntrada --> DatosEntrada[Cantidad + Motivo + Usuario]
    DatosEntrada --> ValidarEntrada{Validar Cantidad > 0}
    ValidarEntrada -->|Válido| CalcularEntrada[Stock Actual + Cantidad]
    ValidarEntrada -->|Inválido| ErrorEntrada[Error: Cantidad inválida]
    CalcularEntrada --> ActualizarEntrada[Actualizar Stock en BD]
    ActualizarEntrada --> RegistrarMovimientoEntrada[Registrar Movimiento]
    RegistrarMovimientoEntrada --> VerificarMinimoEntrada{Stock <= Mínimo?}
    VerificarMinimoEntrada -->|Sí| NotificarBajoEntrada[Notificar Stock Bajo]
    VerificarMinimoEntrada -->|No| NotificarEntrada[Notificación de Entrada]
    NotificarBajoEntrada --> NotificarEntrada
    
    Salida --> FormSalida[Formulario de Salida]
    FormSalida --> DatosSalida[Cantidad + Motivo + Usuario]
    DatosSalida --> ValidarSalida{Validar Cantidad <= Stock}
    ValidarSalida -->|Válido| CalcularSalida[Stock Actual - Cantidad]
    ValidarSalida -->|Inválido| ErrorSalida[Error: Stock insuficiente]
    CalcularSalida --> ActualizarSalida[Actualizar Stock en BD]
    ActualizarSalida --> RegistrarMovimientoSalida[Registrar Movimiento]
    RegistrarMovimientoSalida --> VerificarMinimoSalida{Stock <= Mínimo?}
    VerificarMinimoSalida -->|Sí| NotificarBajoSalida[Notificar Stock Bajo]
    VerificarMinimoSalida -->|No| NotificarSalida[Notificación de Salida]
    NotificarBajoSalida --> NotificarSalida
    
    NotificarEntrada --> Fin([Fin])
    NotificarSalida --> Fin
    
    style Start fill:#2563eb,stroke:#1e40af,stroke-width:3px,color:#fff
    style Entrada fill:#10b981,stroke:#059669,stroke-width:2px,color:#fff
    style Salida fill:#ef4444,stroke:#dc2626,stroke-width:2px,color:#fff
    style Fin fill:#6b7280,stroke:#4b5563,stroke-width:2px,color:#fff
```

---

## Slide 4: Flujo de Solicitudes de Materiales

```mermaid
flowchart TD
    Start([Usuario Solicita Material]) --> CrearSolicitud[Crear Solicitud]
    
    CrearSolicitud --> VerificarAprobacion{Material requiere<br/>Aprobación?}
    
    VerificarAprobacion -->|No| AutoAprobado[Aprobación Automática]
    AutoAprobado --> NotificarManagerAuto[Notificar Responsable Almacén]
    NotificarManagerAuto --> EstadoAutoAprobado[Estado: APPROVED]
    
    VerificarAprobacion -->|Sí| EstadoPendiente[Estado: PENDING]
    EstadoPendiente --> NotificarJefe[Notificar Jefe de Área]
    
    NotificarJefe --> Preparar{Responsable Almacén<br/>Prepara?}
    Preparar -->|Sí| EstadoPreparacion[Estado: IN_PREPARATION]
    EstadoPreparacion --> EstadoEsperaAprobacion[Estado: AWAITING_APPROVAL]
    EstadoEsperaAprobacion --> NotificarJefeAprobacion[Notificar Jefe de Área]
    
    Preparar -->|No| JefeAprueba{Jefe de Área<br/>Aprueba?}
    JefeAprueba -->|Sí| EstadoAprobado[Estado: APPROVED]
    JefeAprueba -->|No| EstadoRechazado[Estado: REJECTED]
    EstadoRechazado --> NotificarRechazo[Notificar Solicitante]
    NotificarRechazo --> FinRechazo([Fin - Rechazado])
    
    NotificarJefeAprobacion --> JefeAprueba
    
    EstadoAprobado --> NotificarManager[Notificar Responsable Almacén]
    EstadoAutoAprobado --> CompletarEntrega{Responsable Almacén<br/>Completa Entrega?}
    NotificarManager --> CompletarEntrega
    
    CompletarEntrega -->|Sí| DebitarStock[Debitar Stock del Material]
    DebitarStock --> RegistrarMovimiento[Registrar Movimiento de Salida]
    RegistrarMovimiento --> EstadoCompletado[Estado: COMPLETED]
    EstadoCompletado --> NotificarCompletado[Notificar Solicitante]
    NotificarCompletado --> FinExitoso([Fin - Completado])
    
    CompletarEntrega -->|No| Esperar[Esperar Completar]
    Esperar --> CompletarEntrega
    
    style Start fill:#2563eb,stroke:#1e40af,stroke-width:3px,color:#fff
    style EstadoPendiente fill:#f59e0b,stroke:#d97706,stroke-width:2px,color:#fff
    style EstadoPreparacion fill:#3b82f6,stroke:#2563eb,stroke-width:2px,color:#fff
    style EstadoEsperaAprobacion fill:#8b5cf6,stroke:#7c3aed,stroke-width:2px,color:#fff
    style EstadoAprobado fill:#10b981,stroke:#059669,stroke-width:2px,color:#fff
    style EstadoRechazado fill:#ef4444,stroke:#dc2626,stroke-width:2px,color:#fff
    style EstadoCompletado fill:#059669,stroke:#047857,stroke-width:2px,color:#fff
    style FinExitoso fill:#10b981,stroke:#059669,stroke-width:3px,color:#fff
    style FinRechazo fill:#ef4444,stroke:#dc2626,stroke-width:3px,color:#fff
```

---

## Slide 5: Flujo de Devoluciones de Materiales

```mermaid
flowchart TD
    Start([Usuario Solicita Devolución]) --> VerificarSolicitud{Solicitud Original<br/>Completada?}
    
    VerificarSolicitud -->|No| ErrorSolicitud[Error: Solicitud no completada]
    ErrorSolicitud --> FinError([Fin - Error])
    
    VerificarSolicitud -->|Sí| VerificarUsuario{Usuario es el<br/>Solicitante Original?}
    VerificarUsuario -->|No| ErrorUsuario[Error: Solo el solicitante puede devolver]
    ErrorUsuario --> FinError
    
    VerificarUsuario -->|Sí| VerificarCantidad{Cantidad <=<br/>Cantidad Entregada?}
    VerificarCantidad -->|No| ErrorCantidad[Error: Cantidad excede lo entregado]
    ErrorCantidad --> FinError
    
    VerificarCantidad -->|Sí| CrearDevolucion[Crear Devolución]
    CrearDevolucion --> EstadoPendiente[Estado: PENDING]
    EstadoPendiente --> NotificarJefe[Notificar Jefe de Área]
    
    NotificarJefe --> JefeAprueba{Jefe de Área<br/>Aprueba?}
    JefeAprueba -->|No| EstadoRechazado[Estado: REJECTED]
    EstadoRechazado --> NotificarRechazo[Notificar Solicitante]
    NotificarRechazo --> FinRechazo([Fin - Rechazado])
    
    JefeAprueba -->|Sí| EstadoAprobado[Estado: APPROVED]
    EstadoAprobado --> NotificarManager[Notificar Responsable Almacén]
    
    NotificarManager --> ManagerAcepta{Responsable Almacén<br/>Acepta Devolución?}
    ManagerAcepta -->|No| EsperarAceptacion[Esperar Aceptación]
    EsperarAceptacion --> ManagerAcepta
    
    ManagerAcepta -->|Sí| MoverStock[Mover Stock a returnedStock]
    MoverStock --> ActualizarMaterial[Actualizar Material en BD]
    ActualizarMaterial --> RegistrarMovimiento[Registrar Movimiento]
    RegistrarMovimiento --> EstadoAceptado[Estado: ACCEPTED]
    EstadoAceptado --> NotificarAceptado[Notificar Solicitante]
    NotificarAceptado --> FinExitoso([Fin - Devolución Aceptada])
    
    style Start fill:#2563eb,stroke:#1e40af,stroke-width:3px,color:#fff
    style EstadoPendiente fill:#f59e0b,stroke:#d97706,stroke-width:2px,color:#fff
    style EstadoAprobado fill:#10b981,stroke:#059669,stroke-width:2px,color:#fff
    style EstadoRechazado fill:#ef4444,stroke:#dc2626,stroke-width:2px,color:#fff
    style EstadoAceptado fill:#059669,stroke:#047857,stroke-width:2px,color:#fff
    style FinExitoso fill:#10b981,stroke:#059669,stroke-width:3px,color:#fff
    style FinRechazo fill:#ef4444,stroke:#dc2626,stroke-width:3px,color:#fff
    style FinError fill:#ef4444,stroke:#dc2626,stroke-width:3px,color:#fff
```

---

## Slide 6: Roles y Permisos

```mermaid
graph TB
    Usuario[Usuario Regular] --> PermisoUsuario[Permiso: warehouse_requests]
    Usuario --> AccionUsuario[Acciones:<br/>- Crear solicitudes<br/>- Ver sus solicitudes<br/>- Crear devoluciones]
    
    Manager[Responsable de Almacén<br/>WAREHOUSE_MANAGER] --> PermisoManager1[Permiso: warehouse_requests]
    Manager --> PermisoManager2[Permiso: warehouse_requests_manage]
    Manager --> AccionManager[Acciones:<br/>- Ver todas las solicitudes<br/>- Preparar solicitudes<br/>- Completar entregas<br/>- Aceptar devoluciones<br/>- Gestionar materiales<br/>- Registrar movimientos]
    
    Jefe[Jefe de Área<br/>AREA_CHIEF] --> PermisoJefe1[Permiso: warehouse_requests]
    Jefe --> PermisoJefe2[Permiso: warehouse_requests_approve]
    Jefe --> AccionJefe[Acciones:<br/>- Ver solicitudes pendientes<br/>- Aprobar/Rechazar solicitudes<br/>- Aprobar/Rechazar devoluciones]
    
    Admin[Administrador] --> PermisoAdmin[Permisos: Todos]
    Admin --> AccionAdmin[Acciones:<br/>- Todas las acciones<br/>- Configuración del sistema]
    
    style Usuario fill:#3b82f6,stroke:#2563eb,stroke-width:2px,color:#fff
    style Manager fill:#10b981,stroke:#059669,stroke-width:2px,color:#fff
    style Jefe fill:#8b5cf6,stroke:#7c3aed,stroke-width:2px,color:#fff
    style Admin fill:#ef4444,stroke:#dc2626,stroke-width:2px,color:#fff
```

---

## Slide 7: Estados de Solicitudes y Devoluciones

```mermaid
stateDiagram-v2
    [*] --> PENDING: Usuario crea solicitud
    
    PENDING --> IN_PREPARATION: Responsable prepara
    PENDING --> AWAITING_APPROVAL: Responsable prepara (auto)
    PENDING --> APPROVED: Jefe aprueba directamente
    PENDING --> REJECTED: Jefe rechaza
    
    IN_PREPARATION --> AWAITING_APPROVAL: Automático
    
    AWAITING_APPROVAL --> APPROVED: Jefe aprueba
    AWAITING_APPROVAL --> REJECTED: Jefe rechaza
    
    APPROVED --> COMPLETED: Responsable completa entrega
    
    REJECTED --> [*]
    COMPLETED --> [*]
    
    note right of PENDING
        Estado inicial cuando
        requiere aprobación
    end note
    
    note right of APPROVED
        Listo para entrega
        Stock se debita al completar
    end note
    
    note right of COMPLETED
        Entrega completada
        Stock actualizado
    end note
```

---

## Slide 8: Estados de Devoluciones

```mermaid
stateDiagram-v2
    [*] --> PENDING: Usuario solicita devolución
    
    PENDING --> APPROVED: Jefe aprueba
    PENDING --> REJECTED: Jefe rechaza
    
    APPROVED --> ACCEPTED: Responsable acepta
    APPROVED --> REJECTED: Responsable rechaza
    
    REJECTED --> [*]
    ACCEPTED --> [*]
    
    note right of PENDING
        Esperando aprobación
        del Jefe de Área
    end note
    
    note right of APPROVED
        Esperando aceptación
        del Responsable Almacén
    end note
    
    note right of ACCEPTED
        Stock movido a
        returnedStock
    end note
```

---

## Slide 9: Sistema de Notificaciones

```mermaid
graph LR
    Evento[Evento del Sistema] --> NotificacionApp[Notificación en App]
    Evento --> NotificacionEmail[Email]
    
    NotificacionApp --> Campana[Campana de Notificaciones]
    Campana --> Leer[Usuario Lee]
    Leer --> MarcarLeida[Marcar como Leída]
    
    NotificacionEmail --> EnviarEmail[Enviar Email]
    EnviarEmail --> Inbox[Bandeja de Entrada]
    Inbox --> LinkAccion[Link de Acción Rápida]
    
    Eventos[Eventos Notificables:<br/>- Solicitud Creada<br/>- Solicitud Preparada<br/>- Solicitud Aprobada<br/>- Solicitud Rechazada<br/>- Solicitud Completada<br/>- Devolución Creada<br/>- Devolución Aprobada<br/>- Devolución Aceptada<br/>- Devolución Rechazada<br/>- Stock Bajo]
    
    style Evento fill:#2563eb,stroke:#1e40af,stroke-width:2px,color:#fff
    style NotificacionApp fill:#10b981,stroke:#059669,stroke-width:2px,color:#fff
    style NotificacionEmail fill:#f59e0b,stroke:#d97706,stroke-width:2px,color:#fff
```

---

## Slide 10: Control de Stock y Estados

```mermaid
flowchart TD
    Material[Material] --> StockActual[Stock Actual]
    StockActual --> Comparar{Comparar con<br/>Stock Mínimo}
    
    Comparar -->|Stock = 0| SinStock[Estado: out_of_stock<br/>🔴 Sin Stock]
    Comparar -->|Stock <= Mínimo| StockBajo[Estado: low_stock<br/>🟡 Stock Bajo]
    Comparar -->|Stock > Mínimo| Disponible[Estado: available<br/>🟢 Disponible]
    
    SinStock --> NotificarSinStock[Notificar Administradores]
    StockBajo --> NotificarBajoStock[Notificar Administradores]
    
    Movimiento[Movimiento de Stock] --> ActualizarStock[Actualizar Stock]
    ActualizarStock --> RecalcularEstado[Recalcular Estado]
    RecalcularEstado --> Comparar
    
    Devolucion[Devolución Aceptada] --> MoverReturnedStock[Mover a returnedStock]
    MoverReturnedStock --> ReducirStock[Reducir Stock Disponible]
    ReducirStock --> ActualizarStock
    
    style Material fill:#2563eb,stroke:#1e40af,stroke-width:3px,color:#fff
    style SinStock fill:#ef4444,stroke:#dc2626,stroke-width:2px,color:#fff
    style StockBajo fill:#f59e0b,stroke:#d97706,stroke-width:2px,color:#fff
    style Disponible fill:#10b981,stroke:#059669,stroke-width:2px,color:#fff
```

---

## 📝 Resumen de Funcionalidades

### Gestión de Materiales
- ✅ Crear, editar, eliminar materiales
- ✅ Control de stock (mínimo, máximo, actual)
- ✅ Estados automáticos (disponible, bajo stock, sin stock)
- ✅ Categorización y ubicación
- ✅ Gestión de proveedores

### Movimientos de Stock
- ✅ Registro de entradas y salidas
- ✅ Historial completo de movimientos
- ✅ Notificaciones automáticas de stock bajo
- ✅ Trazabilidad de cada movimiento

### Solicitudes de Materiales
- ✅ Creación de solicitudes por usuarios
- ✅ Flujo de aprobación (Jefe de Área)
- ✅ Preparación y entrega (Responsable Almacén)
- ✅ Aprobación automática para materiales sin requerimiento
- ✅ Notificaciones en cada etapa

### Devoluciones de Materiales
- ✅ Solicitud de devolución por usuarios
- ✅ Aprobación por Jefe de Área
- ✅ Aceptación por Responsable Almacén
- ✅ Gestión de stock devuelto (returnedStock)

### Sistema de Notificaciones
- ✅ Notificaciones en app (campana)
- ✅ Notificaciones por email
- ✅ Configuración de destinatarios por evento
- ✅ Links de acción rápida

---

## 🎯 Conclusión

El módulo de Inventario de Materiales proporciona un sistema completo para:
- 📦 Gestión de inventario
- 🔄 Control de movimientos
- 📋 Procesamiento de solicitudes
- 🔙 Gestión de devoluciones
- 🔔 Sistema de notificaciones integrado

Todo con control de roles y permisos para garantizar la seguridad y el flujo correcto de operaciones.

