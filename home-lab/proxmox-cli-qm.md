## Proxmox VE qm Commands

The `qm` command-line tool in Proxmox VE is used for managing virtual machines (VMs). Here's a breakdown of some common `qm` commands:

* **List VMs:**
    * `qm list`: Displays a list of all VMs with their IDs and names.

* **Information about a VM:**
    * `qm info <vm_id>`: Shows detailed information about a specific VM, including its ID, name, OS, CPU, memory, and status.

* **Control VM Power State:**
    * `qm start <vm_id>`: Starts a stopped VM.
    * `qm stop <vm_id>`: Stops a running VM.
    * `qm shutdown <vm_id>`: Gracefully shuts down a running VM. (Similar to stop)
    * `qm reboot <vm_id>`: Restarts a running VM.

* **Create and Clone VMs:**
    * `qm create <vm_id> <template>`: Creates a new VM using a specified template. Replace `<vm_id>` with your desired ID and `<template>` with the name of the template.
    * `qm clone <vm_id> <new_vm_id>`: Creates a clone of an existing VM. Replace `<vm_id>` with the ID of the source VM and `<new_vm_id>` with the desired ID for the clone.

* **Configure VM Settings:**
    * `qm config <vm_id>`: Shows the current configuration options for a VM. Use the output along with the specific option and value to modify settings. Refer to the `qm config` man page for details.
    * `qm set <vm_id> <option>=<value>`: Sets a specific configuration option for a VM. Example: `qm set 100 memory=2048M` sets memory to 2048 MB for VM 100.

* **Advanced Options:**
   * `qm help`: Provides a full list of available `qm` commands and options.
   * `man qm`: Displays the manual page for the `qm` command with detailed explanations and examples.

**Remember:** Always replace `<vm_id>` with the actual ID of your virtual machine.

For in-depth information and a wider range of options, consult the Proxmox VE documentation: [https://pve.proxmox.com/pve-docs/qm.1.html](https://pve.proxmox.com/pve-docs/qm.1.html)