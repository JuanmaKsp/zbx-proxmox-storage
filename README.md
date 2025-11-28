# zbx-proxmox-storage

Paquete `.deb` para **monitorizar el porcentaje de almacenamiento local usado** en nodos Proxmox mediante `zabbix-agent2`.

El objetivo es exponer en Zabbix un único ítem:

> `proxmox.local_storage_used_pct`

que devuelve el **porcentaje global de espacio usado** en los storages **locales** de un nodo Proxmox (excluyendo storages de backup remotos, PBS, etc.).

---

## Características

- Calcula el **% de uso total** de los storages locales de Proxmox a partir de `pvesm status`.
- Incluye estos tipos de storage:
  - `dir`
  - `zfspool`
  - `lvm`
  - `lvmthin`
  - `btrfs`
- Excluye cualquier storage cuyo **nombre contenga** `copias` (por ejemplo: `local-ext4Copias`).
- Actualiza el valor cada 5 minutos mediante `cron`.
- Zabbix **no ejecuta comandos privilegiados**: solo lee un fichero en `/var/lib/zabbix`.
- Pensado para usarse con un **template de Zabbix** que define el ítem `proxmox.local_storage_used_pct`.

---

## Cómo funciona internamente

En cada nodo Proxmox:

1. El script `zbx_disk_used_pct.sh` (instalado en `/usr/local/bin/`) ejecuta:

   ```bash
   /usr/sbin/pvesm status
