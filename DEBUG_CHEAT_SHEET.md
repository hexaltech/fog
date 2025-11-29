# 🛠️ Guide de Dépannage & Commandes Debug

Fiche rapide pour diagnostiquer problèmes de capture / déploiement FOG.

## Mode Debug (Client PXE)
1. Dans l'interface FOG → Host → Capture/Deploy → cocher **Schedule as a debug task**.
2. Boote le client en PXE : tu obtiendras un shell `[root@fogclient ~]#`.

## Tests réseau & HTTP
```bash
# Tester l'accès à la ressource iPXE
wget --spider http://192.168.66.251/fog/service/ipxe/boot.php

# Vérifier les arguments de boot
cat /proc/cmdline
# Chercher : web=http://192.168.66.251/fog/
```

## Montage NFS (Mount Failed)
```bash
# Test de montage manuel
mkdir -p /test
mount -o nolock 192.168.66.252:/images/dev /test

# Si ok : la commande ne renvoie rien. Sinon lire l'erreur.
```

Causes fréquentes :
- `Permission denied` → vérifier `/etc/exports` (no_root_squash) et droits sur /images.
- `Connection refused` → service NFS arrêté sur Storage Node.
- `Stalled during boot` → mot de passe SQL avec `#` dans `/opt/fog/.fogsettings`.

## Commandes utiles côté Master
```bash
# Logs Apache
sudo tail -f /var/log/apache2/error.log

# Redémarrer DHCP (si géré)
sudo systemctl restart isc-dhcp-server

# Vérifier FOG services
sudo systemctl status FOGImageReplicator
sudo systemctl status FOGTaskScheduler
```

## Commandes utiles côté Storage
```bash
# Re-exporter et redémarrer NFS
sudo exportfs -ra
sudo systemctl restart nfs-kernel-server

# Vérifier exports
sudo exportfs -v

# Droits /images
sudo ls -la /images
sudo chmod -R 777 /images
sudo chown -R fogproject:fogproject /images
```

## Astuces rapides
- Si l'IP Web n'est pas la bonne dans FOG → le client ne peut pas check-in. Vérifier FOG Settings → Web Host / TFTP Host.
- En cas d'erreurs étranges sur le boot PXE, tester avec une VM temporaire client en E1000e et secure boot désactivé.
