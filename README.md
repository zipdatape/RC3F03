# Diagrama de Flujo - Sistema de Auto-Sincronización de Usuarios

## Flujo Principal del Sistema

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                           SISTEMA DE AUTO-SYNC                                 │
└─────────────────────────────────────────────────────────────────────────────────┘

┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   GLPI Database │    │  Gesinfra API   │    │ Inventory API   │    │  App Licencias  │
│                 │    │                 │    │                 │    │                 │
│ • Equipos       │    │ • Estado        │    │ • Registro      │    │ • Licencias     │
│ • Usuarios      │    │   Licenciamiento│    │   Masivo        │    │   Activas       │
│ • Estados       │    │ • Tipo Licencia │    │ • Validación    │    │ • Asignaciones  │
│ • Fechas        │    │ • Username      │    │ • Proyectos     │    │ • Proyectos     │
└─────────────────┘    └─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │                       │
         │ 1. Consulta           │                       │                       │
         │    equipos nuevos     │                       │                       │
         ▼                       │                       │                       │
┌─────────────────────────────────────────────────────────────────────────────────┐
│ 1. CONSULTA GLPI                                                               │
│ • Filtro: fecha_creacion > última_sync                                        │
│ • Estados: "En uso", "Asignado"                                               │
│ • Requisito: numero_usuario_alternativo IS NOT NULL                           │
│ • Orden: fecha_creacion DESC                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
         │
         │ 2. Para cada equipo encontrado
         ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│ 2. EXTRACCIÓN DE DATOS                                                         │
│ • codigo_sap: numero_usuario_alternativo                                      │
│ • usuario: usuario (nombre completo)                                          │
│ • equipo: modelo, ubicacion, tipo, estado                                     │
│ • fechas: fecha_creacion, fecha_modificacion                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
         │
         │ 3. Consulta Gesinfra
         ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│ 3. CONSULTA GESINFRA                                                           │
│ • Endpoint: /api/users/search                                                 │
│ • Método: POST                                                                │
│ • Auth: Bearer Token                                                          │
│ • Body: { "username": codigo_sap }                                            │
└─────────────────────────────────────────────────────────────────────────────────┘
         │
         │ 4. Transformación de datos
         ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│ 4. TRANSFORMACIÓN GESINFRA                                                     │
│ • codigo_sap: data.exp                                                        │
│ • usuario: data.nombre (nombre completo)                                      │
│ • username: data.username (usuario corto)                                     │
│ • estado: data.status                                                         │
│ • tipo_licencia: data.detalles[].table_1[].Licencia                          │
│ • correo: data.correo                                                         │
│ • puesto: data.puesto                                                         │
│ • ubicacion: data.ou                                                          │
└─────────────────────────────────────────────────────────────────────────────────┘
         │
         │ 5. Registro en inventario
         ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│ 5. REGISTRO EN INVENTARIO                                                      │
│ • Endpoint: /api/v1/licenses/bulk                                             │
│ • Método: POST                                                                │
│ • Auth: Bearer Token                                                          │
│ • Body: {                                                                     │
│     "licenses": [{                                                            │
│       "codigo_sap": "50800998",                                               │
│       "nombre_licencia": "O365_M365F3",                                       │
│       "usuario_asignado": "ambard",                                           │
│       "tipo_licencia": "O365_M365F3",                                         │
│       "estado": "activa",                                                     │
│       "codigo_proyecto": "51238",                                             │
│       "codigo_lote": "2",                                                     │
│       "observaciones": "Auto-registro desde GLPI - Equipo: 83A1"             │
│     }],                                                                       │
│     "codigo_proyecto": "51238",                                               │
│     "autoDebit": true                                                         │
│   }                                                                           │
└─────────────────────────────────────────────────────────────────────────────────┘
         │
         │ 6. Confirmación
         ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│ 6. RESULTADO                                                                   │
│ • Usuario registrado exitosamente                                             │
│ • Licencia asignada automáticamente                                           │
│ • Proyecto y lote configurados                                                │
│ • Estado: activa                                                              │
└─────────────────────────────────────────────────────────────────────────────────┘
```

## Dependencias Críticas del GLPI

### **Campos Obligatorios en GLPI**

| Campo | Descripción | Impacto si falta |
|-------|-------------|------------------|
| `numero_usuario_alternativo` | Código SAP del usuario | **CRÍTICO** - No se procesa el equipo |
| `estado` | Estado del equipo | **CRÍTICO** - Solo procesa "En uso" y "Asignado" |
| `fecha_creacion` | Fecha de registro | **CRÍTICO** - No se identifica como nuevo |
| `usuario` | Nombre del usuario | **IMPORTANTE** - Se usa para identificación |
| `modelo` | Modelo del equipo | **IMPORTANTE** - Se incluye en observaciones |

### **Estados Válidos en GLPI**

- ✅ **"En uso"** - Equipo activo con usuario asignado
- ✅ **"Asignado"** - Equipo asignado a usuario
- ❌ **"En almacén"** - No se procesa
- ❌ **"Fuera de servicio"** - No se procesa
- ❌ **"Pendiente"** - No se procesa

### **Filtros de Búsqueda**

```sql
SELECT * FROM equipos 
WHERE fecha_creacion > '2025-01-01 00:00:00'
  AND estado IN ('En uso', 'Asignado')
  AND numero_usuario_alternativo IS NOT NULL
  AND numero_usuario_alternativo != ''
ORDER BY fecha_creacion DESC
LIMIT 100
```

## Flujo de Errores

```
┌─────────────────┐
│ Error en GLPI   │
└─────────────────┘
         │
         ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│ ERRORES COMUNES                                                                │
│ • numero_usuario_alternativo vacío o NULL                                     │
│ • Estado incorrecto (no es "En uso" o "Asignado")                            │
│ • fecha_creacion incorrecta o faltante                                        │
│ • Usuario sin nombre                                                          │
└─────────────────────────────────────────────────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│ CONSECUENCIAS                                                                  │
│ • Equipo no se procesa                                                        │
│ • Usuario no se registra en inventario                                        │
│ • Licencia no se asigna                                                       │
│ • Reporte de sincronización incompleto                                        │
└─────────────────────────────────────────────────────────────────────────────────┘
```

## Validaciones del Sistema

### **Pre-validación GLPI**
- [ ] `numero_usuario_alternativo` existe y no está vacío
- [ ] `estado` es "En uso" o "Asignado"
- [ ] `fecha_creacion` es válida
- [ ] `usuario` tiene nombre

### **Validación Gesinfra**
- [ ] Código SAP existe en Gesinfra
- [ ] Usuario tiene estado de licenciamiento
- [ ] Tipo de licencia identificado
- [ ] Username disponible

### **Validación Inventario**
- [ ] Código de proyecto válido
- [ ] Código de lote válido
- [ ] Token de autenticación válido
- [ ] Estructura de datos correcta

## Métricas de Calidad

### **Indicadores de Éxito**
- **Tasa de procesamiento**: > 95% de equipos válidos
- **Tasa de registro**: > 90% de usuarios registrados
- **Tiempo de sincronización**: < 5 minutos por 100 equipos
- **Errores de validación**: < 5% por sincronización

### **Alertas del Sistema**
- ⚠️ **Alerta**: Equipos sin `numero_usuario_alternativo`
- ⚠️ **Alerta**: Estados incorrectos en equipos
- ⚠️ **Alerta**: Usuarios no encontrados en Gesinfra
- ⚠️ **Alerta**: Fallos en registro de inventario

## Mantenimiento Preventivo

### **Tareas Diarias**
- [ ] Verificar sincronización automática
- [ ] Revisar logs de errores
- [ ] Validar nuevos equipos en GLPI

### **Tareas Semanales**
- [ ] Auditoría de datos en GLPI
- [ ] Verificación de estados de equipos
- [ ] Revisión de códigos SAP

### **Tareas Mensuales**
- [ ] Reporte de calidad de datos
- [ ] Análisis de tendencias de errores
- [ ] Optimización de consultas
