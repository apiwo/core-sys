# core-sys

Welcome to core-sys. This repository contains custom, highly optimized Linux system packages and scripts. The flagship component of this repository is **core-init**, a lightweight, bash-driven init system designed to completely replace complex, monolithic init systems (like systemd) with explicit, transparent, and user-controlled shell scripting. 

By taking absolute ownership of PID 1, core-init allows advanced users to strip their boot process down to the bare minimum, reducing overhead and providing an absolute understanding of exactly what services are running on their system.

---

## Core Philosophy

Modern init systems often abstract the boot process behind layers of configuration files, binary logs, and dependency trees. `core-sys` takes a different approach:
* **Absolute Transparency:** The entire boot sequence is a single, readable bash script. If it is not in the script, it does not run.
* **Minimal Footprint:** Designed for ultra-lean systems where disk space and memory usage are paramount.
* **Direct Control:** You define the exact execution order, the wait times, and the terminal output styling.
* **Tinkerer First:** Built specifically for users of source-based or heavily customized distributions (like Gentoo or Arch) who want to build their system architecture from the ground up.

---

## Installation: core-init

The `core-init` script must be placed in a directory accessible to the Linux kernel immediately upon boot, and your bootloader must be configured to pass it to the kernel as the primary init process.

### 1. Acquire the Script
Clone this repository or download the specific `core-init` flavor you wish to use (e.g., standard, testing, or parallelized/fast).

### 2. Place in System Binaries
Move the initialization script into a secure system binary directory. `/sbin` is the standard location for system initialization and administration binaries.

```bash
cp core-init /sbin/core-init
chmod +x /sbin/core-init
```
**Important:** The script *must* be marked as executable (`chmod +x`), otherwise the kernel will panic with a "No init found" error upon reboot.

### 3. Update the Bootloader (GRUB)
You must instruct the kernel to bypass its default init search path and explicitly load `core-init`. Open your default GRUB configuration file:

```bash
nano /etc/default/grub
```

Find the `GRUB_CMDLINE_LINUX_DEFAULT` line and append the `init` parameter pointing to your script's exact location. For example:

```text
GRUB_CMDLINE_LINUX_DEFAULT="quiet init=/sbin/core-init"
```
*(Note: Replace `core-init` with `core-init-testing` or your specific flavor if applicable).*

### 4. Regenerate GRUB Configuration
Apply the changes to your actual boot configuration file:

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

---

## How to Set Up and Configure

By default, the provided `core-init` scripts come populated with basic core processes and generic filesystem mount locations. Because every custom Linux build is unique, **you must edit the script to match your specific hardware and software stack before rebooting.**

Open `/sbin/core-init` in your preferred text editor to modify the boot sequence.

### Part 1: Filesystems and Mounts
The script handles mounting virtual filesystems (`/proc`, `/sys`, `/dev`) and your physical drives. You must locate the `mount` commands in the script and ensure the target partitions match your machine. 

If your root partition or other drives are specifically defined, replace the placeholder names.
* Example: Change generic placeholders to your actual block devices, such as `/dev/nvme0n1p1` or `/dev/sda2`.

### Part 2: Hardware and Drivers
Ensure your necessary kernel modules are loaded early in the script. If you require specific graphics drivers (such as proprietary Nvidia modules) or networking modules, configure the `modprobe` section accordingly to ensure hardware nodes are established before the display manager starts.

### Part 3: The Anatomy of a Service Block
The core strength of this init system is that starting a "service" is simply running a command. You can control exactly how each service launches using standard bash syntax.

Here is the breakdown of a standard service initialization block within the script:

```bash
/usr/lib/elogind/elogind &
sleep 2
echo -e "  \e[1;32m[ OK ]\e[0m Started elogind"
```

**Understanding the components:**
1. **The Process (`/usr/lib/elogind/elogind &`):** This is the absolute path to the binary you want to run. The ampersand (`&`) at the end is critical for most services; it tells the script to push this process to the background so the rest of the boot sequence can continue. If you omit the `&`, the boot sequence will pause entirely until that program closes.
2. **The Wait Time (`sleep 2`):** Some services require a moment to initialize, bind to a socket, or create a PID file before dependent services can launch. You can use `sleep` for arbitrary wait times, or advanced `while` loops to check for the existence of specific sockets (e.g., waiting for the DBus socket to appear).
3. **The Status Output (`echo -e ...`):** This prints a formatted, color-coded message to TTY1. `\e[1;32m` initiates bright green text, and `\e[0m` resets the terminal formatting. This allows you to build a visually clean, professional boot splash directly in the console.

### Part 4: TTY Management and Desktop Environments
By default, the script takes ownership of TTY1 to output boot logs. It is highly recommended to allocate TTY2 for an emergency shell using `agetty` (e.g., `setsid /sbin/agetty -8 tty2 linux &`), and let your Display Manager (like SDDM or LightDM) automatically claim TTY3 or higher.

---

## Power Management (Safe Reboot and Shutdown)

Because `core-init` replaces standard init systems, default `reboot` and `poweroff` commands will no longer function properly, as the kernel protects PID 1 from standard termination signals.

To allow for graceful shutdowns, the `core-init` script is programmed to listen for custom user signals (`SIGUSR1` for reboot, `SIGUSR2` for shutdown). 

You should create wrapper scripts in `/usr/local/bin/` to send these signals. For example, a custom `reboot` script would look like this:

```bash
#!/bin/bash
if [ "$EUID" -ne 0 ]; then
    echo "Must be run as root."
    exit 1
fi
echo "Rebooting..."
sleep 1
kill -SIGUSR1 1
```

---

## Disclaimer

Replacing PID 1 with a custom shell script gives you ultimate power over your system, but it also removes standard safety nets. Ensure you have a live USB or a secondary working kernel entry in your bootloader available in case of syntax errors or boot failures during your initial setup phase.
