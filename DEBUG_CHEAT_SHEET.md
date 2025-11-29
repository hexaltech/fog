---

# 🛠️ DEBUG FOG — NFS, Capture & Déploiement

## 📖 Sommaire (Chapitres)

1. **Vérifier /etc/exports (NFS Storage)**
2. **Droits corrects /images**
3. **Recharger NFS proprement**
4. **Mode Debug (PXE)**
5. **Tests Réseau & HTTP**
6. **Tester le montage NFS**
7. **Commandes utiles côté MASTER**
8. **Commandes utiles côté STORAGE**
9. **Astuces rapides**

---

# 1️⃣ Vérifier `/etc/exports` (Fog Storage)

### ❌ Ancienne configuration (problème : `/images` en RO)

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
✔ `async` accélère la capture

---

# 2️⃣ Droits corrects `/images`

```bash
sudo chown -R fogproject:fogproject /images
sudo chmod -R 777 /images
```

---

# 3️⃣ Recharger NFS proprement

```bash
sudo exportfs -ra
sudo systemctl restart nfs-kernel-server
sudo exportfs -v
```

---

# 4️⃣ Mode Debug (Client PXE)

1. FOG Web UI → Host
2. Capture/Deploy → **Schedule as a debug task**
3. Boot PXE → Terminal :

```
[root@fogclient ~]#
```

---

# 5️⃣ Tests Réseau & HTTP

```bash
wget --spider http://<FOG-MASTER-IP>/fog/service/ipxe/boot.php
cat /proc/cmdline
```

Vérifier que :
`web=http://<FOG-MASTER-IP>/fog/`

---

# 6️⃣ Tester le montage NFS

```bash
mkdir -p /test
mount -o nolock <FOG-STORAGE-IP>:/images/dev /test
```

**Erreurs fréquentes :**

* `Permission denied` → /etc/exports mauvais
* `Connection refused` → NFS down
* Freeze capture → mot de passe SQL contenant `#`

---

# 7️⃣ Commandes utiles côté MASTER

```bash
tail -f /var/log/apache2/error.log
systemctl restart isc-dhcp-server
systemctl status FOGImageReplicator
systemctl status FOGTaskScheduler
```

---

# 8️⃣ Commandes utiles côté STORAGE

```bash
exportfs -ra
systemctl restart nfs-kernel-server
exportfs -v

ls -la /images
chmod -R 777 /images
chown -R fogproject:fogproject /images
```

---

# 9️⃣ Astuces rapides

* Vérifier **Web Host** & **TFTP Host**
* Tester VM avec **E1000e** & Secure Boot **OFF**
* Réplication lente → regarder **FOGImageReplicator**

---

Si tu veux, je te fais **un fichier .md téléchargeable** propre à importer dans GitHub.
