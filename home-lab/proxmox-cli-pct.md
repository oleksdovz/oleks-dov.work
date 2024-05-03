## Proxmox VE pct Commands

The `pct` command allows you to manage containers within Proxmox VE. Here are some commonly used `pct` commands:

* **List Containers:**
   * `pct list`: Shows a list of all containers with their IDs and names.

* **Information about a Container:**
   * `pct info <container_id>`: Displays detailed information about a specific container, including its ID, name, OS, CPU, memory, and status.

* **Start/Stop/Restart a Container:**
   * `pct start <container_id>`: Starts a stopped container.
   * `pct stop <container_id>`: Stops a running container.
   * `pct reboot <container_id>`: Restarts a running container.

* **Create a Container:**
   * `pct create <container_id> <template>`: Creates a new container using a specified template. Replace `<container_id>` with your desired ID and `<template>` with the name of the template.

* **Delete a Container:**
   * `pct destroy <container_id>`: Deletes a container. **Use with caution!**

* **Enter Container Console:**
   * `pct console <container_id>`: Opens a console window where you can interact directly with the container.

* **Backup and Restore:**
   * `vzdump <container_id> <backup_file>`: Creates a backup of a container.  **Note:** `vzdump` is a separate command but often used with `pct`.
   * `pct restore <container_id> <backup_file>`: Restores a container from a backup created with `vzdump`.

* **Advanced Options:**
   * `pct help`: Provides a full list of available `pct` commands and options.
   * `man pct`: Displays the manual page for the `pct` command with detailed explanations and examples.

**Remember:** Replace `<container_id>` with the actual ID of your container whenever applicable.

For comprehensive information and further options, refer to the Proxmox VE documentation: [https://pve.proxmox.com/pve-docs/pct.1.html](https://pve.proxmox.com/pve-docs/pct.1.html)
