# GitHub Configuration — ai-hirekit

## 1. Repo GitHub

### Création

```bash
# Créer le repo sur GitHub
gh repo create YoLoADR/ai-hirekit --public --description "Template de posting automatique d'offres d'emploi sur sites de recrutement via agents Hermes (Playwright)"

# Push initial
git remote add origin git@github.com:YoLoADR/ai-hirekit.git
git push -u origin main
```

### Branche par défaut
`main` (pas de dev/develop — template simple)

### Protection de branche
```bash
gh api repos/YoLoADR/ai-hirekit/branches/main/protection \
  -H "Accept: application/vnd.github+json" \
  --method PUT \
  --field required_status_checks='{"strict":true,"contexts":[]}' \
  --field enforce_admins=false \
  --field required_pull_request_reviews=null \
  --field restrictions=null
```
→ Pas de protection stricte pour le pilote (les agents doivent pouvoir push).

---

## 2. GitHub Project V2 — Kanban

### Colonnes

```
Backlog → Recon Done → Comptes Created → Posting In Progress → Posting Review → Done
```

| Colonne | Signification |
|---|---|
| Backlog | Sites à traiter (1 issue par site) |
| Recon Done | Recon du formulaire de posting complétée |
| Comptes Created | Compte recruteur créé + cookies exportés |
| Posting In Progress | Agent Hermes en cours de posting |
| Posting Review | Offre postée, vérification en cours |
| Done | Offre visible publiquement, vérifiée |

### Création

```bash
# Créer le Project V2
gh project create --owner YoLoADR --title "ai-hirekit Kanban" --format TABLE

# Récupérer le project ID
PROJECT_ID=$(gh project list --owner YoLoADR --format json | jq -r '.projects[0].id')

# Ajouter les colonnes (fields)
gh project field-create $PROJECT_ID --name "Status" --data-type SINGLE_SELECT \
  --options "Backlog,Recon Done,Comptes Created,Posting In Progress,Posting Review,Done"
```

### Labels

```bash
gh label create "site-bjemploi" --description "BJemploi.com (Bénin)" --color "0B7541"
gh label create "site-job2mada" --description "Job2mada.com (Madagascar)" --color "0B7541"
gh label create "site-asako"    --description "Asako.mg (Madagascar)"      --color "0B7541"
gh label create "site-wabajob"  --description "WabaJob.com (Bénin)"        --color "0B7541"

gh label create "recon"          --description "Phase de recon"            --color "FBCA04"
gh label create "comptes"        --description "Création de comptes"       --color "1D76DB"
gh label create "posting"        --description "Phase de posting"          --color "A371F7"
gh label create "review"         --description "Vérification lead"         --color "D93F0B"
gh label create "blocked-admin"  --description "Bloqué par validation admin" --color "B60205"
gh label create "blocked-captcha" --description "Bloqué par captcha"       --color "B60205"
gh label create "success"        --description "Offre publiée avec succès" --color "0E8A16"
gh label create "failed"         --description "Échec de posting"          --color "B60205"
```

---

## 3. GitHub App — "ai-hirekit-poster"

### Création (manuel via GitHub UI)

1. Aller sur https://github.com/settings/apps/new
2. Remplir :
   - **GitHub App name**: `ai-hirekit-poster`
   - **Homepage URL**: `https://github.com/YoLoADR/ai-hirekit`
   - **Webhook URL**: `http://<vps-contabo-ip>:8080/webhook` (à configurer plus tard)
   - **Webhook secret**: générer un secret aléatoire
3. Permissions :
   - **Issues**: Read & Write
   - **Pull requests**: Read & Write
   - **Contents**: Read & Write
   - **Metadata**: Read-only (requis)
   - **Pull request reviews**: Read & Write (pour le Lead-Review)
4. Subscribe to events :
   - Issues
   - Pull requests
   - Pull request reviews
5. Create App → génère un App ID + Private Key (à télécharger)

### Tokens machine (1 App, 4 bot-identities)

| Bot | Rôle | Permissions | Utilisation |
|---|---|---|---|
| recon-bot | Recon | issues (write), metadata (read) | Crée les issues de recon |
| poster-bot | Posting | issues (write), contents (write) | Poste les offres via Hermes |
| review-bot | Lead Review | pull_requests (write), pull_request_reviews (write), issues (write) | Vérifie les postings |
| ops-bot | Ops | all (admin) | Opérateur (Yohann) manuel |

### Variables d'environnement (sur le VPS Contabo)

```bash
# .env sur le VPS
GITHUB_APP_ID=123456
GITHUB_APP_PRIVATE_KEY_PATH=/opt/ai-hirekit/github-app-private-key.pem
GITHUB_WEBHOOK_SECRET=your-webhook-secret
OLLAMA_CLOUD_API_KEY=your-ollama-key
```

### Installation sur le repo

1. Aller sur https://github.com/settings/apps/ai-hirekit-poster
2. Install App → Install on `YoLoADR/ai-hirekit`

---

## 4. Templates

### Issue Template — Posting

Voir `.github/ISSUE_TEMPLATE/posting-task.yml`

### Issue Template — Recon

Voir `.github/ISSUE_TEMPLATE/recon-task.yml`

### PR Template

Voir `.github/PULL_REQUEST_TEMPLATE.md`

---

## 5. Workflow — Loop Engineering adapté au Job Posting

```
1. OPS-BOT crée 4 issues (1 par site) avec label "site-xxx" + "recon"
   ↓
2. RECON-BOT (agent Hermes) fait la recon du formulaire de posting
   → commit sites/<site>/RECON.md
   → comment sur l'issue avec les champs identifiés
   → ajoute label "comptes"
   → move → "Recon Done"
   ↓
3. OPS-BOT (Yohann) crée manuellement le compte recruteur
   → ⚠️ Intervention manuelle (captchas éventuels, validation admin)
   → exporte les cookies JSON
   → commit sites/<site>/cookies.json (gitignored, stocké sur VPS)
   → commente sur l'issue avec le statut du compte
   → ajoute label "posting"
   → move → "Comptes Created"
   ↓
4. POSTER-BOT (agent Hermes, profil spécifique au site) lance le posting
   → browser_navigate → browser_snapshot → browser_type → browser_click
   → vérifie que l'offre est visible
   ├── SUCCÈS → ouvre PR avec preuve (screenshot/logs)
   │   → ajoute label "review"
   │   → move → "Posting Review"
   └── ÉCHEC → commente l'erreur sur l'issue
       → ajoute label "failed" ou "blocked-captcha"
       → move → "Posting In Progress" (retry)
   ↓
5. REVIEW-BOT (agent Hermes, modèle d'analyse) vérifie le posting
   → visiter l'URL publique de l'offre
   → vérifier : titre correct, description complète, offre visible
   → vérifier : pas de champ vide, pas d'erreur d'encodage
   ├── APPROVE → merge PR → close issue → label "success" → move "Done"
   └── REQUEST_CHANGES → comment formaté : "champ: problème → suggestion"
       → POSTER-BOT doit corriger et re-poster
```

### Transitions Kanban automatiques

| Événement | Transition |
|---|---|
| Issue créée | → Backlog |
| Label "recon" ajouté | → Recon (reste Backlog) |
| RECON.md commité | → Recon Done |
| Label "comptes" ajouté | → Comptes Created |
| Label "posting" ajouté | → Posting In Progress |
| PR ouverte | → Posting Review |
| PR approved + merged | → Done |
| PR changes requested | → Posting In Progress |

---

## 6. Cron jobs Hermes (sur VPS Contabo)

| Bot | Cron | Action |
|---|---|---|
| recon-bot | every 1h | Lit issues label "recon", fait la recon |
| poster-bot | every 30m | Lit issues label "posting", lance le posting |
| review-bot | every 30m | Lit PRs ouvertes, fait la review |

### Configuration cron Hermes

```bash
hermes cron create "every 1h" \
  "Check GitHub issues labeled 'recon' in YoLoADR/ai-hirekit. For each, navigate to the site URL, do a browser_snapshot of the posting form, identify all fields, commit sites/<site>/RECON.md, and comment on the issue with the fields found. Use profile: recon-bot" \
  --skill job-posting \
  --name "Recon bot" \
  --deliver github

hermes cron create "every 30m" \
  "Check GitHub issues labeled 'posting' in YoLoADR/ai-hirekit. For each, load cookies from sites/<site>/cookies.json, navigate to the posting form, fill it with the content from job.md, submit, verify the offer is visible. If success, open a PR with proof. If fail, comment the error on the issue." \
  --skill job-posting \
  --name "Poster bot" \
  --deliver github

hermes cron create "every 30m" \
  "Check open PRs in YoLoADR/ai-hirekit. For each, visit the posted offer URL, verify the title, description, and visibility. If correct, approve the PR. If not, request changes with specific comments." \
  --skill job-posting \
  --name "Review bot" \
  --deliver github
```

---

## 7. Secrets GitHub

### À configurer dans le repo

```bash
gh secret set OLLAMA_CLOUD_API_KEY --body "your-key"
gh secret set GITHUB_APP_PRIVATE_KEY < github-app-private-key.pem
gh secret set GITHUB_WEBHOOK_SECRET --body "your-webhook-secret"
```

### Secrets pour les agents Hermes (sur le VPS, pas dans GitHub)

```bash
# /opt/ai-hirekit/.env
OLLAMA_CLOUD_API_KEY=your-key
GITHUB_APP_ID=123456
GITHUB_APP_PRIVATE_KEY_PATH=/opt/ai-hirekit/keys/private-key.pem
GITHUB_WEBHOOK_SECRET=your-webhook-secret
GH_TOKEN_RECON=ghp_xxx   # token machine recon-bot
GH_TOKEN_POSTER=ghp_xxx  # token machine poster-bot
GH_TOKEN_REVIEW=ghp_xxx  # token machine review-bot
```

---

## 8. A/B Test via Issues

Chaque site aura **4 issues en parallèle** (1 par modèle LLM) pour comparer :

### Issue naming convention
```
[BJemploi] Post offer with kimi-k2.7-code
[BJemploi] Post offer with glm-5.2
[BJemploi] Post offer with minimax-m3
[BJemploi] Post offer with deepseek-v4-pro

[Job2mada] Post offer with kimi-k2.7-code
[Job2mada] Post offer with glm-5.2
...
```

### Labels A/B
```bash
gh label create "model-kimi"      --description "kimi-k2.7-code" --color "5319E7"
gh label create "model-glm"       --description "glm-5.2"        --color "5319E7"
gh label create "model-minimax"   --description "minimax-m3"     --color "5319E7"
gh label create "model-deepseek"  --description "deepseek-v4-pro" --color "5319E7"
```

### Tableau de résultats

| Site | Modèle | Issue | Statut | Durée | Notes |
|---|---|---|---|---|---|
| BJemploi | kimi-k2.7-code | #1 | ? | ? | |
| BJemploi | glm-5.2 | #2 | ? | ? | |
| BJemploi | minimax-m3 | #3 | ? | ? | |
| BJemploi | deepseek-v4-pro | #4 | ? | ? | |
| Job2mada | kimi-k2.7-code | #5 | ? | ? | |
| ... | ... | ... | ... | ... | |

→ Documenté dans `.agent/tasks/recon/results.md` (à créer après tests)