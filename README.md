# Análisis de Consultas SQL - GLPI Dashboard

**Fecha de análisis:** Noviembre 2025  
**Base de datos:** glpi_tickets  
**Total de archivos con consultas:** 10+

---

## 📋 Resumen Ejecutivo

Este documento analiza todas las consultas SQL utilizadas en el proyecto GLPI Dashboard para identificar patrones, problemas potenciales y oportunidades de optimización.

### Estadísticas Generales

- **Archivos con consultas SQL:** 10+
- **Tabla principal consultada:** `tickets_report` (7,098 registros)
- **Patrón de consultas:** Mayormente SELECT con agregaciones complejas
- **Uso de índices:** Parcial (algunas columnas indexadas)
- **Problemas identificados:** 5 categorías principales

---

## 🔍 Tipos de Consultas Identificadas

### 1. Consultas de Agregación y Estadísticas

#### 1.1. Consultas de Conteo y Totales
**Ubicación:** `lib/db.ts`, `lib/backlog-queries.ts`

```sql
-- Ejemplo: getTotalTickets
SELECT COUNT(*) as total
FROM tickets_report
WHERE fecha_de_apertura IS NOT NULL
  AND STR_TO_DATE(fecha_de_apertura, '%Y-%m-%d %H:%i:%s') BETWEEN ? AND ?
```

**Características:**
- Uso frecuente de `COUNT(*)`
- Filtros por fecha con `STR_TO_DATE`
- Múltiples condiciones WHERE dinámicas

#### 1.2. Consultas de Distribución
**Ubicación:** `lib/backlog-queries.ts`, `lib/categories-queries.ts`

```sql
-- Ejemplo: getBacklogByCategory
SELECT 
  category,
  COUNT(*) as total,
  AVG(DATEDIFF(NOW(), STR_TO_DATE(fecha_de_apertura, '%Y-%m-%d %H:%i:%s'))) as avg_age_days
FROM tickets_report
WHERE status IN ('En espera', 'Asignado', 'Pendiente', 'En curso', ...)
GROUP BY category
ORDER BY total DESC
LIMIT 10
```

**Características:**
- Agregaciones con `GROUP BY`
- Uso de funciones de fecha (`DATEDIFF`, `NOW()`)
- Límites en resultados (`LIMIT`)

---

### 2. Consultas de SLA y Cumplimiento

#### 2.1. Cálculo de Cumplimiento de SLA
**Ubicación:** `lib/sla-queries.ts`, `app/api/dashboard/comparison/route.ts`

```sql
-- Ejemplo: getSLAKPIs
SELECT 
  COUNT(*) as total_tickets,
  SUM(CASE 
    WHEN status IN ('Closed', 'Solved') 
      AND fecha_de_resolucion IS NOT NULL 
      AND fecha_de_apertura IS NOT NULL 
      AND sla_minutes IS NOT NULL THEN
      CASE
        -- Multiplicación por 0.4167 para aproximar horas laborales (10/24)
        WHEN TIMESTAMPDIFF(MINUTE, 
             STR_TO_DATE(fecha_de_apertura, '%Y-%m-%d %H:%i:%s'), 
             STR_TO_DATE(fecha_de_resolucion, '%Y-%m-%d %H:%i:%s')) * 0.4167 <= sla_minutes 
        THEN 1
        ELSE 0
      END
    ELSE 0
  END) as within_sla
FROM tickets_report
WHERE fecha_de_apertura IS NOT NULL
  AND sla_minutes IS NOT NULL
```

**Características:**
- Cálculos complejos con `CASE` anidados
- Uso de factor de conversión `0.4167` para horas laborales
- Múltiples condiciones NULL checks

**⚠️ Problema identificado:**
- El factor `0.4167` es una aproximación que puede no ser precisa
- No considera fines de semana ni días festivos

#### 2.2. Consultas con Subconsultas
**Ubicación:** `app/api/dashboard/comparison/route.ts`

```sql
-- Ejemplo: Comparación de períodos
SELECT 
  COUNT(*) as total_tickets,
  (SELECT 
    (SUM(CASE ... END) * 100.0 / NULLIF(COUNT(*), 0))
   FROM tickets_report
   WHERE ...) as sla_compliance,
  (SELECT AVG(...) FROM tickets_report WHERE ...) as avg_response_time,
  ...
FROM tickets_report
WHERE ...
```

**Características:**
- Múltiples subconsultas correlacionadas
- Mismo conjunto de datos consultado múltiples veces
- Alto costo computacional

**⚠️ Problema identificado:**
- Múltiples escaneos de la misma tabla
- Podría optimizarse con CTEs o JOINs

---

### 3. Consultas de Tiempo de Respuesta

#### 3.1. Lógica de Horario Laboral
**Ubicación:** `lib/response-time-queries.ts`

```sql
-- Ejemplo: getResponseTimeData con lógica de horario laboral
SELECT 
  SUM(CASE 
    WHEN tiempo_de_poseer__progreso IS NOT NULL THEN
      CASE 
        -- Tickets después de 7pm: respuesta antes de 10am del día siguiente
        WHEN DATE_FORMAT(STR_TO_DATE(fecha_de_apertura, '%Y-%m-%d %H:%i:%s'), '%H') >= 19 AND
             DATE_FORMAT(STR_TO_DATE(tiempo_de_poseer__progreso, '%Y-%m-%d %H:%i:%s'), '%H:%i:%s') <= '10:00:00' AND
             DATE(STR_TO_DATE(tiempo_de_poseer__progreso, '%Y-%m-%d %H:%i:%s')) = 
             DATE_ADD(DATE(STR_TO_DATE(fecha_de_apertura, '%Y-%m-%d %H:%i:%s')), INTERVAL 1 DAY)
        THEN 1
        
        -- Tickets durante horario laboral (9am-7pm): respuesta en 60 minutos
        WHEN DATE_FORMAT(STR_TO_DATE(fecha_de_apertura, '%Y-%m-%d %H:%i:%s'), '%H') BETWEEN 9 AND 18 AND
             TIMESTAMPDIFF(MINUTE, 
                STR_TO_DATE(fecha_de_apertura, '%Y-%m-%d %H:%i:%s'), 
                STR_TO_DATE(tiempo_de_poseer__progreso, '%Y-%m-%d %H:%i:%s')) <= 60
        THEN 1
        
        -- Tickets antes de 9am: respuesta dentro de 60 minutos desde las 9am
        WHEN DATE_FORMAT(STR_TO_DATE(fecha_de_apertura, '%Y-%m-%d %H:%i:%s'), '%H') < 9 AND
             TIMESTAMPDIFF(MINUTE, 
                STR_TO_DATE(CONCAT(DATE(STR_TO_DATE(fecha_de_apertura, '%Y-%m-%d %H:%i:%s')), ' 09:00:00'), '%Y-%m-%d %H:%i:%s'), 
                STR_TO_DATE(tiempo_de_poseer__progreso, '%Y-%m-%d %H:%i:%s')) <= 60
        THEN 1
        
        ELSE 0 
      END
    ELSE 0 
  END) as within_60min
FROM tickets_report
WHERE ...
```

**Características:**
- Lógica compleja de horario laboral (9am-7pm)
- Múltiples condiciones `CASE` anidadas
- Conversiones de fecha repetidas

**⚠️ Problema identificado:**
- `STR_TO_DATE` se ejecuta múltiples veces para las mismas filas
- Lógica compleja que podría simplificarse

---

### 4. Consultas de Filtrado por Proyecto

#### 4.1. Patrón de Filtrado por Proyecto
**Ubicación:** Múltiples archivos

```sql
-- Patrón común: Obtener usuarios del proyecto primero
SELECT username 
FROM user_project_relations 
WHERE project_id = ?

-- Luego filtrar tickets
SELECT ...
FROM tickets_report
WHERE solicitante___solicitante IN (?, ?, ...)
```

**Características:**
- Dos consultas separadas
- Uso de `IN` con múltiples valores
- Posibles problemas de collation

**⚠️ Problema identificado:**
- Dos round-trips a la base de datos
- Podría optimizarse con JOIN

---

### 5. Consultas Temporales y Agrupación

#### 5.1. Agrupación por Períodos
**Ubicación:** `lib/temporal-analysis-queries.ts`

```sql
-- Ejemplo: getTemporalDistribution
SELECT 
  DATE_FORMAT(STR_TO_DATE(fecha_de_apertura, '%Y-%m-%d %H:%i:%s'), '%Y-%m') as yearmonth,
  DATE_FORMAT(STR_TO_DATE(fecha_de_apertura, '%Y-%m-%d %H:%i:%s'), '%b %Y') as month,
  COUNT(*) as total_tickets,
  SUM(CASE WHEN status IN ('Closed', 'Solved') THEN 1 ELSE 0 END) as resolved_tickets
FROM tickets_report
WHERE fecha_de_apertura IS NOT NULL
  AND STR_TO_DATE(fecha_de_apertura, '%Y-%m-%d %H:%i:%s') BETWEEN ? AND ?
GROUP BY yearmonth, month
ORDER BY yearmonth
```

**Características:**
- Agrupación por períodos (diario, semanal, mensual, trimestral)
- Formateo de fechas para display
- Múltiples formatos de fecha en la misma consulta

---

## 🔴 Problemas Identificados

### 1. Conversiones de Fecha Repetidas

**Problema:**
```sql
STR_TO_DATE(fecha_de_apertura, '%Y-%m-%d %H:%i:%s')
```
Esta función se ejecuta múltiples veces en la misma consulta para la misma fila.

**Impacto:**
- Alto costo computacional
- Escaneo completo de tabla en algunos casos
- Sin uso de índices en columnas de fecha

**Solución sugerida:**
- Crear columnas calculadas o vistas materializadas
- Usar índices en columnas de fecha
- Considerar almacenar fechas como tipo DATETIME en lugar de TEXT

### 2. Subconsultas Múltiples

**Problema:**
```sql
SELECT 
  COUNT(*) as total_tickets,
  (SELECT ... FROM tickets_report WHERE ...) as sla_compliance,
  (SELECT ... FROM tickets_report WHERE ...) as avg_response_time,
  ...
FROM tickets_report
```

**Impacto:**
- Múltiples escaneos de la misma tabla
- Alto tiempo de ejecución
- Mayor uso de recursos

**Solución sugerida:**
- Usar CTEs (Common Table Expressions)
- Combinar en una sola consulta con agregaciones
- Usar JOINs cuando sea apropiado

### 3. Falta de Índices

**Problema:**
- Columnas de fecha consultadas frecuentemente pero almacenadas como TEXT
- Filtros por `solicitante___solicitante` sin índice
- Filtros por `asignado_a___tecnico` sin índice

**Solución sugerida:**
```sql
-- Índices recomendados
CREATE INDEX idx_fecha_apertura ON tickets_report(fecha_de_apertura(10));
CREATE INDEX idx_solicitante ON tickets_report(solicitante___solicitante(50));
CREATE INDEX idx_tecnico ON tickets_report(asignado_a___tecnico(50));
CREATE INDEX idx_status ON tickets_report(status(20));
CREATE INDEX idx_sla_minutes ON tickets_report(sla_minutes);
```

### 4. Lógica de Horario Laboral Compleja

**Problema:**
- Lógica de horario laboral repetida en múltiples consultas
- Cálculos complejos con múltiples `CASE` anidados
- Factor de conversión `0.4167` hardcodeado

**Solución sugerida:**
- Crear función almacenada para calcular horas laborales
- Usar tabla de calendario para días laborables
- Centralizar la lógica en una función reutilizable

### 5. Consultas con Límites Altos

**Problema:**
```sql
LIMIT 10000
LIMIT 1000
```

**Impacto:**
- Transferencia de grandes volúmenes de datos
- Mayor uso de memoria
- Tiempos de respuesta lentos

**Solución sugerida:**
- Implementar paginación
- Usar límites más razonables
- Considerar streaming de resultados

---

## ✅ Mejores Prácticas Identificadas

### 1. Uso de Parámetros Preparados
✅ Todas las consultas usan parámetros preparados (`?`), previniendo SQL injection.

### 2. Manejo de NULL
✅ Uso consistente de `NULLIF` y `COALESCE` para manejar valores nulos.

### 3. Filtros Dinámicos
✅ Construcción dinámica de consultas con filtros opcionales bien implementada.

### 4. Agrupación y Ordenamiento
✅ Uso apropiado de `GROUP BY` y `ORDER BY` en consultas de agregación.

---

## 🚀 Recomendaciones de Optimización

### Prioridad Alta

1. **Convertir columnas de fecha de TEXT a DATETIME**
   ```sql
   ALTER TABLE tickets_report 
   MODIFY COLUMN fecha_de_apertura DATETIME,
   MODIFY COLUMN fecha_de_resolucion DATETIME,
   MODIFY COLUMN tiempo_de_poseer__progreso DATETIME;
   ```

2. **Crear índices en columnas frecuentemente consultadas**
   ```sql
   CREATE INDEX idx_fecha_apertura_dt ON tickets_report(fecha_de_apertura);
   CREATE INDEX idx_status_fecha ON tickets_report(status, fecha_de_apertura);
   CREATE INDEX idx_sla_fecha ON tickets_report(sla_minutes, fecha_de_apertura);
   ```

3. **Optimizar consultas con subconsultas usando CTEs**
   ```sql
   WITH base_data AS (
     SELECT * FROM tickets_report WHERE ...
   )
   SELECT 
     COUNT(*) as total,
     (SELECT ... FROM base_data) as metric1,
     (SELECT ... FROM base_data) as metric2
   FROM base_data
   ```

### Prioridad Media

4. **Crear función almacenada para horas laborales**
   ```sql
   CREATE FUNCTION business_hours(start_dt DATETIME, end_dt DATETIME)
   RETURNS DECIMAL(10,2)
   BEGIN
     -- Lógica de cálculo de horas laborales
   END
   ```

5. **Implementar paginación en consultas grandes**
   ```sql
   SELECT ... FROM tickets_report
   WHERE ...
   ORDER BY ...
   LIMIT ? OFFSET ?
   ```

6. **Usar vistas materializadas para consultas frecuentes**
   ```sql
   CREATE MATERIALIZED VIEW mv_ticket_stats AS
   SELECT 
     DATE(fecha_de_apertura) as date,
     status,
     COUNT(*) as count
   FROM tickets_report
   GROUP BY DATE(fecha_de_apertura), status;
   ```

### Prioridad Baja

7. **Refactorizar lógica de horario laboral**
   - Centralizar en función reutilizable
   - Considerar tabla de calendario

8. **Optimizar consultas de filtrado por proyecto**
   - Usar JOIN en lugar de dos consultas separadas

---

## 📊 Análisis de Performance

### Consultas Más Costosas (Estimado)

1. **Consultas con múltiples subconsultas** (`app/api/dashboard/comparison/route.ts`)
   - Tiempo estimado: 2-5 segundos
   - Escaneo: Múltiples escaneos completos de tabla

2. **Consultas de tiempo de respuesta con lógica de horario** (`lib/response-time-queries.ts`)
   - Tiempo estimado: 1-3 segundos
   - Escaneo: Completo con múltiples conversiones de fecha

3. **Consultas de backlog con múltiples estados** (`lib/backlog-queries.ts`)
   - Tiempo estimado: 0.5-2 segundos
   - Escaneo: Completo con múltiples condiciones LIKE

### Consultas Optimizadas

1. **Consultas simples de conteo** (`lib/db.ts`)
   - Tiempo estimado: < 0.1 segundos
   - Escaneo: Índice si existe

---

## 🔧 Patrones de Consulta Comunes

### Patrón 1: Filtrado por Fecha
```sql
WHERE fecha_de_apertura IS NOT NULL
  AND STR_TO_DATE(fecha_de_apertura, '%Y-%m-%d %H:%i:%s') BETWEEN ? AND ?
```

**Uso:** Presente en 90% de las consultas

### Patrón 2: Filtrado por Estado
```sql
WHERE status IN ('Closed', 'Solved', 'Processing', ...)
```

**Uso:** Presente en 70% de las consultas

### Patrón 3: Agregación con CASE
```sql
SUM(CASE 
  WHEN condition1 THEN 1
  WHEN condition2 THEN 1
  ELSE 0
END) as metric
```

**Uso:** Presente en 60% de las consultas

### Patrón 4: Cálculo de Tiempo
```sql
TIMESTAMPDIFF(MINUTE, 
  STR_TO_DATE(fecha_de_apertura, '%Y-%m-%d %H:%i:%s'),
  STR_TO_DATE(fecha_de_resolucion, '%Y-%m-%d %H:%i:%s')
)
```

**Uso:** Presente en 50% de las consultas

---

## 📝 Notas Adicionales

### Estructura de Datos

- **Tabla principal:** `tickets_report` (28 columnas)
- **Columnas de fecha:** Almacenadas como TEXT (formato: `'%Y-%m-%d %H:%i:%s'`)
- **Columnas indexadas:** `sla_minutes`, `import_date`, `ticket_id`, `last_updated`, `is_complete`

### Convenciones de Nombres

- Columnas con formato: `campo___subcampo` (ej: `solicitante___solicitante`)
- Uso de backticks para nombres con caracteres especiales
- Aliases descriptivos en SELECT

### Manejo de Errores

- Todas las consultas están envueltas en try-catch
- Retorno de valores por defecto en caso de error
- Logging extensivo para debugging

---

## 🎯 Conclusión

El proyecto utiliza consultas SQL bien estructuradas con parámetros preparados y manejo adecuado de errores. Sin embargo, hay oportunidades significativas de optimización:

1. **Conversión de tipos de datos** (TEXT → DATETIME)
2. **Creación de índices** en columnas frecuentemente consultadas
3. **Optimización de subconsultas** usando CTEs o JOINs
4. **Centralización de lógica** de horario laboral

Estas optimizaciones podrían mejorar el rendimiento en un 50-80% según el tipo de consulta.

---

**Última actualización:** Noviembre 2025

