# Configuration VM n°1 : FOG Master Node

Ce fichier décrit la configuration recommandée pour le serveur principal (Web/DB) de ton LAB FOG.

## 1) Configuration réseau (Debian) — `/etc/network/interfaces`
Objectif : Avoir internet sur `ens192`, et tout le trafic FOG (PXE/Web) sur `ens224`. **Ne PAS mettre de gateway sur ens224**.

```bash
auto lo
iface lo inet loopback

# Carte 1 : INTERNET (Mises à jour)
auto ens192
iface ens192 inet dhcp
# Ou static si nécessaire :
# iface ens192 inet static
#     address 192.168.1.100
#     netmask 255.255.255.0
#     gateway 192.168.1.1

# Carte 2 : VLAN FOG (Réseau PXE) — PAS de gateway !
auto ens224
iface ens224 inet static
    address 192.168.66.251
    netmask 255.255.255.0
```

## 2) Installation FOG — réponses clés (installfog.sh)
- **Installation Type** : `N` (Normal / Master)
- **Quelle interface ?** : `ens224` (interface du VLAN FOG)
- **Setup a router address for the DHCP server?** : `No`
- **Setup a DNS address for the DHCP server?** : `No`
- **Would you like to use the FOG server for DHCP service?** : `Yes` (si tu veux que FOG fournisse DHCP sur le VLAN privé)

> Remarque : on ne fournit pas ici le fichier DHPC complet — FOG génère le fichier. Ces réponses garantissent que la
> configuration interne de FOG cible `ens224` sans exposer la gateway publique.

## 3) Post-install : vérifications essentielles
### A) `.fogsettings`
Vérifier `/opt/fog/.fogsettings` pour s'assurer que le mot de passe SQL ne contient pas de `#` ni d'autres caractères problématiques.
```bash
sudo nano /opt/fog/.fogsettings
# Vérifier : password='ton_mot_de_passe'
```

### B) FOG Settings (Interface Web)
Dans **FOG Configuration → FOG Settings**, modifier :
- `Web Server / WEB HOST` : `192.168.66.251`
- `TFTP Server / TFTP HOST` : `192.168.66.251`
- `FOG Server` et `FOG Storage` doivent être configurés avec les IP internes OK.

### C) Services utiles
```bash
# Vérifier services
systemctl status apache2
systemctl status mysql
systemctl status nfs-kernel-server  # si stockage local sur la même VM (rare)
```
