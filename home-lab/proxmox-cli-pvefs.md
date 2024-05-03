## Proxmox VE pvefs Commands

The `pvefs` command-line tool in Proxmox VE is used to manage the Proxmox VE filesystem (`pvefs`). Here's a summary of common `pvefs` commands:

**List Files and Directories:**

* `pvefs ls [<path>]`: Lists the contents of the specified directory or the current directory if no path is given.

* `pvefs tree [<path>]`: Displays a tree-like structure of the specified directory or the current directory if no path is given.

**Create Directories:**

* `pvefs mkdir <path>`: Creates a directory at the specified path.

**Remove Files and Directories:**

* `pvefs rm <path>`: Removes the specified file or directory.
* `pvefs rmdir <path>`: Removes the specified empty directory.

* `pvefs rf <path>`: **Use with caution!** Recursively removes the specified directory and all its contents.

**Copy Files:**

* `pvefs cp <source_path> <destination_path>`: Copies the specified file to the destination path.

**Move Files:**

* `pvefs mv <source_path> <destination_path>`: Moves the specified file to the destination path.

**Create Symbolic Links:**

* `pvefs ln -s <source_path> <link_path>`: Creates a symbolic link at `<link_path>` pointing to `<source_path>`.

**Change File Permissions:**

* `pvefs chmod <permissions> <path>`: Modifies the file permissions of the specified file or directory. Use octal notation for permissions (e.g., `755`).

**Change File Ownership:**

* `pvefs chown <user>:<group> <path>`: Changes the ownership of the specified file or directory to the given user and group.

**Additional Options:**

* `pvefs help`: Displays help information for `pvefs` commands.
* `man pvefs`: Provides the man page for the `pvefs` command with detailed explanations and examples.

**Remember:**

* `pvefs` operates on the Proxmox VE filesystem, which is used to store VM and container configurations, backups, and other data.
* Use `pvefs` with caution, especially when modifying permissions or ownership, as it can affect system operations.

For comprehensive details and advanced usage, refer to the Proxmox VE documentation: [премахнат e невалиден URL адрес]
