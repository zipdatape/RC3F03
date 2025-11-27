graph TD
    %% Inicio
    A[Inicio de Sesión] -->|Credenciales Únicas| B{Rol de Usuario}

    ---
    %% Flujo Agente (Ciclo de Tipificación)
    B -->|Agente| C[Panel de Búsqueda]
    C --> D{Ingresar ID/DNI}
    D -->|Buscar| E[Consultar Backend]

    %% Ramificación de Consulta
    E -->|Cliente No Encontrado| F[Alerta Visual]
    E -->|Cliente Encontrado| G[Mostrar Datos Cliente + Historial]

    %% Tipificación
    G --> H[Formulario de Tipificación]
    H --> I[Seleccionar Categoría/Subcategoría]
    I --> J[Agregar Comentarios]
    J --> K[Guardar Tipificación]
    K -->|Insertar en BD| L[Confirmación Éxito]
    
    L --> C 
    %% Retorno al Panel de Búsqueda para el siguiente ciclo
    
    ---
    %% Flujo Administrador (Métricas y Gestión)
    B -->|Administrador| M[Dashboard de Métricas]

    %% Opciones de Dashboard
    M --> N[Ver Tabla de Tipificaciones]
    M --> O[Ver Auditoría de Consultas]
    M --> R[Gestión de Base de Datos]

    %% Sub-flujo de Reportes
    N --> P[Filtrar por Fecha/Agente]
    P --> Q[Descargar Reporte (CSV/Excel)]

    %% Sub-flujo de Gestión de BD
    R --> S{Subir Archivo CSV/Excel}
    S -->|Validar Formato| T[Procesar Carga Masiva]
    T -->|Insertar/Actualizar| U[Actualizar BD Clientes]

    %% Conexiones Finales (Opcional, para cerrar el flujo Admin)
    Q --> Z[Fin de Tarea]
    O --> Z
