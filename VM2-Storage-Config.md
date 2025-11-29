# Configuration VM n°2 : FOG Storage Node

VM dédiée au stockage des images (`/images`) et au service NFS.

## 1) Configuration réseau (Debian) — `/etc/network/interfaces`
```bash
auto lo
iface lo inet loopback

# Carte 1 : INTERNET
auto ens192
iface ens192 inet dhcp

# Carte 2 : VLAN FOG (Stockage) — PAS de gateway !
auto ens224
iface ens224 inet static
    address 192.168.66.252
    netmask 255.255.255.0
```

## 2) Installation FOG (Storage Node)
- **Installation Type** : `S` (Storage Node)
- **Interface** : `ens224`
- **IP Address of the FOG Server (Master)** : `192.168.66.251`

Pendant l'installation, FOG te demandera l'IP du Master pour lier le Storage Node.

## 3) NFS — `/etc/exports` (Recommandé)
Les erreurs de montage `/images/dev` viennent souvent d'une export NFS trop restrictive. Exemple testé et fonctionnel :

```
/images *(ro,sync,no_wdelay,insecure_locks,no_root_squash,insecure,fsid=0)
/images/dev *(rw,async,no_wdelay,no_root_squash,insecure,fsid=1)
```

Appliquer les changements :
```bash
sudo exportfs -ra
sudo systemctl restart nfs-kernel-server
```

## 4) Permissions et ownership
Assurer que l'utilisateur `fogproject` a la main sur `/images` :
```bash
sudo chmod -R 777 /images
sudo chown -R fogproject:fogproject /images
```

**Remarque** : `chmod 777` est permissif et acceptable pour un LAB isolé. En production, restreindre les droits.

## 5) Tests rapides
Sur un client ou dans un chroot PXE debug :
```bash
mkdir /test
mount -o nolock 192.168.66.252:/images/dev /test
ls -la /test
```

Si `mount` renvoie `permission denied` vérifier `/etc/exports` et `no_root_squash`. Si `connection refused` vérifier `systemctl status nfs-kernel-server`.
