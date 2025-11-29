Voici **le fichier complet en Markdown pur**, prêt à copier-coller :

---

````markdown
# 🛠️ DEBUG FOG — NFS, Capture & Déploiement (Cheat Sheet Complète)

## 📌 1. Vérifier /etc/exports (Fog Storage)

### Ancienne configuration que tu avais :
```bash
/images *(ro,sync,no_wdelay,subtree_check,insecure_locks,all_squash,anonuid=1001,anongid=1001,fsid=0)
/images/dev *(rw,async,no_wdelay,subtree_check,all_squash,anonuid=1001,anongid=1001,fsid=1)
````

Cette config bloque la capture car `/images` est en **RO**.

### Configuration corrigée recommandée :

```bash
/images *(rw,async,no_wdelay,no_subtree_check,insecure_locks,all_squash,anonuid=1001,anongid=1001,fsid=0)
/images/dev *(rw,async,no_wdelay,no_subtree_check,all_squash,anonuid=1001,anongid=1001,fsid=1)
```

* `rw` obligatoire
* `no_subtree_check` évite les freezes NFS
* `async` améliore la vitesse de capture

---

## 📌 2. Droits corrects /images

```bash
sudo chown -R fogproject:fogproject /images
sudo chmod -R 777 /images
```

---

## 📌 3. Recharger NFS proprement

```bash
sudo exportfs -ra
sudo systemctl restart nfs-kernel-server
sudo exportfs -v
```

Tu dois voir **no_subtree_check** sur les deux exports.

---

# 🧪 Mode Debug (Client PXE)

## Activer le Debug Task

1. FOG Web UI → Host → Capture/Deploy
2. Cocher : **Schedule as a debug task**
3. Boot PXE → tu arrives sur un shell :

```
[root@fogclient ~]#
```

---

# 🌐 Tests Réseau & HTTP

```bash
# Vérifier accès au boot.php
wget --spider http://<FOG-MASTER-IP>/fog/service/ipxe/boot.php

# Voir les arguments envoyés au client
cat /proc/cmdline
# Chercher : web=http://<FOG-MASTER-IP>/fog/
```

---

# 📦 Tester Montage NFS (Mount Failed)

```bash
mkdir -p /test
mount -o nolock <FOG-STORAGE-IP>:/images/dev /test
```

### Si erreur :

* `Permission denied` → problème de /etc/exports ou permissions
* `Connection refused` → NFS down sur le Storage
* Freeze PXE ou tâches bloquées → mot de passe SQL contenant `#` dans `.fogsettings`

---

# 🖥️ Commandes utiles côté MASTER

```bash
# Logs Apache
sudo tail -f /var/log/apache2/error.log

# Redémarrer DHCP (si FOG gère le DHCP)
sudo systemctl restart isc-dhcp-server

# Services FOG
sudo systemctl status FOGImageReplicator
sudo systemctl status FOGTaskScheduler
```

---

# 🗄️ Commandes utiles côté STORAGE

```bash
# Recharger NFS
sudo exportfs -ra
sudo systemctl restart nfs-kernel-server
sudo exportfs -v

# Vérifier permissions
ls -la /images
chmod -R 777 /images
chown -R fogproject:fogproject /images
```

---

# ⚡ Astuces rapides

* Si l’IP Web dans FOG est incorrecte → le client ne check-in pas.
  → FOG Settings → Web Host / TFTP Host
* Tester une VM client en **E1000e** avec Secure Boot OFF pour diagnostiquer le PXE.
* Si réplication KO ou lente → vérifier `FOGImageReplicator` côté Master.

```

---

Si tu veux, je te le génère aussi en **fichier téléchargeable (.md)** ou en **PDF**.
```
