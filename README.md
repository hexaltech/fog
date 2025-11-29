# ☁️ LAB FOG Project : Architecture Distribuée sur ESXi (Dual NIC)

Ce dépôt contient la documentation technique et les fichiers de configuration pour un LAB FOG (FOG Project)
séparant les rôles **Master** (Web/DB) et **Storage** (Images). Conçu pour être déployé sur des machines virtuelles
sous ESXi avec un réseau privé isolé (VLAN FOG).

## Structure du dépôt
- `VM1-Master-Config.md` — Configuration complète et recommandations pour la VM FOG‑Master.
- `VM2-Storage-Config.md` — Configuration complète et recommandations pour la VM FOG‑Storage (NFS).
- `DEBUG_CHEAT_SHEET.md` — Fiches de dépannage et commandes de debug utiles pour captures/déploiements.
- `LICENSE` — Licence recommandée (MIT) si tu veux rendre le dépôt public. (Optionnel)

## Règles importantes / Rappels
1. **Deux interfaces réseau** : une interface pour Internet (mises à jour) et une interface dédiée au VLAN FOG
   (PXE / NFS) — ne pas placer de gateway sur l'interface FOG.
2. **Ne PAS exposer la configuration DHCP** dans ce dépôt — FOG génère et gère le DHCP. Fournis seulement
   les informations nécessaires à l'interface FOG lors de l'installation.
3. **Permissions NFS** : les erreurs de montage `/images` sont presque toujours liées à `/etc/exports` ou aux
   permissions des dossiers `/images`.
4. **Drivers VM** : utiliser E1000e pour éviter des problèmes PXE avec certains noyaux.
5. **Sécurité** : évite les mots de passe contenant `#` ou caractères problématiques pour `.fogsettings`.

## Comment utiliser ce dépôt
1. Cloner le dépôt sur ta machine locale.
2. Lire `VM1-Master-Config.md` puis `VM2-Storage-Config.md`.
3. Avant d'effectuer des captures, suivre les étapes de `DEBUG_CHEAT_SHEET.md`.
4. Pour la vidéo : présenter l'architecture, puis montrer la configuration réseau, l'installation du Storage Node,
   et finir par une capture en mode debug.
