# USB-automount

## USB Automount & SMB Share

This project sets up a Linux system (e.g., Raspberry Pi) to:

1. Automatically mount USB drives when plugged in.
2. Share these USB drives over the network using Samba.

### Summary

This configuration ensures that any USB drive plugged into the system is automatically mounted and made accessible over the network for other devices. It uses udev rules for mounting and Samba for network sharing.

### Installation

#### 1. Install Prerequisites

Ensure the following tools are installed:

```bash
sudo apt update
sudo apt install samba systemd
```

#### 2. Configure udev for Automounting

Copy the file `10-usb-drive-automount.rules` to `/etc/udev/rules.d/`:

```bash
sudo cp udev-rules/10-usb-drive-automount.rules /etc/udev/rules.d/
```

Reload udev:

```bash
sudo udevadm control --reload-rules
```

#### 3. Configure Samba for Network Sharing

Add the following configuration to the Samba configuration file (`/etc/samba/smb.conf`):

```
[USB-Share]
path = /media
browsable = yes
writable = yes
guest ok = yes
public = yes
create mask = 0777
directory mask = 0777
force user = nobody
```

Restart the Samba service:

```bash
sudo systemctl restart smbd
```

#### 4. Test the Network Share

Plug in a USB drive, then access the shared folder via the network address of your machine (e.g., `\\192.168.x.x\USB-Share`).

---

If issues arise, check the logs for udev and Samba:

```bash
journalctl -u udev
journalctl -u smbd
```
