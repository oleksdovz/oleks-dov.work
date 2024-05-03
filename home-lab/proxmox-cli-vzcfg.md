## Proxmox VE vzcfg Commands

The `vzcfg` command-line tool in Proxmox VE is used to configure and manage aspects of containers within Proxmox VE. It offers a more granular level of control compared to the higher-level `pct` tool.

**Key vzcfg Commands:**

1. **Create a Configuration File:**

   * `vzcfg create <container_id> <template>`: Generates a configuration file (`vz.conf`) for a new container based on the specified template. Replace `<container_id>` with the desired ID and `<template>` with the template name.

2. **Modify Container Configuration:**

   * `vzcfg edit <container_id>`: Opens the `vz.conf` file for the specified container in a text editor for manual modifications.

3. **Set Configuration Options:**

   * `vzcfg set <container_id> <option>=<value>`: Sets a specific configuration option for the container. Replace `<option>` with the desired option name and `<value>` with the desired value. Example: `vzcfg set 100 onboot=1` enables automatic startup for container 100.

4. **View Configuration Options:**

   * `vzcfg dump <container_id>`: Displays the current configuration options for the specified container in a structured format.

5. **Validate Configuration:**

   * `vzcfg check <container_id>`: Checks the syntax and validity of the container's configuration file.

6. **Apply Configuration Changes:**

   * `vzcfg update <container_id>`: Applies the changes made to the container's configuration file.

**Additional Options:**

* `vzcfg help`: Provides help information for `vzcfg` commands.
* `man vzcfg`: Displays the man page for the `vzcfg` command with detailed explanations and examples.

**Remember:**

* `vzcfg` directly modifies configuration files, so it's important to understand the syntax and implications of the changes you make.
* Use `vzcfg` with caution, as incorrect configurations can affect container functionality.

For comprehensive details and advanced usage, refer to the Proxmox VE documentation: [https://pve.proxmox.com/pve-docs-6/pve-admin-guide.html](https://pve.proxmox.com/pve-docs-6/pve-admin-guide.html)
