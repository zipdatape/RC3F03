# Análisis de Base de Datos GLPI Tickets

**Fecha de análisis:** Noviembre 2025  
**Base de datos:** glpi_tickets  
**Host:** 10.172.1.226:3306  
**Versión MySQL:** 8.0.41  
**Tamaño de BD:** 9.63 MB

---

## 📊 Resumen Ejecutivo

- **Total de tickets:** 7,098
- **IDs únicos de GLPI:** 7,098
- **Solicitantes únicos:** 800
- **Técnicos únicos:** 18
- **Categorías únicas:** 213
- **Rango de fechas:** Marzo 2025 - Noviembre 2025

---

## 📋 Estructura de la Base de Datos

### Tablas Principales (27 tablas)

1. **tickets_report** - Tabla principal con información completa de tickets (7,098 registros)
2. **tickets** - Tabla de tickets (43 registros)
3. **ticket_history** - Historial de cambios de tickets (43 registros)
4. **ticket_assignments** - Asignaciones de tickets (0 registros)
5. **projects** - Proyectos (56 registros)
6. **user_project_relations** - Relaciones usuario-proyecto (2 registros)
7. **stats** - Estadísticas (1 registro)
8. **daily_ticket_reports** - Reportes diarios
9. **ticket_summary_daily** - Resúmenes diarios
10. **ticket_summary_monthly** - Resúmenes mensuales
11. Y otras tablas de soporte...

### Estructura de tickets_report (28 columnas principales)

- `id` - ID único
- `glpi_id` - ID del ticket en GLPI
- `titulo` - Título del ticket
- `category` - Categoría
- `solicitante___solicitante` - Solicitante
- `asignado_a___tecnico` - Técnico asignado
- `status` - Estado del ticket
- `fecha_de_apertura` - Fecha de apertura
- `fecha_de_resolucion` - Fecha de resolución
- `glpi_type` - Tipo (Request/Incident)
- `sla_level` - Nivel de SLA
- `sla_minutes` - Minutos de SLA
- `tiempo_de_poseer__progreso` - Tiempo de primera respuesta
- Y más...

---

## 📈 Estadísticas Generales

### Distribución por Estado

| Estado | Cantidad | Porcentaje |
|--------|----------|------------|
| Closed | 6,851 | 96.52% |
| Solved | 116 | 1.63% |
| Processing (assigned) | 87 | 1.23% |
| Pending | 41 | 0.58% |
| New | 3 | 0.04% |

**Observación:** La mayoría de los tickets están cerrados (96.52%), lo que indica un buen nivel de resolución.

### Distribución por Tipo

| Tipo | Cantidad | Porcentaje |
|------|----------|------------|
| Request | 6,076 | 85.60% |
| Incident | 1,022 | 14.40% |

**Observación:** La mayoría son solicitudes (Requests) en lugar de incidentes.

### Tickets Resueltos vs Pendientes

- **Total de tickets:** 7,098
- **Tickets resueltos:** 7,098 (100.00%)
- **Tickets pendientes:** 0 (0.00%)
- **Tiempo promedio de resolución:** 58.77 horas

**Nota:** Aunque todos los tickets tienen fecha de resolución, hay 131 tickets con estados que no son "Closed" o "Solved" (Processing, Pending, New), lo que sugiere que algunos tickets pueden estar marcados como resueltos pero aún en proceso.

---

## 📂 Top 10 Categorías

| # | Categoría | Cantidad |
|---|-----------|----------|
| 1 | Proceso > Equipo de computo / preparar y asignar equipo de computo | 834 |
| 2 | Hardware > Laptop > CESE DE PUESTO | 441 |
| 3 | Soporte Técnico / Infraestructura | 435 |
| 4 | Hardware > Laptop > PUESTO NUEVO | 338 |
| 5 | Hardware > Laptop > CAMBIO DE EQUIPO | 270 |
| 6 | Software > Estandar > INSTALACION-OTROS | 255 |
| 7 | Negocio > VPN / configuracion y acceso | 239 |
| 8 | Documentacion > Base de responsivas / proporcionar información | 229 |
| 9 | Hardware > Accesorios > INSTALACION-OTROS | 222 |
| 10 | Negocio > Site / acceso | 213 |

**Observación:** Las categorías más comunes están relacionadas con equipos de cómputo, laptops y soporte técnico.

---

## 👤 Top 10 Técnicos

| # | Técnico | Cantidad de Tickets |
|---|---------|---------------------|
| 1 | Huarancca Pacheco Kenyo Alfonso | 1,531 |
| 2 | Rodriguez Alegre Juan Daniel | 1,473 |
| 3 | Valderrama Pajuelo Jonhatan Deaves | 968 |
| 4 | Prieto Arangoitia Patricia | 966 |
| 5 | Gutierrez Chagua Luis Alberto | 897 |
| 6 | Baca Florentino Alexander Antonio | 758 |
| 7 | Atoche Quiroz Robinson Abrahan | 260 |
| 8 | Blas Alvarado Ivan | 115 |
| 9 | Romero Araujo Hector Miguel | 51 |
| 10 | Huaman Portuguez Humberto Abelardo | 44 |

---

## ⏱️ Análisis de SLA

### Estadísticas Generales

- **Tickets con SLA definido:** 6,279 (88.46% del total)
- **Niveles de SLA únicos:** 6
- **SLA promedio:** 1,582.18 minutos (26.37 horas)
- **SLA mínimo:** 90 minutos (1.5 horas)
- **SLA máximo:** 2,880 minutos (48 horas)

### Niveles de SLA

| Nivel | Minutos | Horas | Total Tickets | Tiempo Promedio Resolución |
|-------|---------|-------|---------------|----------------------------|
| 1 | 90 | 1.5 | 351 | 67.01 horas |
| 2 | 120 | 2.0 | 1,424 | 39.72 horas |
| 3 | 180 | 3.0 | 239 | 31.62 horas |
| 4 | 240 | 4.0 | 489 | 52.96 horas |
| 5 | 480 | 8.0 | 543 | 43.41 horas |
| 6 | 2,880 | 48.0 | 3,233 | 63.40 horas |

**Observación:** El nivel 6 (48 horas) es el más común con 3,233 tickets. El tiempo promedio de resolución en todos los niveles excede significativamente el SLA establecido.

### Cumplimiento de SLA

- **Tickets resueltos dentro del SLA:** 3,332 (53.07% de los tickets con SLA)
- **Tickets que excedieron el SLA:** 2,850 (45.39% de los tickets con SLA)

**⚠️ Problema identificado:** Casi la mitad de los tickets exceden el SLA establecido.

---

## ⏱️ Tiempo de Respuesta por Técnico

| Técnico | Total Tickets | Tiempo Respuesta Promedio | Tiempo Resolución Promedio |
|---------|---------------|---------------------------|----------------------------|
| Huarancca Pacheco Kenyo Alfonso | 1,531 | 238.07 min (3.97 hrs) | 23.89 horas |
| Rodriguez Alegre Juan Daniel | 1,473 | 233.06 min (3.88 hrs) | 49.06 horas |
| Valderrama Pajuelo Jonhatan Deaves | 968 | 206.20 min (3.44 hrs) | 127.61 horas |
| Prieto Arangoitia Patricia | 966 | 238.07 min (3.97 hrs) | 103.87 horas |
| Gutierrez Chagua Luis Alberto | 897 | 323.03 min (5.38 hrs) | 25.18 horas |
| Baca Florentino Alexander Antonio | 758 | 358.64 min (5.98 hrs) | 34.71 horas |
| Atoche Quiroz Robinson Abrahan | 260 | 300.54 min (5.01 hrs) | 99.93 horas |
| Blas Alvarado Ivan | 115 | 276.57 min (4.61 hrs) | 75.64 horas |
| Romero Araujo Hector Miguel | 51 | 320.96 min (5.35 hrs) | 60.98 horas |
| Huaman Portuguez Humberto Abelardo | 44 | 989.14 min (16.49 hrs) | 6.50 horas |

**Observación:** El tiempo de respuesta promedio varía entre 3.4 y 16.5 horas. Algunos técnicos tienen tiempos de respuesta muy altos.

---

## 📅 Distribución Temporal

### Tickets por Mes (Últimos 12 meses)

| Mes | Cantidad |
|-----|----------|
| 2025-11 | 260 |
| 2025-10 | 1,225 |
| 2025-09 | 849 |
| 2025-08 | 637 |
| 2025-07 | 889 |
| 2025-06 | 1,126 |
| 2025-05 | 787 |
| 2025-04 | 596 |
| 2025-03 | 729 |

**Observación:** Octubre 2025 tuvo el mayor volumen de tickets (1,225), seguido de junio (1,126).

---

## 📁 Proyectos

La base de datos contiene **56 proyectos**, incluyendo:

1. SANTANDER - Servicio de Consultoría Lideres Técnicos (Código: 73731)
2. SANTANDER - Servicio de Ambiente No Productivo Cloud (Código: 76616)
3. PROM. TUR. NUEVO MUNDO - Desarrollo de SW (Código: 78872)
4. NEOAUTO - Locación de Servicios Especializados (Código: 78208)
5. INFRAESTRUCTURA _PERÚ 2025
6. HITSS INC - Wholesale CPT TELCEL (Código: 79433)
7. HITSS INC - Mejoras Sistemas CES (Código: 79435)
8. HITSS INC - Consultoría de Sistemas (Código: 76723)
9. GASTOS ADMINISTRATIVOS_PERÚ 2025
10. DIRECCIÓN PREVENTA_PERU 2025

---

## 🔍 Hallazgos y Recomendaciones

### ✅ Puntos Positivos

1. **Alto porcentaje de resolución:** 96.52% de tickets cerrados
2. **Buena cobertura de SLA:** 88.46% de tickets tienen SLA definido
3. **Diversidad de categorías:** 213 categorías diferentes permiten buena clasificación
4. **Equipo distribuido:** 18 técnicos gestionando los tickets

### ⚠️ Áreas de Mejora

1. **Cumplimiento de SLA:** 45.39% de tickets exceden el SLA
   - **Recomendación:** Revisar los tiempos de SLA o mejorar los procesos de resolución

2. **Tiempo de respuesta:** Algunos técnicos tienen tiempos de respuesta muy altos (hasta 16.5 horas)
   - **Recomendación:** Implementar alertas para tickets sin respuesta en las primeras horas

3. **Tiempo promedio de resolución:** 58.77 horas es alto
   - **Recomendación:** Analizar tickets que toman más tiempo y optimizar procesos

4. **Tickets pendientes:** Aunque todos tienen fecha de resolución, hay 131 tickets con estados activos
   - **Recomendación:** Revisar y actualizar el estado de estos tickets

### 📊 Métricas Clave

- **Tasa de resolución:** 100% (todos tienen fecha de resolución)
- **Cumplimiento de SLA:** 53.07%
- **Tiempo promedio de respuesta:** ~4-5 horas
- **Tiempo promedio de resolución:** 58.77 horas
- **Tickets por técnico (promedio):** ~394 tickets

---

## 🔗 Tablas Relacionadas

### user_project_relations
- **Total de relaciones:** 2
- **Usuarios únicos:** 2
- **Proyectos únicos:** 2

Esta tabla parece estar poco utilizada. Considerar expandir su uso para mejor seguimiento de proyectos.

---

## 📝 Notas Técnicas

1. **Formato de fechas:** Las fechas están almacenadas como texto en formato `'%Y-%m-%d %H:%i:%s'`
2. **Nombres de columnas:** Algunas columnas usan formato con guiones bajos múltiples (ej: `solicitante___solicitante`)
3. **Índices:** La tabla `tickets_report` tiene índices en:
   - `sla_minutes`
   - `import_date`
   - `ticket_id`
   - `last_updated`
   - `is_complete`

---

## 🛠️ Scripts de Análisis

Se han creado los siguientes scripts para análisis:

1. **scripts/analyze-db.ts** - Análisis general de la base de datos
2. **scripts/query-tickets.ts** - Consultas detalladas de tickets

Para ejecutar:
```bash
MYSQL_HOST=10.172.1.226 MYSQL_PORT=3306 MYSQL_USER=glpi_monitor MYSQL_PASSWORD=glpi_password MYSQL_DATABASE=glpi_tickets npx tsx scripts/analyze-db.ts
```

---

**Última actualización:** Noviembre 2025

