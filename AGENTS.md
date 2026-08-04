# AGENTS.md — Équipe Guyane (ai-team-guyane)

> **Construit avec Kimi** (kimi-k2.7-code:cloud) et **GLM** (glm-5.2:cloud) via Ollama Cloud.

## Mission

Automatiser le posting d'offres d'emploi sur 4 sites de recrutement
(BJemploi, Job2mada, Asako, WabaJob) via agents Hermes avec Playwright,
en testant 4 modèles LLM en A/B pour identifier le meilleur par site.

## Équipe

| Rôle | Prénom | Profil Hermes | Modèle Ollama Cloud |
|---|---|---|---|
| Recon Agent | **Léopold** | `recon-bot` | `glm-5.2:cloud` |
| Poster BJemploi | **Manon** | `bjemploi-poster` | `kimi-k2.7-code:cloud` |
| Poster Job2mada | **Manon** | `job2mada-poster` | `glm-5.2:cloud` |
| Poster Asako | **Manon** | `asako-poster` | `minimax-m3:cloud` |
| Poster WabaJob | **Manon** | `wabajob-poster` | `deepseek-v4-pro:cloud` |
| Review Agent | **Sylviane** | `review-bot` | `deepseek-v4-pro:cloud` |
| Dev Agent | **Ludovic** | `dev-bot` | `kimi-k2.7-code:cloud` |
| Lead Dev Agent | **Roseline** | `lead-dev-bot` | `deepseek-v4-pro:cloud` |

## Architecture

```
ai-team-guyane/                     ← Repo équipe (skills, infra, docs)
├── AGENTS.md                       # Ce fichier
├── .agent/tasks/recon/             # Suivi (pattern Merenza)
├── docs/                           # GITHUB_SETUP, TELEGRAM_SETUP, DELEGATION, ARCHITECTURE
├── infra/hermes/                   # config.yaml + profiles.yaml (8 profils)
├── skills/                        # Skills Hermes (recon, poster, review, dev, lead)
└── docker-compose.yml             # 9 services Hermes (théorique)

ai-hirekit/                         ← Repo projet (séparé)
├── .github/workflows/              # guyane-loop.yml + motherboard.yml + ci.yml
├── .github/ISSUE_TEMPLATE/         # posting-task.yml, recon-task.yml
├── sites/<site>/                   # RECON.md + cookies.json (gitignored)
├── job.md                          # Offre d'emploi + comptes recruteurs
├── README.md                       # Doc projet
└── setup-github.sh                 # Script setup GitHub (labels, Project V2)
```

## Règles de l'équipe

### Recon Agent — Léopold (recon-bot, glm-5.2:cloud)
- Navigue sur chaque site avec Playwright (browser_snapshot)
- Documente les champs de registration, login, posting
- Détecte captchas, multi-steps, Google Sign-in
- Commit `sites/<site>/RECON.md`
- Commente sur l'issue GitHub avec le résumé

### Poster Agents — Manon (4 profils, 1 par site, 4 modèles A/B)
- Charge les cookies de session pré-authentifiée
- Navigue vers la page de posting
- Remplit le formulaire depuis `job.md`
- Soumet et vérifie la visibilité de l'offre
- Ouvre une PR avec preuves (URL, screenshot, logs)

### Review Agent — Sylviane (review-bot, deepseek-v4-pro:cloud)
- Visite l'URL de l'offre publiée
- Vérifie : titre, description, champs, visibilité
- approve (merge PR) ou request-changes (corriger)
- Commente sur la PR avec le résultat

## Workflow

1. Opérateur crée les comptes recruteurs manuellement (captchas, validation admin)
2. Opérateur exporte les cookies de session → `sites/<site>/cookies.json`
3. Léopold (Recon) documente les formulaires → issue GitHub
4. Manon (Poster) poste l'offre → ouvre PR avec preuves
5. Sylviane (Review) vérifie l'offre en ligne → approve/request-changes
6. Si approve → merge PR + close issue

## Variables d'environnement

```bash
# Ollama Cloud (ne jamais committer les vraies valeurs)
OLLAMA_CLOUD_API_KEY=<dans ~/.hermes/.env, chmod 600>
OLLAMA_CLOUD_BASE_URL=https://ollama.com/v1

# Telegram
TELEGRAM_BOT_TOKEN=<dans GitHub Secrets>
TELEGRAM_CHAT_ID=<dans GitHub Secrets>

# GitHub App (planned)
APP_ID=<dans GitHub Secrets>
APP_PRIVATE_KEY=<dans GitHub Secrets>
```

## Commandes Telegram

| Commande | Action |
|---|---|
| `/guyane-recon <msg>` | Parler à Léopold (Recon) |
| `/guyane-poster <msg>` | Parler à Manon (Poster) |
| `/guyane-review <msg>` | Parler à Sylviane (Review) |
| `/teams` | Statut de toutes les équipes |
| `/motherboard` | Lien vers le Kanban unifié |

## Infra (Hermes sur VM 102)

```
Precision (100.111.21.3) → Proxmox
└── VM 102 (192.168.1.76, user: hermes)
    ├── /home/hermes/.hermes/profiles/
    │   ├── guyane-recon-bot/config.yaml       → glm-5.2:cloud
    │   ├── guyane-bjemploi-poster/config.yaml  → kimi-k2.7-code:cloud
    │   ├── guyane-job2mada-poster/config.yaml  → glm-5.2:cloud
    │   ├── guyane-asako-poster/config.yaml     → minimax-m3:cloud
    │   ├── guyane-wabajob-poster/config.yaml   → deepseek-v4-pro:cloud
    │   ├── guyane-review-bot/config.yaml       → deepseek-v4-pro:cloud
    │   ├── guyane-dev-bot/config.yaml          → kimi-k2.7-code:cloud
    │   └── guyane-lead-dev-bot/config.yaml      → deepseek-v4-pro:cloud
    ├── /home/hermes/projects/ai-team-guyane/  → repo équipe (skills, infra, docs)
    └── /home/hermes/projects/ai-hirekit/       → repo projet (sites, job.md)
```

**Note** : Les profils Guyane sont préfixés `guyane-` pour éviter les conflits
avec les profils Haiti (`po-bot`, `dev-bot`, `lead-dev-bot`) sur la même VM 102.
```

**Bug Hermes connu** : les profils corrompent leur config.yaml à chaque session
en ajoutant `provider: custom:ollama_cloud` qui n'existe pas. Workaround : réécrire
le config.yaml propre avant chaque appel.

## Contraintes dures

- ❌ Pas de Claude/GPT : utiliser glm-5.2, kimi-k2.7-code, minimax-m3, deepseek-v4-pro via Ollama Cloud.
- ❌ Pas d'Hydre : tout sur Ollama Cloud.
- ❌ Pas de cookies commités : `sites/*/cookies.json` dans `.gitignore`.
- ❌ Pas de credentials dans les logs Hermes ni les commentaires GitHub.
- ❌ Pas de secrets dans AGENTS.md — utiliser `.env` (chmod 600) et GitHub Secrets.
- ✅ Un profil Hermes par site (isolation stricte).
- ✅ A/B testing : 4 modèles testés en parallèle sur 4 sites.
- ✅ PR avec preuves (URL, screenshot, logs) avant review.
- ✅ Cookies pré-authentifiés (contournent captchas + validation admin).