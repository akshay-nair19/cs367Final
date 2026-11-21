# WU_FTPD Final Project

We are studying CVE-2000-0573 in the WU-FTPD file transfer server.

## Current Progress

- Installed Red Hat Linux 6.2 in QEMU on Windows
- Confirmed that the VM boots with the 'noapic' setting
- Verified the installed server package: 'wu-ftpd-2.6.0-3'
- Confirmed anonymous FTP login form inside the VM
- Verified that Ubuntu in WSL 2 Received a message Over QEMU's serial port

The vulnerability itself has still not been tested


[Set up and run the Red Hat VM on Windows](docs/qemu-redhat-setup.md)