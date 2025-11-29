---

# 🛠️ DEBUG FOG — NFS, Capture & Déploiement (Cheat Sheet Complète)

## 📌 1. Vérifier `/etc/exports` (Fog Storage)

### ❌ Ancienne configuration (posait problème : `/images` en RO)

```bash
/images *(ro,sync,no_wdelay,subtree_check,insecure_locks,all_squash,anonuid=1001,anongid=1001,fsid=0)
/images/dev *(rw,async,no_wdelay,subtree_check,all_squash,anonuid=1001,anongid=1001,fsid=1)
```

### ✅ Configuration corrigée recommandée

```bash
/images *(rw,async,no_wdelay,no_subtree_check,insecure_locks,all_squash,anonuid=1001,anongid=1001,fsid=0)
/images/dev *(rw,async,no_wdelay,no_subtree_check,all_squash,anonuid=1001,anongid=1001,fsid=1)
```

✔ `rw` obligatoire
✔ `no_subtree_check` évite les freezes NFS
✔ `async` améliore la vitesse de capture

---

## 📌 2. Droits corrects `/images`

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

---

# 🧪 Mode Debug (Client PXE)

## Activer le Debug Task

1. FOG Web UI → Host → Capture/Deploy
2. Cocher : **Schedule as a debug task**
3. Boot PXE → tu arrives sur :

```
[root@fogclient ~]#
```

---

# 🌐 Tests Réseau & HTTP

```bash
wget --spider http://<FOG-MASTER-IP>/fog/service/ipxe/boot.php
cat /proc/cmdline
```

Vérifier que :
`web=http://<FOG-MASTER-IP>/fog/`

---

# 📦 Tester Montage NFS (Mount Failed)

```bash
mkdir -p /test
mount -o nolock <FOG-STORAGE-IP>:/images/dev /test
```

### Causes fréquentes

* `Permission denied` → mauvais `/etc/exports` ou permissions
* `Connection refused` → service NFS down
* Freeze capture → mot de passe SQL contenant `#` dans `.fogsettings`

---

# 🖥️ Commandes utiles côté MASTER

```bash
tail -f /var/log/apache2/error.log
systemctl restart isc-dhcp-server
systemctl status FOGImageReplicator
systemctl status FOGTaskScheduler
```

---

# 🗄️ Commandes utiles côté STORAGE

```bash
exportfs -ra
systemctl restart nfs-kernel-server
exportfs -v

ls -la /images
chmod -R 777 /images
chown -R fogproject:fogproject /images
```

---

# ⚡ Astuces rapides

* Vérifier **Web Host** & **TFTP Host** dans FOG Settings
* Tester un client PXE avec **E1000e** + Secure Boot **OFF**
* Si réplication lente → vérifier **FOGImageReplicator**

---

Si tu veux, je te génère **un fichier .md téléchargeable**, ou je te le mets dans **un ZIP GitHub**.
