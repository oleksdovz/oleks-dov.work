## Proxmox VE vzdump Commands

The `vzdump` command-line tool in Proxmox VE is used to create backups of containers and virtual machines (VMs). It offers flexible backup options and can be used for both local and remote backups.

**Common vzdump Commands:**

1. **Create a Backup:**

   * `vzdump <container_id> <backup_file>`: Creates a backup of the specified container to the specified file. Replace `<container_id>` with the container's ID and `<backup_file>` with the desired backup file path.

   * `vzdump <vm_id> <backup_file>`: Creates a backup of the specified VM to the specified file. Replace `<vm_id>` with the VM's ID and `<backup_file>` with the desired backup file path.

2. **Create a Snapshot Backup:**

   * `vzdump <container_id> --mode snapshot <backup_file>`: Creates a snapshot backup of the specified container to the specified file. This captures the container's state at a specific point in time.

   * `vzdump <vm_id> --mode snapshot <backup_file>`: Creates a snapshot backup of the specified VM to the specified file. This captures the VM's state at a specific point in time.

3. **Specify Compression Level:**

   * `vzdump ... --compress <level>`: Sets the compression level for the backup. Options range from 0 (no compression) to 9 (highest compression). Default is 4.

4. **Encrypt Backup:**

   * `vzdump ... --encrypt` or `vzdump ... --encryption <method>`: Encrypts the backup using the specified encryption method. Options include `aes256` (default) and `blowfish`.

5. **Include or Exclude Files:**

   * `vzdump ... --include <path>`: Includes the specified path and its contents in the backup.
   * `vzdump ... --exclude <path>`: Excludes the specified path and its contents from the backup.

6. **Verify Backup Integrity:**

   * `vzdump verify <backup_file>`: Verifies the integrity of the specified backup file.

**Additional Options:**

* `vzdump help`: Displays help information for `vzdump` commands.
* `man vzdump`: Provides the man page for the `vzdump` command with detailed explanations and examples.

**Remember:**

* Backups are crucial for data protection. Regularly back up your containers and VMs to prevent data loss.
* Use encryption for sensitive backups to safeguard your data.
* Store backups in a secure and accessible location.

For comprehensive details and advanced usage, refer to the Proxmox VE documentation: [https://pve.proxmox.com/pve-docs/vzdump.1.html](https://pve.proxmox.com/pve-docs/vzdump.1.html)
