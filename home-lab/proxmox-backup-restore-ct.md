## Backing Up and Restoring Containers in Proxmox VE (CLI)

This guide shows how to create a container backup and restore it using the command line in Proxmox VE.

**Backup:**

1. **List Containers:** Get a list of containers and their IDs with:

```
pct list
```

2. **Backup Container:** Replace `<container_id>` and `<backup_file>`:

```
vzdump <container_id> <backup_file>.tar.gz
```
```
vzdump 100 --mode snapshot --storage local --node hp --notes-template debian --remove 0 --compress zstd --notification-mode auto
```

**Restore:**

1. **Specify Backup Location:** Replace `<backup_file>` and `<container_id>`:

```
pct restore <container_id> <backup_file>.tar.gz
```
```
pct restore 111 /var/lib/vz/dump/vzdump-lxc-100-2024_05_03-19_33_25.tar.zst
```

**Tips:**

* Replace `.tar.gz` with `.lzo` or `.gz` depending on your chosen compression during backup.
* Use `vzdump --help` for more options like specifying backup mode (e.g., `-mode suspend`).

**Remember:** This is a basic guide. Always refer to the Proxmox VE documentation for detailed information and advanced options: [https://pbs.proxmox.com/docs/pve-integration.html](https://pbs.proxmox.com/docs/pve-integration.html)
