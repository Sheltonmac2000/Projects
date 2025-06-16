# Junos OS Python Script Installation and Configuration Guide

This guide outlines the steps to copy Python scripts to a Junos OS switch, set appropriate permissions, and configure the system to run them.

## 1. Copy and Prepare Scripts

### Copy Scripts to the Appropriate Directory
For op scripts (most common for general tasks), copy the scripts to the `/var/db/scripts/op/` directory:

```bash
root@switch% cp /var/tmp/usb/your_script1.py /var/db/scripts/op/
root@switch% cp /var/tmp/usb/your_script2.py /var/db/scripts/op/
```

**Note**: For commit or event scripts, use the respective directories:
- Commit scripts: `/var/db/scripts/commit/`
- Event scripts: `/var/db/scripts/event/`

### Set Permissions
Junos OS requires specific ownership and permissions for Python scripts for security:

```bash
root@switch% chmod 700 /var/db/scripts/op/your_script1.py
root@switch% chown root:wheel /var/db/scripts/op/your_script1.py
root@switch% chmod 700 /var/db/scripts/op/your_script2.py
root@switch% chown root:wheel /var/db/scripts/op/your_script2.py
```

This sets permissions to read, write, and execute only for the root user, with `root:wheel` as the owner.

### Unmount the USB Drive
After copying the scripts, unmount the USB drive:

```bash
root@switch% umount /var/tmp/usb
```

### Exit the Shell
Return to the operational mode:

```bash
root@switch% exit
user@switch>
```

## 2. Configure and Run the Scripts

Once the scripts are on the switch, configure Junos OS to recognize and allow their execution.

### Enter Configuration Mode
Enter configuration mode using the `private` option to avoid impacting others in a shared environment:

```bash
user@switch> configure private
```

### Enable Python as the Scripting Language
If not already enabled, set Python as the scripting language:

```bash
set system scripts language python
```

**Important Note**: Junos OS 20.x generally defaults to Python 2.7. If your scripts require Python 3, specify it:

```bash
set system scripts language python3
```

Verify your Junos version’s Python support to ensure compatibility.

### Enable Your Op Script(s)
Register the scripts with Junos OS:

```bash
set system scripts op file your_script1.py
set system scripts op file your_script2.py
```

### (Optional) Add Aliases
For easier execution, assign aliases to your scripts:

```bash
set system scripts op file your_script1.py command myscript1
set system scripts op file your_script2.py command myscript2
```

### Commit the Changes
Apply the configuration:

```bash
commit
```

### Run Your Script
From operational mode, execute the scripts using either the alias or filename:

- With aliases:
  ```bash
  user@switch> op myscript1
  user@switch> op myscript2
  ```

- Without aliases:
  ```bash
  user@switch> op your_script1.py
  user@switch> op your_script2.py
  ```