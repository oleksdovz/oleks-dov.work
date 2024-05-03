## Proxmox VE pve restore Commands

The `pve restore` command-line tool in Proxmox VE is used to restore containers and virtual machines (VMs) from backups created with `vzdump`. Here's a breakdown of common `pve restore` commands:

**Restore a Container:**

* `pve restore <container_id> <backup_file>`: Restores the specified container from the backup file. Replace `<container_id>` with the desired container ID and `<backup_file>` with the path to the backup file.

**Restore a Virtual Machine:**

* `pve restore <vm_id> <backup_file>`: Restores the specified VM from the backup file. Replace `<vm_id>` with the desired VM ID and `<backup_file>` with the path to the backup file.

**Additional Options:**

* **Specify Restore Mode:**
    * `--restore-mode <active|inactive>`: Sets the state of the restored container/VM. 
        * `active` (default): Starts the container/VM after restoration.
        * `inactive`: Leaves the container/VM stopped after restoration.
* **Verify Backup Before Restore:**
    * `--verify`: Verifies the integrity of the backup file before proceeding with the restore.
* **Advanced Options:**
    * `pve restore help`: Provides help information for `pve restore` commands.
    * `man pve restore`: Displays the man page for the `pve restore` command with detailed explanations and examples.

**Remember:**

* Ensure the backup file is compatible with the Proxmox VE version you're using.
* Restoring a container/VM overwrites existing data. Proceed with caution and back up any critical data beforehand if necessary.
* Choose the appropriate restore mode based on your needs.

For comprehensive details and advanced usage, refer to the Proxmox VE documentation: [invalid URL removed]