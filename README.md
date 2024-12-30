# USB-automount
### Structure du dépôt
##### usb-automount-smb/
 ├── udev-rules/10-usb-drive-automount.rules
 ├── smb.conf
 ├── README.md

# Contenu du fichier udev-rules/10-usb-drive-automount.rules

KERNEL!="sd[a-z][1-9]", GOTO="usb_drive_automount_end"

ENV{mount_point}="/media"

ENV{ID_FS_LABEL}!="", ENV{dir_name}="%k-%E{ID_FS_LABEL}"
ENV{ID_FS_LABEL}=="", ENV{dir_name}="%k"

# Création du dossier de montage
ACTION=="add", RUN+="/bin/mkdir -p '%E{mount_point}/%E{dir_name}'"

# Montage automatique avec systemd-mount
ACTION=="add", SUBSYSTEMS=="usb", SUBSYSTEM=="block", ENV{ID_FS_USAGE}=="filesystem", RUN{program}+="/usr/bin/systemd-mount --no-block --automount=yes --options=dmask=000,fmask=000 --owner=nobody --collect $devnode %E{mount_point}/%E{dir_name}"

# Démontage lors du retrait de la clé
ACTION=="remove", RUN{program}+="/usr/bin/systemd-umount '%E{mount_point}/%E{dir_name}'"
ACTION=="remove", ENV{dir_name}!="", RUN+="/bin/sh -c 'rm -rf \"%E{mount_point}/%E{dir_name}\"'"
ACTION=="remove", ENV{dir_name}!="", RUN+="/usr/bin/logger 'Suppression du dossier %E{mount_point}/%E{dir_name}'"

LABEL="usb_drive_automount_end"

# Contenu du fichier smb.conf

[USB-Share]
path = /media
browsable = yes
writable = yes
guest ok = yes
public = yes
create mask = 0777
directory mask = 0777
force user = nobody

# Contenu du fichier README.md

# USB Automount & SMB Share

Ce projet configure un système Linux (par exemple, Raspberry Pi) pour :

1. Monter automatiquement les clés USB lorsqu'elles sont branchées.
2. Partager ces clés USB sur le réseau grâce à Samba.

## Installation

### 1. Installer les prérequis

Assurez-vous que les outils suivants sont installés :

```bash
sudo apt update
sudo apt install samba systemd
```

### 2. Configurer udev pour l'automontage

Copiez le fichier `10-usb-drive-automount.rules` dans `/etc/udev/rules.d/` :

```bash
sudo cp udev-rules/10-usb-drive-automount.rules /etc/udev/rules.d/
```

Rechargez udev :

```bash
sudo udevadm control --reload-rules
```

### 3. Configurer Samba pour le partage réseau

Ajoutez la configuration suivante au fichier Samba (`/etc/samba/smb.conf`) :

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

Redémarrez le service Samba :

```bash
sudo systemctl restart smbd
```

### 4. Testez le partage réseau

Connectez une clé USB, puis accédez au dossier partagé via l'adresse réseau de votre machine (par exemple, `\\192.168.x.x\USB-Share`).

---

En cas de problème, consultez les journaux udev et Samba :

```bash
journalctl -u udev
journalctl -u smbd
