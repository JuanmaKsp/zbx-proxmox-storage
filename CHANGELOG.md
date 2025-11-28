# Changelog

Todas las versiones de `zbx-proxmox-storage`.  
Formato: `Versión - Fecha - Descripción`.

---

## 1.1-1 — 2025-11-28

### Cambios

- Ampliado el cálculo de almacenamiento local para incluir más tipos de storage:
  - Antes: `dir`, `zfspool`, `lvmthin`
  - Ahora: `dir`, `zfspool`, `lvm`, `lvmthin`, `btrfs`
- Se mantiene la exclusión de storages cuyo nombre contenga `copias` (por ejemplo: `local-ext4Copias`).
- Endurecido el script principal:
  - Añadido `set -euo pipefail`.
  - Uso de rutas absolutas (`/usr/sbin/pvesm`, etc.).
  - Comprobación defensiva por si el valor calculado queda vacío.
- Documentación mejorada:
  - Añadido `README.md` con descripción del paquete, instalación y uso en Zabbix.
  - Detallados permisos recomendados y estructura del `.deb`.

### Impacto

- El ítem de Zabbix `proxmox.local_storage_used_pct` sigue siendo el mismo.
- No se requiere ningún cambio en plantillas de Zabbix.
- Tras actualizar el `.deb`, el porcentaje global incluirá los nuevos tipos de almacenamiento (lvm y btrfs) sin intervención manual.

---

## 1.0-1 — 2025-11-27

### Cambios

- Versión inicial del paquete `zbx-proxmox-storage`.
- Funcionalidad principal:
  - Script `/usr/local/bin/zbx_disk_used_pct.sh` que:
    - Ejecuta `pvesm status`.
    - Filtra storages locales de tipos `dir`, `zfspool`, `lvmthin`.
    - Excluye storages cuyo nombre contenga `copias`.
    - Calcula el porcentaje global de uso: `used / total * 100`.
    - Escribe el resultado en `/var/lib/zabbix/proxmox_local_storage_used_pct`.
  - Cron `/etc/cron.d/zbx_disk_used_pct` para actualizar el valor cada 5 minutos.
  - `UserParameter` para `zabbix-agent2`:
    - `proxmox.local_storage_used_pct,cat /var/lib/zabbix/proxmox_local_storage_used_pct`
- Scripts de mantenimiento del paquete:
  - `postinst`: ajusta permisos, crea directorio de datos, ejecuta el script una vez y reinicia `zabbix-agent2`.
  - `postrm`: elimina el fichero de datos en caso de `apt purge`.

### Impacto

- Introduce el ítem `proxmox.local_storage_used_pct` como número flotante (%) listo para usar en plantillas de Zabbix.
- Permite crear dashboards con gauges, top hosts y triggers basados en el porcentaje de uso de almacenamiento local.
