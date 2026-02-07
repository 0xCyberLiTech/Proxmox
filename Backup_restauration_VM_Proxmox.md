<div align="center">
  
  <br></br>
  
  <a href="https://github.com/0xCyberLiTech">
    <img src="https://readme-typing-svg.herokuapp.com?font=JetBrains+Mono&size=50&duration=6000&pause=1000000000&color=FF0048&center=true&vCenter=true&width=1100&lines=%3EPROXMOX_" alt="Titre dynamique FIREWALL" />
  </a>
  
  <br></br>

  <h2>Laboratoire numérique pour la cybersécurité, Linux & IT.</h2>

  <p align="center">
    <a href="https://0xcyberlitech.github.io/">
      <img src="https://img.shields.io/badge/Portfolio-0xCyberLiTech-181717?logo=github&style=flat-square" alt="🌐 Portfolio" />
    </a>
    <a href="https://github.com/0xCyberLiTech">
      <img src="https://img.shields.io/badge/Profil-GitHub-181717?logo=github&style=flat-square" alt="🔗 Profil GitHub" />
    </a>
    <a href="https://github.com/0xCyberLiTech/Firewall/releases/latest">
      <img src="https://img.shields.io/github/v/release/0xCyberLiTech/Firewall?label=version&style=flat-square&color=blue" alt="📦 Dernière version" />
    </a>
    <a href="https://github.com/0xCyberLiTech/Firewall/blob/main/CHANGELOG.md">
      <img src="https://img.shields.io/badge/📄%20Changelog-Firewall-blue?style=flat-square" alt="📄 CHANGELOG Firewall" />
    </a>
    <a href="https://github.com/0xCyberLiTech?tab=repositories">
      <img src="https://img.shields.io/badge/Dépôts-publics-blue?style=flat-square" alt="📂 Dépôts publics" />
    </a>
    <a href="https://github.com/0xCyberLiTech/Firewall/graphs/contributors">
      <img src="https://img.shields.io/badge/👥%20Contributeurs-cliquez%20ici-007ec6?style=flat-square" alt="👥 Contributeurs Firewall" />
    </a>
  </p>
  
</div>

<div align="center">
  <img src="https://img.icons8.com/fluency/96/000000/cyber-security.png" alt="CyberSec" width="80"/>
</div>

<div align="center">
  <p>
    <strong>Cybersécurité</strong> <img src="https://img.icons8.com/color/24/000000/lock--v1.png"/> • <strong>Linux Debian</strong> <img src="https://img.icons8.com/color/24/000000/linux.png"/> • <strong>Sécurité informatique</strong> <img src="https://img.icons8.com/color/24/000000/shield-security.png"/>
  </p>
</div>

---

<div align="center">
  
## À propos & Objectifs.

</div>

Ce projet propose des solutions innovantes et accessibles en cybersécurité, avec une approche centrée sur la simplicité d’utilisation et l’efficacité. Il vise à accompagner les utilisateurs dans la protection de leurs données et systèmes, tout en favorisant l’apprentissage et le partage des connaissances.

Le contenu est structuré, accessible et optimisé SEO pour répondre aux besoins de :
- 🎓 Étudiants : approfondir les connaissances
- 👨‍💻 Professionnels IT : outils et pratiques
- 🖥️ Administrateurs système : sécuriser l’infrastructure
- 🛡️ Experts cybersécurité : ressources techniques
- 🚀 Passionnés du numérique : explorer les bonnes pratiques

---

# Sauvegarde et restauration Proxmox — guide détaillé

Objectif
- Fournir une procédure claire, complète et pédagogique pour sauvegarder et restaurer des VMs/QEMU et conteneurs LXC sur Proxmox VE.

Prérequis
- Accès root sur le nœud Proxmox.
- Proxmox VE installé (pve-manager, pvesm, vzdump disponibles).
- Espace disque suffisant sur le stockage de sauvegarde.
- Pour snapshots cohérents des VM : installez `qemu-guest-agent` dans les VMs.

1) - Préparer le stockage

- Stockage local simple (ex. `/backup`) :

```bash
sudo mkdir -p /backup
sudo chown root:root /backup
sudo chmod 750 /backup
```

- Ajouter comme storage dans Proxmox (CLI) :

```bash
pvesm add dir backup --path /backup --content backup --nodes pve --maxfiles 7
```

- Exemples pour autres types de stockage :

```bash
# NFS
pvesm add nfs backup-nfs --server 10.0.0.5 --export /exports/backups --path /mnt/backup-nfs --content backup --maxfiles 14

# LVM Thin (pour stocker sur un volume group LVM)
pvesm add lvmthin local-thin --vgname vg_backup --thinpool data --content images,backup
```

- Conseils :
- Pour stockage distant, montez via `/etc/fstab` ou utilisez `pvesm add nfs`/`cifs` pour intégration Proxmox.
- Ne placez pas de sauvegardes critiques sur `local-lvm` si vous craignez la perte du nœud.

2) - Modes et options de `vzdump`

- Modes :
	- `snapshot` : prise d'un snapshot cohérente (recommandé pour VM) — nécessite stockage compatible (LVM snapshot ou qemu/KVM) ; `qemu-guest-agent` améliore la cohérence.
	- `suspend` : met en pause la VM, sauvegarde, puis reprend.
	- `stop` : arrête la VM pour sauvegarde (plus sûr mais interruption de service).

- Compression : `zstd` (bon compromis vitesse/taux), `lzo` (rapide, moins compressé). Exemple : `--compress zstd`.
- Autres options utiles :
	- `--dumpdir /chemin` : écrire directement dans un répertoire
	- `--exclude-path /chemin` : exclure des chemins (pour CT)
	- `--ionice 3` : réduire priorité I/O
	- `--mailto adresse@ex` : envoi d'alerte par mail

Exemples :

```bash
# Sauvegarde d'une VM (ID 101)
vzdump 101 --mode snapshot --compress zstd --storage backup --ionice 3 --mailto mm.sab8572@outlook.fr

# Sauvegarde de toutes les VMs (nœud)
vzdump --all --mode snapshot --compress zstd --storage backup --ionice 3

# Sauvegarde directe dans /backup sans ajouter de storage
vzdump 101 --mode snapshot --compress zstd --dumpdir /backup
```

3) - Planification — cron ou GUI

- Fichier `/etc/cron.d/vzdump-backup` (cron) :

```bash
# Sauvegarde quotidienne de toutes les VMs à 02:00
0 2 * * * root vzdump --all --mode snapshot --compress zstd --storage backup --mailto mm.sab8572@outlook.fr
```

```bash
sudo chmod 644 /etc/cron.d/vzdump-backup
sudo systemctl restart cron.service
```

- Option GUI (Datacenter → Backup → Add) :
	- ID: `backup-daily`
	- Nodes: sélectionnez le/les nœuds
	- Selection mode: `All` ou liste manuelle
	- Storage: `backup`
	- Schedule: `0 2 * * *` (ou sélection via l'UI)
	- Mode: `snapshot`
	- Compression: `zstd`
	- Max backups: `7`

4) - Rotation et rétention

- `--maxfiles` côté `pvesm add dir` permet la suppression automatique des anciens backups stockés via ce storage.
- Si vous écrivez dans un répertoire directement (`--dumpdir`), implémentez un script de rotation ou `find /backup -type f -mtime +14 -delete`.

5) - Vérifications après sauvegarde

- Lister les fichiers :

```bash
ls -lh /backup
```

- Lister le storage Proxmox :

```bash
pvesm list backup
```

- Logs `vzdump` :

```bash
ls -l /var/log/vzdump/
tail -n 200 /var/log/vzdump/*.log
```

- Vérifier l'espace disque :

```bash
df -h /backup
du -sh /backup/* | sort -h
```

6) - Restauration — étapes et options

- Restaurer une VM QEMU :

```bash
# Restaurer le backup vers la VMID 201 (adapter chemin/fichier)
qmrestore /backup/vzdump-qemu-101-2026_02_07-02_00_00.vma.zst 201 --storage local-lvm
```

- Options utiles :
	- `--storage` : destination des disques restaurés
	- `--force` : forcer l'écrasement si VMID existe

- Restaurer un conteneur LXC :

```bash
pct restore 202 /backup/vzdump-lxc-102-2026_02_07-02_00_00.tar.gz
```

- Bonnes pratiques de test :
	- Restaurez d'abord sur un VMID/CTID de test.
	- Vérifiez les services, la connectivité réseau et l'intégrité des données après restauration.

7) - Dépannage courant

- Échec de sauvegarde : consulter `/var/log/vzdump/*.log` et `journalctl -u pve-cluster`.
- Problèmes de cohérence : installer/activer `qemu-guest-agent` dans la VM.
- Cron non exécuté : vérifier permissions (`644`) et logs système (`journalctl -u cron` ou `/var/log/syslog`).
- Problème d'envoi mail : tester avec `sendmail`/`mail` ou config MTA (`postfix`, `msmtp`) sur le nœud.

Commandes de test rapide mail (ex. `msmtp` ou `sendmail` selon config) :

```bash
echo "Test email" | mail -s "Test backup" mm.sab8572@outlook.fr
```

8) - Sécurité et bonnes pratiques

- Limitez l'accès au répertoire de sauvegarde (propriétaire root, chmod 750).
- Chiffrez hors-site si nécessaire (rsync vers une destination chiffrée ou utiliser Proxmox Backup Server avec chiffrement).
- Conservez au moins une copie hors-site si possible.

9) - Option : Proxmox Backup Server (PBS)

- Pour une solution robuste : considérez Proxmox Backup Server (PBS). PBS offre déduplication, chiffrement, gestion fine des rétentions et intégration native avec Proxmox VE.

10) Résumé rapide
- Préparer stockage, choisir `snapshot` si possible, automatiser via GUI/cron, tester la restauration, surveiller l'espace et les logs.

---

<div align="center">
  <a href="https://github.com/0xCyberLiTech" target="_blank" rel="noopener">
    <img src="https://skillicons.dev/icons?i=linux,debian,bash,docker,nginx,git,vim,python,markdown" alt="Skills" width="440">
  </a>
</div>

<div align="center">
  <b>🔒 Un guide proposé par <a href="https://0xcyberlitech.com/">0xCyberLiTech</a> • Pour des tutoriels accessibles à tous. 🔒</b>
</div>

