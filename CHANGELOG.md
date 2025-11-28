# Changelog

Todas las versiones de `zbx-proxmox-storage`.  
Formato: `Versión - Fecha - Descripción`.

## 1.1-2 — 2025-11-28

### Cambios

- Unificación de la lógica de Proxmox VE y Proxmox Backup Server (PBS) en **un solo script**:
  - Nuevo script: `zbx_storage_used_pct.sh`.
  - El script **autodetecta el entorno**:
    - Si existe `/usr/sbin/pvesm` → se asume **Proxmox VE**.
    - Si existe `/etc/proxmox-backup/datastore.cfg` → se asume **Proxmox Backup Server (PBS)**.
    - Si no se reconoce el entorno → devuelve `0` sin hacer nada peligroso.
- Proxmox VE:
  - Se mantiene el cálculo basado en `pvesm status`.
  - Se incluyen tipos de storage: `dir`, `zfspool`, `lvm`, `lvmthin`, `btrfs`.
  - Se excluyen storages cuyo nombre contenga `copias`.
- Proxmox Backup Server (PBS):
  - El cálculo de uso se basa en `df -B1 -x tmpfs -x devtmpfs`.
  - Solo se tiene en cuenta la línea cuyo mountpoint es `/` (el filesystem raíz donde están los datastores), evitando contar `/rpool`, `/boot/efi`, etc. y duplicar espacio.
- Nuevo fichero de salida unificado:
  - `OUTFILE`: `/var/lib/zabbix/proxmox_storage_used_pct`.
- Nuevo `UserParameter` único:
  - Key: `proxmox.storage_used_pct`.
  - Comando: `cat /var/lib/zabbix/proxmox_storage_used_pct`.
- Nuevo cron único:
  - `/etc/cron.d/zbx_storage_used_pct` → ejecuta `zbx_storage_used_pct.sh` cada 5 minutos.
- El paquete queda preparado para usarse tanto en nodos Proxmox VE como en Proxmox Backup Server con el **mismo .deb** y el mismo ítem en Zabbix.

### Impacto

- En Zabbix:
  - El nuevo ítem recomendado es `proxmox.storage_used_pct` (sustituye a las keys anteriores específicas de VE/PBS).
  - Se puede usar el **mismo template** para nodos VE y PBS, o bien plantillas separadas que compartan el mismo ítem.
- En los nodos:
  - El script se adapta automáticamente al tipo de Proxmox.
  - Se simplifica el despliegue: un solo cron, un solo UserParameter, un único fichero de salida.

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
