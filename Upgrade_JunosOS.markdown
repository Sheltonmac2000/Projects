# Guide on how to update the JUNOS OS

This guide provides steps to perform a Junos OS upgrade after a shutdown, including mounting a USB, running the upgrade, and handling recovery or troubleshooting scenarios.

## 1. Retry the Upgrade (After Shutdown)

After restarting the switch, follow these steps to retry the upgrade.

### Mount the USB
Mount the USB drive to access the upgrade file:

```bash
cli
show system storage
mkdir /var/tmp/usb
mount_msdosfs /dev/da0s1 /var/tmp/usb
```

### Verify the .tgz File
Confirm the presence of the `.tgz` file on the USB:

```bash
ls /var/tmp/usb
```

### Run the Upgrade
**This is the most important command**
Execute the upgrade command in operational mode:

```bash
request system software add /var/tmp/usb/junos-install-media-usb-ex-23.4R1.8-signed.tgz no-copy no-validate
```

For a Virtual Chassis setup, include the `all-members` option:

```bash
request system software add /var/tmp/usb/junos-install-media-usb-ex-23.4R1.8-signed.tgz all-members no-copy
```

Monitor the console for progress messages (e.g., “Uncompressing package…,” “Installing package…”). **Do not press Ctrl+C during the process.**

## 2. Important Notes

- **Safe Shutdown**: Always use `request system halt` before powering off the switch to prevent file system damage, especially after an interrupted upgrade.
- **.tgz File**: Ensure the USB contains an unextracted `.tgz` file, not a folder. Download the correct file from [Juniper’s website](https://www.juniper.net) if needed.
- **Serial Console**: Use Tera Term or CoolTerm for reliable monitoring. If the console appears frozen, press **Ctrl+Q** to check for an Xoff issue.
- **Risk of Corruption**: An interrupted upgrade (e.g., via Ctrl+C) may leave partial files. Follow cleanup steps to mitigate this. If the switch fails to boot, proceed to recovery.

## 3. Recovery (If Needed)

If the switch fails to boot after power-on, perform a recovery:

1. Insert the USB with the `.tgz` file.
2. Interrupt the bootloader (U-Boot) by pressing any key within 3–5 seconds.
3. For EX series switches, run:

```bash
usb start
run usbboot
```

4. Follow Juniper’s recovery guide (e.g., [KB16652](https://support.juniper.net/support/kb16652)).

## 4. Troubleshooting

### CLI Unresponsive
If `request system halt` fails, try:

```bash
request system power-off
```

As a last resort, use the physical power switch/button (note: this is risky).

Verify serial settings in Tera Term or CoolTerm:
- Baud rate: 9600
- Data bits: 8
- Parity: None
- Stop bits: 1

### Switch Doesn’t Shut Down
Check for stuck processes:

```bash
show system processes extensive
```

Terminate any `install` or `pkg` processes if necessary. If the switch remains unresponsive, unplug the power cable (risky).

### Post-Shutdown Issues
If the switch doesn’t boot, use the USB recovery process described above.

Verify the `.tgz` file’s integrity before retrying:

```bash
file checksum md5 /var/tmp/usb/junos-install-media-usb-ex-23.4R1.8-signed.tgz
```

## 5. Next Steps

1. **Shut Down**: Run `request system halt` and confirm the switch powers off (LEDs off, fans stopped).
2. **Retry Upgrade**: After restarting, retry the upgrade and monitor the console closely.
3. **Report Details**: If the shutdown fails, the switch doesn’t boot, or the upgrade fails again, provide:
   - Switch model (e.g., EX3400).
   - Exact `.tgz` file name and Junos version.
   - Console output or error messages.
   - Output of:

```bash
show system processes extensive | match install
```