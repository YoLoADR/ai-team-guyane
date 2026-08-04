# Plan — ai-hirekit

## 1. Vue d'ensemble

Poster automatiquement une offre d'emploi sur 4 sites de recrutement
(BJemploi.com, job2mada.com, asako.mg, wabajob.com) via agents Hermes
avec browser automation Playwright, en testant 4 modèles LLM différents
pour identifier la configuration la plus efficace.

Le workflow est orchestré via **GitHub Issues + PRs + Project V2 Kanban** :
les 3 rôles d'agents (recon-bot, poster-bot, review-bot) collaborent via
GitHub, comme dans le pattern loop engineering de ai-team.

## 2. Architecture

```
/Users/YoLoADR/Factory/ai-hirekit/
│
├── .agent/tasks/recon/               # Recon + plan + suivi
│   ├── context.md
│   ├── PLAN.md (ce fichier)
│   ├── SESSION_NOTES.md
│   ├── SESSION_LOG.md
│   ├── insights.md
│   ├── issues.md                     # Issues GitHub initiales
│   └── todos.md
│
├── .github/                          # GitHub configuration
│   ├── workflows/
│   │   ├── ci.yml                    # CI: validate files, no cookies committed
│   │   └── kanban.yml                # Auto-move issues on PR events
│   ├── ISSUE_TEMPLATE/
│   │   ├── posting-task.yml          # Template issue de posting
│   │   └── recon-task.yml            # Template issue de recon
│   └── PULL_REQUEST_TEMPLATE.md      # Template PR (preuves de posting)
│
├── docs/
│   └── GITHUB_SETUP.md               # Setup repo, App, Project V2, secrets
│
├── sites/                            # Recon par site + cookies
│   ├── bjemploi/
│   │   ├── RECON.md
│   │   └── cookies.json (gitignored)
│   ├── job2mada/
│   │   ├── RECON.md
│   │   └── cookies.json (gitignored)
│   ├── asako/
│   │   ├── RECON.md
│   │   └── cookies.json (gitignored)
│   └── wabajob/
│       ├── RECON.md
│       └── cookies.json (gitignored)
│
├── infra/hermes/                     # Configs Hermes
│   ├── config.yaml
│   └── profiles/
│       ├── bjemploi-poster.yaml
│       ├── job2mada-poster.yaml
│       ├── asako-poster.yaml
│       └── wabajob-poster.yaml
│
├── skills/job-posting/               # Skill Hermes custom
│   ├── SKILL.md
│   ├── bjemploi.md
│   ├── job2mada.md
│   ├── asako.md
│   └── wabajob.md
│
├── job.md                            # Offre + comptes recruteurs
├── .gitignore
└── README.md

VPS Contabo "hermes-job-poster" (dédié)
├── Docker (shm-size=1g pour Playwright)
├── Hermes Agent
│   ├── Profile: bjemploi-poster (kimi-k2.7-code)
│   ├── Profile: job2mada-poster (glm-5.2)
│   ├── Profile: asako-poster (minimax-m3)
│   └── Profile: wabajob-poster (deepseek-v4-pro)
├── Cookies par site (volumes montés)
└── Ollama Cloud (API key via env var)
```

## 3. Hermes — Configuration globale

```yaml
# infra/hermes/config.yaml
providers:
  ollama_cloud:
    base_url: https://ollama.com/api/v1
    api_key: ${OLLAMA_CLOUD_API_KEY}
    api_type: openai

settings:
  default_provider: ollama_cloud
  confirm_tool_execution: false
  auto_approve: true
  memory_enabled: false
  loop_engineering: false

toolsets:
  - browser
  - shell
  - file

logging:
  level: info
  file: ~/.hermes/logs/job-posting.log
```

## 4. Hermes — Profils (4 agents A/B test)

### Profil BJemploi (kimi-k2.7-code)

```yaml
# infra/hermes/profiles/bjemploi-poster.yaml
name: "BJemploi Poster"
model: kimi-k2.7-code
provider: ollama_cloud
temperature: 0.1
max_tokens: 8192
system_prompt: |
  Tu es un agent qui poste des offres d'emploi sur BJemploi.com.
  Tu utilises les browser tools (navigate, snapshot, click, type).
  Tu charges les cookies de session pré-authentifiés.
  Tu remplis le formulaire de publication avec le contenu fourni.
  Tu vérifies que l'offre est visible après soumission.
  Réponds en français. Sois précis et méthodique.
workspace: ~/workspaces/bjemploi
allowed_tools:
  - browser_navigate
  - browser_snapshot
  - browser_click
  - browser_type
  - browser_scroll
  - browser_vision
  - file_read
```

### Profil Job2mada (glm-5.2)

```yaml
# infra/hermes/profiles/job2mada-poster.yaml
name: "Job2mada Poster"
model: glm-5.2
provider: ollama_cloud
temperature: 0.2
max_tokens: 8192
system_prompt: |
  Tu es un agent qui poste des offres d'emploi sur Job2mada.com.
  Tu utilises les browser tools (navigate, snapshot, click, type).
  Tu charges les cookies de session pré-authentifiés.
  Tu remplis le formulaire de publication avec le contenu fourni.
  Tu vérifies que l'offre est visible après soumission.
  Réponds en français. Sois précis et méthodique.
workspace: ~/workspaces/job2mada
allowed_tools:
  - browser_navigate
  - browser_snapshot
  - browser_click
  - browser_type
  - browser_scroll
  - browser_vision
  - file_read
```

### Profil Asako (minimax-m3)

```yaml
# infra/hermes/profiles/asako-poster.yaml
name: "Asako Poster"
model: minimax-m3
provider: ollama_cloud
temperature: 0.1
max_tokens: 16384
system_prompt: |
  Tu es un agent qui poste des offres d'emploi sur Asako.mg.
  Tu utilises les browser tools (navigate, snapshot, click, type).
  Tu charges les cookies de session pré-authentifiés.
  Asako propose une IA de génération d'offre — tu peux l'utiliser ou
  remplir le formulaire manuellement.
  Tu vérifies que l'offre est visible après soumission.
  Réponds en français. Sois précis et méthodique.
workspace: ~/workspaces/asako
allowed_tools:
  - browser_navigate
  - browser_snapshot
  - browser_click
  - browser_type
  - browser_scroll
  - browser_vision
  - file_read
```

### Profil WabaJob (deepseek-v4-pro)

```yaml
# infra/hermes/profiles/wabajob-poster.yaml
name: "WabaJob Poster"
model: deepseek-v4-pro
provider: ollama_cloud
temperature: 0.1
max_tokens: 8192
system_prompt: |
  Tu es un agent qui poste des offres d'emploi sur WabaJob.com.
  Tu utilises les browser tools (navigate, snapshot, click, type).
  Tu charges les cookies de session pré-authentifiés.
  WabaJob a un formulaire multi-étapes — sois attentif aux transitions.
  Tu vérifies que l'offre est visible après soumission.
  Réponds en français. Sois précis et méthodique.
workspace: ~/workspaces/wabajob
allowed_tools:
  - browser_navigate
  - browser_snapshot
  - browser_click
  - browser_type
  - browser_scroll
  - browser_vision
  - file_read
```

## 5. Skill job-posting (SKILL.md)

Voir `skills/job-posting/SKILL.md` — instructions détaillées par site avec
les selectors, les champs, l'ordre de remplissage, et la procédure de
vérification.

## 6. docker-compose.yml (VPS Contabo)

```yaml
services:
  hermes-bjemploi:
    image: nousresearch/hermes-agent:latest
    shm_size: 1g
    volumes:
      - ./profiles/bjemploi-poster.yaml:/root/.hermes/config.yaml
      - ./sites/bjemploi/cookies.json:/root/cookies.json
      - ./job.md:/root/job.md
      - ./skills:/root/.hermes/skills
    environment:
      - OLLAMA_CLOUD_API_KEY=${OLLAMA_CLOUD_API_KEY}
    network_mode: host

  hermes-job2mada:
    image: nousresearch/hermes-agent:latest
    shm_size: 1g
    volumes:
      - ./profiles/job2mada-poster.yaml:/root/.hermes/config.yaml
      - ./sites/job2mada/cookies.json:/root/cookies.json
      - ./job.md:/root/job.md
      - ./skills:/root/.hermes/skills
    environment:
      - OLLAMA_CLOUD_API_KEY=${OLLAMA_CLOUD_API_KEY}
    network_mode: host

  hermes-asako:
    image: nousresearch/hermes-agent:latest
    shm_size: 1g
    volumes:
      - ./profiles/asako-poster.yaml:/root/.hermes/config.yaml
      - ./sites/asako/cookies.json:/root/cookies.json
      - ./job.md:/root/job.md
      - ./skills:/root/.hermes/skills
    environment:
      - OLLAMA_CLOUD_API_KEY=${OLLAMA_CLOUD_API_KEY}
    network_mode: host

  hermes-wabajob:
    image: nousresearch/hermes-agent:latest
    shm_size: 1g
    volumes:
      - ./profiles/wabajob-poster.yaml:/root/.hermes/config.yaml
      - ./sites/wabajob/cookies.json:/root/cookies.json
      - ./job.md:/root/job.md
      - ./skills:/root/.hermes/skills
    environment:
      - OLLAMA_CLOUD_API_KEY=${OLLAMA_CLOUD_API_KEY}
    network_mode: host
```

## 7. Phases d'implémentation

### Phase 0 — Setup projet (terminé)
- [x] Créer `ai-hirekit/` avec structure de dossiers
- [x] Rédiger `job.md` (offre + champs comptes)
- [x] Rédiger `context.md` + `SESSION_NOTES.md` + `PLAN.md`
- [ ] Initialiser git + premier commit

### Phase 1 — Comptes recruteurs (opérateur)
- [ ] Créer compte BJemploi (⚠️ attendre validation admin 1-48h)
- [ ] Créer compte Job2mada
- [ ] Créer compte Asako
- [ ] Créer compte WabaJob (⚠️ étape 4 vérification email)
- [ ] Renseigner les credentials dans `job.md`
- [ ] Exporter cookies JSON pour chaque site

### Phase 2 — Infrastructure VPS Contabo
- [ ] Provisionner VPS Contabo dédié (nouveau, isolé)
- [ ] Installer Docker + Docker Compose
- [ ] Cloner le repo `ai-hirekit`
- [ ] Créer `.env` avec `OLLAMA_CLOUD_API_KEY`
- [ ] Tester connectivité Ollama Cloud

### Phase 3 — Hermes + Skills
- [ ] Déployer `docker-compose.yml` (4 conteneurs Hermes)
- [ ] Créer `skills/job-posting/SKILL.md`
- [ ] Créer les `sites/<site>/RECON.md` détaillés
- [ ] Tester que Hermes démarre correctement

### Phase 4 — Tests one-shot
- [ ] Lancer `hermes-bjemploi` avec prompt de posting
- [ ] Lancer `hermes-job2mada` avec prompt de posting
- [ ] Lancer `hermes-asako` avec prompt de posting
- [ ] Lancer `hermes-wabajob` avec prompt de posting
- [ ] Vérifier visibilité des offres sur chaque site

### Phase 5 — A/B test et optimisation
- [ ] Comparer les 4 agents (taux de succès, qualité, robustesse)
- [ ] Identifier le meilleur modèle LLM pour cette tâche
- [ ] Optimiser le system_prompt du gagnant
- [ ] Documenter les insights dans `insights.md`

## 8. Coûts

| Poste | Coût |
|---|---|
| Ollama Cloud (4 modèles) | $20/mois (plan existant) |
| VPS Contabo dédié | ~$6-15/mois (selon specs) |
| Domaines (sites) | $0 (sites existants) |
| **Total** | **~$26-35/mois** |

## 9. Risques et mitigations

| Risque | Mitigation |
|---|---|
| Captcha caché (reCAPTCHA invisible) | Sessions pré-auth + cookies |
| Cookies expirés | Skill avec étape de re-login automatique |
| Changement de structure des sites | browser_snapshot s'adapte |
| BJemploi validation admin lente | Créer le compte en avance (Phase 1) |
| WabaJob vérification email | Cliquer manuellement le lien |
| Rate limiting / IP bannie | Proxies par agent + delays aléatoires |
| Models Ollama Cloud indisponibles | Fallback sur d'autres modèles |
| Playwright crash (shm) | Docker shm-size=1g obligatoire |