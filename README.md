# TrueNAS SCALE — Déploiement complet

> Projet réalisé dans le cadre du **Bachelor Administrateur Systèmes & Réseaux**  
> La Plateforme_ — Marseille | Mars 2026

---

## Description

Déploiement d'un serveur NAS complet sous **TrueNAS SCALE** en environnement virtualisé (VMware Workstation), incluant :

- Configuration d'un **pool RAID-Z2** (équivalent RAID 6) avec ZFS
- Mise en place des services **SSH, FTP et SFTP** avec gestion des utilisateurs
- Déploiement de **Vaultwarden** (gestionnaire de mots de passe) via Docker
- Création d'une **VM Debian imbriquée** dans TrueNAS via KVM/QEMU

---

## Architecture

```
┌─────────────────────────────────────────────────────┐
│              VMware Workstation (Host)               │
│                                                     │
│  ┌──────────────────────┐  ┌──────────────────────┐ │
│  │   VM1 — TrueNAS      │  │   VM2 — Debian       │ │
│  │   192.168.198.130    │  │   Client DHCP        │ │
│  │                      │  │                      │ │
│  │  ┌────────────────┐  │  │  • Firefox (HTTPS)   │ │
│  │  │ Pool Stockage  │  │  │  • FileZilla (SFTP)  │ │
│  │  │ RAID-Z2 5×2Go  │  │  │  • Terminal (SSH)    │ │
│  │  └────────────────┘  │  │  • VNC (VM Debian)   │ │
│  │  ┌────────────────┐  │  └──────────────────────┘ │
│  │  │  Vaultwarden   │  │                           │
│  │  │  Docker :30032 │  │                           │
│  │  └────────────────┘  │                           │
│  │  ┌────────────────┐  │                           │
│  │  │  VM Debian     │  │                           │
│  │  │  KVM :5900     │  │                           │
│  │  └────────────────┘  │                           │
│  └──────────────────────┘                           │
└─────────────────────────────────────────────────────┘
```

---

## Spécifications techniques

### VM1 — Serveur TrueNAS SCALE

| Composant | Valeur |
|-----------|--------|
| CPU | 2 cœurs |
| RAM | 6 Go (4 Go + augmentation pour Docker) |
| Disques système | 2 × 16 Go |
| Disques RAID | 5 × 2 Go (RAID-Z2) |
| IP | 192.168.198.130 |

### VM2 — Client Debian

| Composant | Valeur |
|-----------|--------|
| CPU | 2 cœurs |
| RAM | 2 Go |
| Disque | 1 × 8 Go |

---

## Services déployés

| Service | Port | Protocole | Description |
|---------|------|-----------|-------------|
| Portail Web | 443 | HTTPS | Interface d'administration TrueNAS |
| SSH / SFTP | 2222 | SSH | Accès shell + transfert de fichiers |
| FTP | 21 | FTP | Partage de fichiers |
| Vaultwarden | 30032 | HTTP | Gestionnaire de mots de passe |
| VNC (VM) | 5900 | VNC | Accès à la VM Debian imbriquée |

---

## Structure du repo

```
truenas-scale/
├── README.md
├── docs/
│   └── documentation_truenas_v2.docx
├── config/
│   └── vaultwarden-config.json
└── screenshots/
    └── (captures d'écran du projet)
```

---

## Installation

### 1. Télécharger l'ISO TrueNAS SCALE
```
https://www.truenas.com/download-truenas-scale/
```

### 2. Créer le pool RAID-Z2
```
Storage → Create Pool → Nom: Stockage → RAIDZ2 → 5 disques de 2 Go
```

### 3. Activer SSH et FTP
```
System → Services → SSH (port 2222) + FTP → Start Automatically
```

### 4. Créer les utilisateurs SFTP
```
Credentials → Local Users → user1 + user2 (shell: bash)
```

### 5. Déployer Vaultwarden
```
Apps → Discover Apps → vaultwarden → Install
```

---

## Vaultwarden — Commandes utiles

```bash
# Voir les conteneurs Docker
docker ps

# Logs Vaultwarden
docker logs ix-vaultwarden-vaultwarden-1 2>&1 | tail -20

# Modifier la configuration
docker exec ix-vaultwarden-vaultwarden-1 cat /data/config.json

# Redémarrer le conteneur
docker restart ix-vaultwarden-vaultwarden-1
```

---

## Comptes de test

| Utilisateur | Rôle | Accès |
|-------------|------|-------|
| `truenas_admin` | Administrateur TrueNAS | Portail HTTPS + SSH |
| `user1` | Utilisateur SFTP | SSH :2222 + SFTP |
| `user2` | Utilisateur SFTP | SSH :2222 + SFTP |

> Les mots de passe ne sont pas stockés dans ce repo pour des raisons de sécurité.

---

## Problèmes rencontrés

| Problème | Solution |
|----------|----------|
| Clavier QWERTY sur VM Debian | `setxkbmap fr` dans le terminal |
| SFTP : publickey error | Activer Password Authentication + groupes SSH |
| SSH : account not available | Changer shell de `nologin` vers `bash` |
| Docker ne démarre pas | Augmenter RAM de 4 → 6 Go dans VMware |
| Vaultwarden : Insecure URL | Modifier `config.json` + `dom.securecontext.allowlist` dans Firefox |
| Pas d'espace pour VM Debian | `zfs destroy` ancienne VM + créer VMPool |
| Virtualisation non supportée | Activer VT-x dans VMware Workstation |

---

## Documentation

La documentation technique complète est disponible dans `docs/documentation_truenas.pdf`.

---

## 👩‍💻 Auteure

**Anna Mamelli**  
Bachelor Administrateur Systèmes & Réseaux — RNCP37680  
La Plateforme_ — Marseille
