# Matriz de Aplicaciones y Permisos - ADPER.HITSS.MX

## Matriz de Permisos por Rol y Perfil de Aplicación

| Rol | ACTIVA<br>Administrador | ACTIVA<br>Gerente | RADIUS INTERNO<br>Administrador | RADIUS INTERNO<br>Operador | Integrador<br>Administrador | Wapper<br>Administrador | NETpoint<br>Administrador | VMBackup<br>Administrador | Vcenter<br>Administrador | Vcenter<br>Operador N2 | Vcenter<br>Operador N1 | Influxdb<br>Administrador | Vaultwarden<br>Administrador | Vaultwarden<br>Operador | Odoo<br>Administrador | grafana<br>Administrador | grafana<br>View | Monitor<br>Administrador | issabelpbx<br>Administrador | truenas<br>Administrador | elastic<br>Administrador | elastic<br>Operador | Fileserver<br>Administrador | Fileserver<br>Operador | AD<br>Administrador | AD<br>Operador N1 |
|-----|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| **Alta Gerencia** | | | | | | | | | | | | | | | | | | | | | | | | | | | |
| CEO | | | | | | | | | | | | | | | | | | | | | | | | | | | |
| COO | | | | | | | | | | | | | | | | | | | | | | | | | | | |
| CTO | | | | | | | | | | | | | | | | | | | | | | | | | | | |
| CFO | | | | | | | | | | | | | | | | | | | | | | | | | | | |
| CTIO | | | | | | | | | | | | | | | | | | | | | | | | | | | |
| **Tecnología** | | | | | | | | | | | | | | | | | | | | | | | | | | | |
| Developers | | | | | | | | | | | | | | | | | | | | | | | | | | | |
| Lider Tecnico | | | | | | | | | | | | | | | | | | | | | | | | | | | |
| Operaciones TI | | | | | | | | | | | | | | | | | | | | | | | | | | | |
| Responsable de Infraestructura | | | | | | | | | | | | | | | | | | | | | | | | | | | |
| **Administradores** | | | | | | | | | | | | | | | | | | | | | | | | | | | |
| Administrador de Servidores | | X | X | X | X | X | X | X | X | X | X | X | X | X | X | X | X | X | X | X | X | X | X | X | X | X |
| Administrador de Redes | X | | X | | | | X | | | | X | | | X | X | | X | | X | | | | | X | | | | |
| Coordinador de Soporte | | | X | | | | | | | | X | | | | | | | | | | | | | | | | |
| **Soporte** | | | | | | | | | | | | | | | | | | | | | | | | | | | |
| Soporte N1 | | | | X | | | | | | | X | | | | | | | | | | | | | | | X | X | |
| Personal de N1 | | | | | | | | | | | | | | | | | | | | | | | | | | | |
| **Gerencia** | | | | | | | | | | | | | | | | | | | | | | | | | | | |
| Gerente | X | | | | | | | | | | | | | | | | | | | | | | | | | | | |

---

## Notas sobre la estructura

Cada aplicación tiene sus perfiles como columnas separadas con encabezados claros:

**ACTIVA**: 2 columnas (Administrador, Gerente)
**RADIUS INTERNO**: 2 columnas (Administrador, Operador)
**Integrador, Wapper, NETpoint, VMBackup, Influxdb, Odoo, Monitor, issabelpbx, truenas**: 1 columna cada uno (Administrador)
**Vcenter**: 3 columnas (Administrador, Operador N2, Operador N1)
**Vaultwarden**: 2 columnas (Administrador, Operador)
**grafana**: 2 columnas (Administrador, View)
**elastic**: 2 columnas (Administrador, Operador)
**Fileserver**: 2 columnas (Administrador, Operador)
**AD**: 2 columnas (Administrador, Operador N1)

**Permisos:**
- **X** = El rol tiene acceso a ese perfil específico
- **Vacío** = El rol NO tiene acceso
- **Alta Gerencia** (CEO, COO, CTO, CFO, CTIO) y **Tecnología** (Developers, Lider Tecnico, Operaciones TI, Responsable de Infraestructura): **NO tienen acceso a ninguna aplicación**

**Edición:**
- Esta matriz puede copiarse directamente a Excel o Google Sheets
- Cada celda puede editarse para agregar o quitar marcas X
- Los encabezados muestran claramente qué perfil corresponde a cada aplicación
