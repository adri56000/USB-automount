# USB-automount

## USB Automount & SMB Share

Ce projet configure un système Linux (par exemple, Raspberry Pi) pour :

1. Monter automatiquement les clés USB lorsqu'elles sont branchées.
2. Partager ces clés USB sur le réseau grâce à Samba.

### Installation

#### 1. Installer les prérequis

Assurez-vous que les outils suivants sont installés :

```bash
sudo apt update
sudo apt install samba systemd
```

#### 2. Configurer udev pour l'automontage

Copiez le fichier `10-usb-drive-automount.rules` dans `/etc/udev/rules.d/` :

```bash
sudo cp udev-rules/10-usb-drive-automount.rules /etc/udev/rules.d/
```

Rechargez udev :

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger
```

#### 3. Configurer Samba pour le partage réseau

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

#### 4. Testez le partage réseau

Connectez une clé USB, puis accédez au dossier partagé via l'adresse réseau de votre machine (par exemple, `\\192.168.x.x\USB-Share`).

---

En cas de problème, consultez les journaux udev et Samba :

```bash
journalctl -u udev
journalctl -u smbd
