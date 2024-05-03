## Proxmox VE pveam Commands

The `pveam` command-line tool in Proxmox VE is used to manage container and appliance templates. Here's a summary of common `pveam` commands:

**List Templates:**

* `pveam list [--section <section>]`: Lists all available templates. Optionally, filter by section (e.g., `mail`, `system`, `turnkeylinux`).

**Download Templates:**

* `pveam download <storage> <template>`: Downloads a template to the specified storage location. Replace `<storage>` with the storage ID and `<template>` with the template name.

**Update Template Database:**

* `pveam update`: Updates the container template database from the official Proxmox repositories.

**Remove Templates:**

* `pveam remove <template_path>`: Removes a template from the local storage. Replace `<template_path>` with the full path to the template file.

**Additional Options:**

* `pveam help [--extra-args <array>] [--verbose <boolean>]`: Provides help information for `pveam` commands.
    * `--extra-args`: Specify a command to get help for.
    * `--verbose`: Display more detailed output.

**Remember:**

* Templates are typically stored in `/var/lib/vz/template` on the Proxmox VE host.
* Use the `pct create` command along with the downloaded template name to create a container from the template.

For comprehensive details and advanced usage, refer to the Proxmox VE documentation: [https://pve.proxmox.com/pve-docs/pveam.1.html](https://pve.proxmox.com/pve-docs/pveam.1.html)
