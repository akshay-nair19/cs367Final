# Why we chose this Red Hat lab setup

Our course project studies CVE-2000-0573, a formatting bug in the WU-FTPD file-transfer server. We wanted a historical system we could identify precisely, boot repeatedly, and observe from our own laptops. Red Hat Linux 6.2 was a useful choice because published accounts identify a specific WU-FTPD build on that release, and Red Hat published an advisory naming the package that fixed the bug. That gives us two versions we can compare. [Red Hat security advisory, June 2000](https://lwn.net/2000/0629/a/rh-wuftpd.php3)

For commands, follow the [QEMU setup guide](qemu-redhat-setup.md). The recorded results at the end of this page are what we verified. The installer choices below also explain the team's recommended settings where we did not capture the final checkbox state.

## Host, guest, and virtual hardware

| Choice | Reason |
|---|---|
| QEMU on Windows; Ubuntu in WSL 2 | Our laptop already ran Windows and WSL. QEMU could emulate the older PC hardware, while Ubuntu provided a separate terminal for observing the VM. We recorded QEMU 11.1.0 so teammates can compare their setup if behavior differs. |
| Red Hat Linux 6.2 “Zoot,” English i386 installation CD | This is the historical release for 32-bit Intel-compatible PCs. The i386 installation disc contains the operating system; the similarly named documentation and source discs serve different purposes. |
| PC machine, Pentium CPU, software emulation (`tcg`) | These choices give the old installer a familiar PC and processor model. Software emulation lets teammates request the same guest CPU even when their Windows PCs have different processors. |
| 128 MB RAM and a 4 GB virtual IDE disk | This gave our text-mode installation enough room to complete. IDE is a disk interface the installer recognized as `hda`. These are values that worked for our VM, rather than minimum requirements. |
| QCOW2 disk and one-time CD boot | QCOW2 stores the guest disk in a file that grows as data is written. Booting the CD once starts the installer; subsequent boots use the installed disk. |

[QEMU system options](https://www.qemu.org/docs/master/system/invocation.html) · [Red Hat Linux 6.2 installation guide](https://legacy.redhat.com/pub/redhat/linux/6.2/en/doc/RH-DOCS/pdf/rh62ig.pdf)

## Installer and disk choices

We chose the text installer and **Custom** so we could select packages for a server and use it entirely from the console. Disk Druid let us review sizes and mount points on one screen. Initializing the partition table made sense because we had just created an empty virtual disk.

| Partition | Reason |
|---|---|
| `/boot`, 16 MB, Linux native | Keeps the kernel and startup files near the beginning of the disk, a practical choice for the BIOS and boot loader of that era. |
| Swap, 128 MB, no mount point | Provides disk space Linux can use when memory is tight. We chose the same size as the guest's RAM. Swap does not hold a directory of files. |
| `/`, Linux native, grow to fill | Holds the operating system, programs, and logs. Giving it the remaining space avoids guessing how large each directory will grow. |

We selected the virtual disk `hda` for all three partitions and formatted the two Linux filesystems. We skipped the optional bad-block scan on the freshly created disk image. [Red Hat installation guide, §§4.4–4.5](https://legacy.redhat.com/pub/redhat/linux/6.2/en/doc/RH-DOCS/pdf/rh62ig.pdf)

## Boot choices and the fix we actually needed

We installed LILO in the virtual disk's master boot record because Red Hat was the only OS on that disk. We kept the default label `linux` and LILO's linear disk-addressing option. We skipped the rescue floppy because we retained the installation ISO and could copy the powered-off disk for recovery.

Initially we left the kernel options empty. On the first reboot, the guest repeatedly printed `hda: lost interrupt` during disk detection. Entering `linux noapic` at the LILO prompt let it reach login. We made that one change permanent in `/etc/lilo.conf`, ran `/sbin/lilo` to apply it, and verified an unattended boot. That before-and-after result is why this setting belongs in our guide. In the editor, **Shift+O** simply opens a blank line above the first line; it has no effect on the boot setting itself.

## Accounts, peripherals, and packages

| Installer choice | Reason |
|---|---|
| English and a keyboard layout matching the physical keyboard | Makes prompts and typed commands predictable. Teammates can choose the layout they actually type on. |
| Hostname `wuftpd-lab`; local timezone; hardware clock in UTC | The hostname identifies the VM in prompts and logs. The timezone controls displayed time; UTC gives the emulated clock a consistent reference. |
| Generic three-button PS/2 mouse; three-button emulation off | Describes the mouse presented by QEMU. The extra option is for simulating a middle button on a two-button mouse. |
| Local accounts, root password, shadow passwords | A regular account is sufficient for routine use. Root handles system changes. Shadow passwords restrict access to stored password hashes. |
| Historical MD5 password option; network authentication off | MD5 was an option offered by this release and is obsolete today. Local accounts cover this lab, which has no NIS, LDAP, or Kerberos infrastructure. |
| Anonymous FTP Server | Installs support for the service we are studying. We checked the installed server and anonymous FTP support separately, then tested an ordinary guest login. |
| Networked Workstation and Development groups | We selected these for network utilities and planned code analysis. Individual compiler or debugger packages should be checked before a task that needs them. |
| Desktop, dial-up, multimedia, extra servers, and “Everything” omitted from the proposed selection | The guest serves files and is administered from a console. It has no modem, audio workload, or desktop requirement. A personal workstation from the same period might reasonably use a different selection. |

[Red Hat installation guide, §§4.8–4.11](https://legacy.redhat.com/pub/redhat/linux/6.2/en/doc/RH-DOCS/pdf/rh62ig.pdf)

## What our checks established

| Observation | What it established |
|---|---|
| The guest reached login after a normal reboot. | The disk installation and saved boot option worked together. |
| RPM reported `wu-ftpd-2.6.0-3` and `anonftp-3.0-3`. | We can identify the exact packages. Red Hat's fixed build was also called 2.6.0, so the suffix matters. |
| A connection to `127.0.0.1` inside Red Hat returned `230 Guest login ok`. | Anonymous login worked locally inside the guest. |
| A line sent through the guest's serial port appeared in WSL. | We had a communication path for observing guest output while keeping the guest's network adapter disabled with `-nic none`. |

FTP access from WSL and experimental reproduction of the vulnerability remain unverified. The shutdown and backup steps in the setup guide let a teammate keep a known working disk before making further changes. Git tracks the guide and results; the ISO and VM disks stay outside the repository.
