# Plan d'Architecture — ai-hirekit

## 0. Raisonnement ayant mené à cette architecture

### Contexte de départ
Yohann a une infrastructure existante avec 3 machines :
- **Contabo** (`ai-sales-vm`, 100.98.194.18) — VPS avec divers services
- **Precision** (`100.111.21.3`) — serveur Proxmox avec 2 conteneurs LXC
- **VM 102** (`ai-agents-vm`) — conteneur LXC sur Precision avec Hermes

Le projet ai-team utilise déjà cette infra : un bot Telegram sur Contabo
(`/opt/telegram-bot.py`) route les commandes vers VM 102 où 3 agents Hermes
(po-bot, dev-bot, lead-dev-bot) travaillent sur le repo `YoLoADR/ai-team`.

### Pourquoi ne pas créer une nouvelle infra
Créer un nouveau VPS pour ai-hirekit serait coûteux et inutile. L'infra
existante a la capacité d'accueillir des profils Hermes supplémentaires.
La contrainte clé est **l'isolation** : ai-hirekit ne doit pas interférer
avec ai-team.

### Solution choisie : profils séparés + routing par projet
- **Même VM 102** pour les agents (pas de nouvelle machine)
- **Profils Hermes séparés** : `hirekit-*` avec `cwd` pointant vers
  `/home/hermes/projects/ai-hirekit/` (différent de `/home/hermes/repo/`
  utilisé par ai-team)
- **Même bot Telegram** (`@loop_engineering_team_bot`) mais avec
  commandes `/hirekit` qui routent vers les profils ai-hirekit
- **Repos GitHub séparés** : `YoLoADR/ai-hirekit` ≠ `YoLoADR/ai-team`

### Décisions clés et leurs raisons
1. **Hermes > OpenHands** : browser toolset Playwright intégré (décisif
   pour le posting sur sites web)
2. **4 modèles en A/B** : pas de retour d'expérience, test en parallèle
   pour identifier le meilleur
3. **Cookies pré-auth** : contournent captchas + validation admin +
   vérification email
4. **GitHub Issues/PRs** : persistance de l'état entre sessions Hermes
   (stateless), audit trail, Kanban visuel
5. **Telegram** : interface mobile, gateway natif Hermes, notifications
   push
6. **Profils isolés sur VM 102** : zéro interférence avec ai-team
7. **Contabo = gateway uniquement** : le script `telegram-bot.py` fait
   juste du SSH routing, aucun agent ne tourne sur Contabo

### Modèles ayant construit cette équipe

Cette équipe et toute la documentation du projet ai-hirekit ont été
**construites avec Kimi (kimi-k2.7-code) et GLM (glm-5.2)** via Ollama Cloud,
lors d'une session unique le 2026-08-03 :
- **GLM-5.2** : analyse, recon des 4 sites, rédaction de la documentation,
  routing Telegram, configuration Hermes
- **Kimi-k2.7-code** : code, profils Hermes, docker-compose, scripts,
  structuration des skills

Les 2 autres modèles (minimax-m3, deepseek-v4-pro) sont assignés à des
rôles spécifiques (Asako, WabaJob) pour l'A/B test mais n'ont pas
participé à la construction de l'équipe.

---

## 1. Vue d'ensemble

Trois machines physiques/VMs协作 pour faire fonctionner une équipe d'agents
Hermes qui reçoivent des tâches via Telegram, implémentent du code, et
postent des offres d'emploi.

```
┌─────────────────────────────────────────────────────────────────────┐
│                         TOI (Telegram)                               │
│                   @loop_engineering_team_bot                         │
└────────────────────────────┬────────────────────────────────────────┘
                             │ Messages Telegram
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│  MACHINE 1 : CONTABO                                                 │
│  Hostname: ai-sales-vm                                              │
│  Tailscale: 100.98.194.18                                            │
│                                                                     │
│  Service: loop-engineering-bot.service (ACTIF)                      │
│  Script:  /opt/telegram-bot.py                                      │
│  Token:   8705472898:AAHBC3KCff188YRewHDYOLDGq3FyBja-lc8            │
│                                                                     │
│  RÔLE: Gateway Telegram                                             │
│  - Reçoit tes messages Telegram                                     │
│  - Route via SSH vers Precision → VM 102                            │
│  - NE FAIT AUCUN CODE — juste du routing                            │
│  - Aucun agent Hermes ne tourne ici (service user inactif)          │
│                                                                     │
│  ⚠️ Hermes est installé (pip) mais INACTIF sur cette machine        │
│     ~/.hermes/config.yaml existe mais le gateway user est arrêté     │
│     Ne pas confondre avec le service système loop-engineering-bot   │
└────────────────────────────┬────────────────────────────────────────┘
                             │ SSH root@100.111.21.3 → pct exec 102
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│  MACHINE 2 : PRECISION                                               │
│  Hostname: precision                                                │
│  Tailscale: 100.111.21.3                                             │
│                                                                     │
│  RÔLE: Hyperviseur Proxmox                                          │
│  - Aucun code, aucun agent ici                                      │
│  - Juste du routing SSH vers les conteneurs LXC                     │
│                                                                     │
│  Conteneurs LXC:                                                    │
│  ├── CT 101: openhands-vm (running) — autre projet                 │
│  └── CT 102: ai-agents-vm (running) — C'EST LÀ QUE SONT LES BOTS   │
└────────────────────────────┬────────────────────────────────────────┘
                             │ pct exec 102 -- su - hermes -c "..."
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│  MACHINE 3 : VM 102 (ai-agents-vm)                                   │
│  Accessible via: ssh root@100.111.21.3 → pct exec 102              │
│  User: hermes                                                       │
│                                                                     │
│  RÔLE: Exécution des agents Hermes                                  │
│                                                                     │
│  Structure:                                                         │
│  /home/hermes/                                                      │
│  ├── .hermes/profiles/                                              │
│  │   ├── po-bot/config.yaml       (minimax-m3:cloud)                │
│  │   ├── dev-bot/config.yaml      (kimi-k2.7-code:cloud)             │
│  │   └── lead-dev-bot/config.yaml (deepseek-v4-pro:cloud)            │
│  ├── repo/                         ← YoLoADR/ai-team (git)         │
│  │   ├── apps/ infra/ skills/ ...                                   │
│  │   └── .git → origin: YoLoADR/ai-team                              │
│  └── projects/ai-hirekit/          ← YoLoADR/ai-hirekit (git)       │
│      ├── job.md sites/ skills/ ...                                  │
│      └── .git → origin: YoLoADR/ai-hirekit                          │
│                                                                     │
│  BOTS ACTIFS (pour ai-team):                                        │
│  ├── po-bot       → cwd: /home/hermes/repo/  → repo ai-team         │
│  ├── dev-bot     → cwd: /home/hermes/repo/  → repo ai-team         │
│  └── lead-dev-bot→ cwd: /home/hermes/repo/  → repo ai-team         │
│                                                                     │
│  BOTS POUR ai-hirekit: PAS ENCORE DÉPLOYÉS                         │
│  (skills et profils existent dans le repo mais pas sur la VM)       │
└─────────────────────────────────────────────────────────────────────┘
```

## 2. Isolation entre projets

### Problème
Les 3 bots (po, dev, lead) sont configurés avec `cwd: /home/hermes/repo/`
qui pointe vers `YoLoADR/ai-team`. Si on veut que les mêmes bots travaillent
sur `YoLoADR/ai-hirekit`, il faut soit :

### Solution A — Profils séparés (recommandé)

Créer de nouveaux profils Hermes dédiés à ai-hirekit :

```
/home/hermes/.hermes/profiles/
├── po-bot/config.yaml              (cwd: /home/hermes/repo/ → ai-team)
├── dev-bot/config.yaml             (cwd: /home/hermes/repo/ → ai-team)
├── lead-dev-bot/config.yaml        (cwd: /home/hermes/repo/ → ai-team)
│
├── hirekit-recon/config.yaml       (cwd: /home/hermes/projects/ai-hirekit/ → ai-hirekit)
├── hirekit-poster/config.yaml      (cwd: /home/hermes/projects/ai-hirekit/ → ai-hirekit)
├── hirekit-review/config.yaml      (cwd: /home/hermes/projects/ai-hirekit/ → ai-hirekit)
├── hirekit-dev/config.yaml         (cwd: /home/hermes/projects/ai-hirekit/ → ai-hirekit)
└── hirekit-leaddev/config.yaml     (cwd: /home/hermes/projects/ai-hirekit/ → ai-hirekit)
```

**Avantage** : zéro interférence avec ai-team. Chaque profil a son cwd.
**Inconvénient** : 5 profils supplémentaires à maintenir.

### Solution B — Routing par projet dans telegram-bot.py

Modifier `/opt/telegram-bot.py` (Contabo) pour détecter le projet :

```python
# Dans PROJECTS, ai-hirekit pointe déjà vers /home/hermes/projects/ai-hirekit
PROJECTS = {
    "ai-team": {"repo": "YoLoADR/ai-team", "path": "/home/hermes/repo"},
    "ai-hirekit": {
        "repo": "YoLoADR/ai-hirekit",
        "path": "/home/hermes/projects/ai-hirekit",
        "profiles": {
            "recon": "hirekit-recon",
            "poster": "hirekit-poster",
            "review": "hirekit-review",
            "dev": "hirekit-dev",
            "leaddev": "hirekit-leaddev",
        },
    },
}

# Commande: /hirekit dev <message>
# → utilise le profil hirekit-dev avec cwd=/home/hermes/projects/ai-hirekit/
```

### Recommandation
**Solution A + B combinées** : profils séparés sur VM 102 + routing par
projet dans telegram-bot.py.

---

## 3. Déploiement ai-hirekit sur VM 102

### Étape 1 — Créer les profils Hermes sur VM 102

```bash
# Depuis Contabo
ssh root@100.111.21.3
pct exec 102 -- bash

# Sur VM 102
su - hermes

# Créer les profils ai-hirekit
for bot in recon poster review dev leaddev; do
  mkdir -p ~/.hermes/profiles/hirekit-${bot}
  cat > ~/.hermes/profiles/hirekit-${bot}/config.yaml << EOF
approvals:
  mode: smart
model:
  api_key: f3bf130d76a44536991f4b6bd47e650e.BszCTL2hUgQT2sNnQb6iUnJC
  base_url: https://ollama.com/v1
  default: <modèle selon le bot>
  provider: custom
terminal:
  backend: local
  container_persistent: true
  cwd: /home/hermes/projects/ai-hirekit
  home_mode: profile
onboarding:
  seen:
    tool_progress_prompt: true
EOF
done
```

### Modèles par profil ai-hirekit

| Profil | Modèle | Rôle |
|---|---|---|
| hirekit-recon | glm-5.2:cloud | Analyse des sites |
| hirekit-poster | kimi-k2.7-code:cloud | Posting d'offres |
| hirekit-review | deepseek-v4-pro:cloud | Vérification postings |
| hirekit-dev | kimi-k2.7-code:cloud | Dev logiciel (TDD) |
| hirekit-leaddev | deepseek-v4-pro:cloud | Review de code |

### Étape 2 — Déployer les skills ai-hirekit

```bash
# Sur VM 102 (user hermes)
cp -r /home/hermes/projects/ai-hirekit/skills/* ~/.hermes/skills/
```

### Étape 3 — Configurer le cron pour ai-hirekit

```bash
# Sur VM 102 (user hermes)
hermes cron create "every 30m" \
  "Check GitHub issues labeled 'recon' in YoLoADR/ai-hirekit..." \
  --profile hirekit-recon \
  --skill recon-workflow \
  --name "ai-hirekit Recon"

hermes cron create "every 30m" \
  "Check GitHub issues labeled 'dev-task' in YoLoADR/ai-hirekit..." \
  --profile hirekit-dev \
  --skill dev-tdd \
  --name "ai-hirekit Dev"
```

### Étape 4 — Mettre à jour telegram-bot.py

Ajouter les commandes `/hirekit` dans le script Contabo :

```python
# Dans /opt/telegram-bot.py (Contabo)

HIREKIT_BOTS = {
    "recon": ("hirekit-recon", "glm-5.2:cloud", "Recon — analyse sites"),
    "poster": ("hirekit-poster", "kimi-k2.7-code:cloud", "Poster — posting d'offres"),
    "review": ("hirekit-review", "deepseek-v4-pro:cloud", "Review — vérification postings"),
    "dev": ("hirekit-dev", "kimi-k2.7-code:cloud", "Dev — TDD, code, PRs"),
    "lead": ("hirekit-leaddev", "deepseek-v4-pro:cloud", "Lead Dev — review code"),
}

# Commande: /hirekit dev <message>
# → utilise hirekit-dev sur /home/hermes/projects/ai-hirekit/
```

---

## 4. Tableau récapitulatif — Qui est où

### Services systemd

| Machine | Service | Type | Statut | Rôle |
|---|---|---|---|---|
| Contabo | `loop-engineering-bot.service` | system | ✅ actif | Gateway Telegram |
| Contabo | `hermes-gateway.service` | user | ❌ inactif | (inutilisé) |
| Contabo | `hermes-gateway-telegram-gateway.service` | system | ❌ failed | (conflit, à supprimer) |
| Precision | (aucun) | — | — | Hyperviseur |
| VM 102 | (aucun service systemd) | — | — | Hermes lancé via SSH |

### Processus actifs

| Machine | Process | PID | Ce qu'il fait |
|---|---|---|---|
| Contabo | `/usr/bin/python3 /opt/telegram-bot.py` | 1688701 | Écoute Telegram, route vers VM 102 |
| VM 102 | (lancé à la demande via SSH) | — | hermes chat --profile X |

### Fichiers de configuration

| Fichier | Machine | Rôle |
|---|---|---|
| `/opt/telegram-bot.py` | Contabo | Script de routing Telegram → VM 102 |
| `/etc/systemd/system/loop-engineering-bot.service` | Contabo | Service du bot Telegram |
| `~/.hermes/config.yaml` | Contabo | Config Hermes (INACTIVE) |
| `~/.hermes/profiles.yaml` | Contabo | Profils théoriques (INACTIFS) |
| `~/.hermes/profiles/{po,dev,lead-dev}-bot/config.yaml` | VM 102 | Profils actifs (ai-team) |
| `/home/hermes/repo/` | VM 102 | Clone de `YoLoADR/ai-team` |
| `/home/hermes/projects/ai-hirekit/` | VM 102 | Clone de `YoLoADR/ai-hirekit` |

### GitHub

| Repo | URL | Utilisé par |
|---|---|---|
| ai-team | `https://github.com/YoLoADR/ai-team` | po-bot, dev-bot, lead-dev-bot (VM 102) |
| ai-hirekit | `https://github.com/YoLoADR/ai-hirekit` | (pas encore déployé sur VM 102) |

### Secrets/credentials

| Secret | Où | Valeur |
|---|---|---|
| Ollama Cloud API Key | VM 102 (dans chaque config.yaml) | `f3bf...UnJC` |
| Telegram Bot Token | Contabo (dans service systemd) | `8705472898:AAH...` |
| GitHub Token | Contabo (dans service systemd) | `gho_s88v...` |
| Telegram User ID | (dans le code telegram-bot.py) | `7211240214` |

---

## 5. Nettoyage à faire

### Sur Contabo

```bash
# Supprimer le service Hermes user (inactif, source de confusion)
systemctl --user stop hermes-gateway
systemctl --user disable hermes-gateway

# Supprimer le service system failed
sudo systemctl stop hermes-gateway-telegram-gateway
sudo systemctl disable hermes-gateway-telegram-gateway
sudo rm /etc/systemd/system/hermes-gateway-telegram-gateway.service
sudo systemctl daemon-reload
```

### Sur VM 102

```bash
# Déployer les skills ai-hirekit
cp -r /home/hermes/projects/ai-hirekit/skills/* /home/hermes/.hermes/skills/

# Créer les profils hirekit-* (voir étape 1 ci-dessus)
```

---

## 6. Diagramme de flux complet

### Flux 1 — Déléguer du dev (ai-team)

```
Toi: /dev Implémente X
  ↓ Telegram
Contabo: telegram-bot.py
  ↓ SSH root@100.111.21.3 → pct exec 102
Precision → VM 102
  ↓ su - hermes -c "dev-bot chat -q 'message'"
VM 102: dev-bot (kimi-k2.7-code)
  ↓ cwd: /home/hermes/repo/
  ↓ git clone YoLoADR/ai-team
  ↓ Implémente (TDD) → git push → gh pr create
GitHub: YoLoADR/ai-team — PR ouverte
  ↓ (cron every 30m)
VM 102: lead-dev-bot (deepseek-v4-pro)
  ↓ gh pr review → approve → merge
GitHub: YoLoADR/ai-team — PR mergée
  ↓ Telegram
Toi: notification "PR #X merged ✅"
```

### Flux 2 — Déléguer du dev (ai-hirekit) — À DÉPLOYER

```
Toi: /hirekit dev Implémente X
  ↓ Telegram
Contabo: telegram-bot.py (mis à jour)
  ↓ SSH root@100.111.21.3 → pct exec 102
Precision → VM 102
  ↓ su - hermes -c "hermes --profile hirekit-dev chat -q 'message'"
VM 102: hirekit-dev (kimi-k2.7-code)
  ↓ cwd: /home/hermes/projects/ai-hirekit/
  ↓ git clone YoLoADR/ai-hirekit
  ↓ Implémente (TDD) → git push → gh pr create
GitHub: YoLoADR/ai-hirekit — PR ouverte
  ↓ (cron every 30m)
VM 102: hirekit-leaddev (deepseek-v4-pro)
  ↓ gh pr review → approve → merge
GitHub: YoLoADR/ai-hirekit — PR mergée
  ↓ Telegram
Toi: notification "PR #X merged ✅"
```

### Flux 3 — Posting d'offres d'emploi (ai-hirekit) — À DÉPLOYER

```
Toi: /hirekit post bjemploi
  ↓ Telegram
Contabo: telegram-bot.py (mis à jour)
  ↓ SSH
VM 102: hirekit-poster (kimi-k2.7-code)
  ↓ cwd: /home/hermes/projects/ai-hirekit/
  ↓ Charge cookies sites/bjemploi/cookies.json
  ↓ browser_navigate → browser_snapshot → browser_type → browser_click
BJemploi.com: offre publiée
  ↓
VM 102: hirekit-review (deepseek-v4-pro)
  ↓ Vérifie l'offre visible
  ↓ gh pr create avec preuves
GitHub: YoLoADR/ai-hirekit — PR + issue fermée
  ↓ Telegram
Toi: "Offre publiée sur BJemploi ✅"
```