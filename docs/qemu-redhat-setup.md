# Running Red Hat Linux 6.2 in QEMU on Windows

The team used QEMU 11.1.0 for Windows to run Red Hat Linux 6.2 (Zoot). If you already installed Red Hat, skip to **Start the VM again**. Commands labeled **Windows** go in Command Prompt (`cmd.exe`). Commands labeled **Red Hat** go inside the QEMU window.

## First installation

**Windows.** Install [QEMU for Windows](https://www.qemu.org/download/#windows) and verify the emulator opens:

```cmd
"C:\Program Files\qemu\qemu-system-i386.exe" --version
```

Create a folder for the VM in your Windows user directory. Keep the disk and ISO here so they stay outside the Git repository.

```cmd
mkdir "%USERPROFILE%\wuftpd-lab"
cd /d "%USERPROFILE%\wuftpd-lab"
```

Place the English i386 installation ISO, `redhat-6.2-i386.iso`, in that folder. The [historical release archive](https://rpmfind.net/linux/redhat-archive/6.2/en/iso/i386/) lists the installation disc alongside separate documentation and source discs.

Create the virtual disk once. The guard skips creation if you already have a disk with this name.

```cmd
if not exist redhat62.qcow2 "C:\Program Files\qemu\qemu-img.exe" create -f qcow2 redhat62.qcow2 4G
```

Start the installer. Paste this as one line:

```cmd
"C:\Program Files\qemu\qemu-system-i386.exe" -machine pc -accel tcg -cpu pentium -m 128 -drive file=redhat62.qcow2,format=qcow2,if=ide -cdrom redhat-6.2-i386.iso -boot once=d -nic none
```

At the CD's `boot:` prompt, enter `text`. Choose **Custom** and **Disk Druid**. For the new 4 GB virtual disk (`hda`), use:

| Mount point | Type | Size |
|---|---|---|
| /boot | Linux native | 16 MB |
| Leave blank | Linux swap | 128 MB |
| / | Linux native | Remaining space; check “Grow to fill disk” |

Format the two Linux native partitions. Install LILO, the Linux boot loader, in the master boot record of the virtual disk (`/dev/hda`). Use `linux` as the default boot label. Choose your own root password. The VM can run without a graphical desktop.

## Start the VM again

**Windows.** These commands boot the saved installation with its network adapter disabled:

```cmd
cd /d "%USERPROFILE%\wuftpd-lab"
"C:\Program Files\qemu\qemu-system-i386.exe" -machine pc -accel tcg -cpu pentium -m 128 -drive file=redhat62.qcow2,format=qcow2,if=ide -nic none
```

**If startup repeatedly prints `hda: lost interrupt`:** Hold Shift as LILO starts and type `linux noapic` at its `boot:` prompt. This got our VM to the login screen. To keep the setting, log in as root, back up the configuration with `cp -p /etc/lilo.conf /etc/lilo.conf.bak`, then edit `/etc/lilo.conf`. In our configuration we added this line above the first `image=` entry:

```text
append="noapic"
```

Run `/sbin/lilo`. Our installation printed `Added linux *` and `Added linux-up`. After a successful update, run `reboot` and allow the VM to boot automatically.

## Verify the installation

**Red Hat.** Record the output from your own copy:

```sh
cat /etc/redhat-release
rpm -q wu-ftpd
rpm -q anonftp
```

Our VM reported Red Hat Linux 6.2, `wu-ftpd-2.6.0-3`, and `anonftp-3.0-3`. In our local FTP check, the server returned `230 Guest login ok` to an anonymous login on `127.0.0.1` inside Red Hat.

We also observed a message travel from Red Hat's virtual serial port to Ubuntu in WSL 2 in a separate test. The offline startup command above does not configure that serial connection. FTP access from WSL and CVE reproduction remain untested.

## Shut down and save a clean copy

**Red Hat.** If logged in as an ordinary user, run `su -` and enter the VM's root password. Then run:

```sh
shutdown -h now
```

Wait for the halt message, then close QEMU. **Windows:** In the lab folder, run this to save a first backup without overwriting an existing one:

```cmd
cd /d "%USERPROFILE%\wuftpd-lab"
if not exist redhat62-clean.qcow2 copy /b redhat62.qcow2 redhat62-clean.qcow2
```

Commit setup notes to Git. Keep passwords, installation media, working disks, and backup disks on your own machine.
