# Délégation de développements — Équipe Guyane 🇬🇫 (ai-team-guyane)

> **Mis à jour** le 2026-08-04 — l'équipe Guyane n'est pas limitée au posting
> d'offres d'emploi. Elle peut aussi déléguer du développement sur le projet
> `ai-hirekit` (ou tout autre projet assigné).

## Vue d'ensemble

L'équipe Guyane n'est pas limitée au posting d'offres d'emploi sur ai-hirekit.
d'emploi. Elle peut aussi **recevoir et implémenter des tâches de
développement logiciel** déléguées par l'opérateur (Yohann).

## 3 canaux de communication

### 1. Telegram (recommandé — le plus rapide)

Tu envoies un message au bot Telegram Hermes, il crée une issue GitHub,
l'assigne au bon agent, et le travail commence.

```
Yohann (Telegram) → Hermes Gateway → GitHub Issue → Agent → PR → Review
```

### 2. GitHub Issues (structuré)

Tu crées une issue avec le label `dev-task`, les agents la prennent en charge.

### 3. CLI Hermes (direct, pour tests)

```bash
hermes chat \
  --profile dev-bot \
  --query "Implémente [feature] dans [fichier] avec TDD" \
  --toolsets coding
```

---

## Canaux de communication détaillés

### Canal 1 — Telegram

#### Setup (one-time)

```bash
# Sur le VPS Contabo
hermes gateway setup
# → Choisir Telegram
# → Entrer le token du bot (créé via @BotFather sur Telegram)
```

#### Usage

Tu envoies un message au bot Telegram :

```
/dev Implémente une fonction de validation d'email dans lib/validators.ts
avec tests Vitest. Suis le TDD : tests d'abord, code ensuite.
```

Le bot Hermes :
1. Crée une issue GitHub avec le label `dev-task`
2. L'assigne au `dev-bot`
3. Le `dev-bot` implémente avec TDD
4. Ouvre une PR
5. Le `review-bot` vérifie
6. Tu reçois une notification Telegram avec le résultat

### Canal 2 — GitHub Issues

#### Créer une tâche de dev

```bash
gh issue create -R YoLoADR/ai-hirekit \
  --title "[Dev] Implémenter validation d'email" \
  --label "dev-task" \
  --assignee "ai-hirekit-poster[bot]" \
  --body "## Tâche de développement

### Objectif
Implémenter une fonction de validation d'email dans lib/validators.ts

### Critères d'acceptation
- [ ] Fonction isEmail(value: string): boolean
- [ ] Tests Vitest couvrant : email valide, email invalide, string vide, null
- [ ] Typecheck passe (tsc --noEmit)
- [ ] Lint passe (eslint)

### Contexte
- Projet: TypeScript + Vitest
- Fichier cible: lib/validators.ts
- Tests: tests/validators.test.ts
"
```

Le `dev-bot` (cron every 30m) lit l'issue et l'implémente.

### Canal 3 — CLI Hermes (direct)

```bash
# SSH sur le VPS Contabo
ssh user@vps-contabo

# Lancer une session chat directe avec le dev-bot
hermes chat \
  --profile dev-bot \
  --query "Implémente une fonction isEmail dans lib/validators.ts avec tests Vitest. TDD strict." \
  --toolsets coding

# Ou en mode one-shot (non interactif)
hermes chat \
  --profile dev-bot \
  --query "Corrige le bug dans src/api/tasks/route.ts ligne 42 : la réponse 404 devrait être 400 quand le body est invalide." \
  --toolsets coding \
  --max-turns 10 \
  --quiet
```

---

## Rôles d'agents pour le développement

### dev-bot (développeur TDD)

| Aspect | Valeur |
|---|---|
| Modèle | kimi-k2.7-code (spécialisé code) |
| Température | 0.2 (précis) |
| Toolsets | `coding` (terminal, file, browser, search, todo, delegate, vision) |
| Skill | `dev-tdd` (TDD strict) |
| Cron | every 30m (lit issues `dev-task`) |
| Workspace | `~/workspaces/dev` |

### lead-dev-bot (review code)

| Aspect | Valeur |
|---|---|
| Modèle | deepseek-v4-pro (analytique) |
| Température | 0.1 (rigoureux) |
| Toolsets | `coding` + `delegation` |
| Skill | `lead-review` (checklist de review) |
| Cron | every 30m (lit PRs ouvertes) |
| Workspace | `~/workspaces/leaddev` |

### recon-bot (analyse/exploration)

| Aspect | Valeur |
|---|---|
| Modèle | glm-5.2 (polyvalent) |
| Température | 0.3 |
| Toolsets | `browser` + `file` + `web` |
| Skill | `recon-workflow` |
| Cron | every 1h |
| Workspace | `~/workspaces/recon` |

---

## Workflow de délégation

```
1. TOI (Telegram / GitHub / CLI)
   → Envoies une tâche de développement
   ↓
2. GATEWAY Hermes (Telegram ou cron GitHub)
   → Crée/relit l'issue GitHub label "dev-task"
   → Assigne au dev-bot
   → Move Kanban → "In Progress"
   ↓
3. DEV-BOT (kimi-k2.7-code, toolset coding)
   → Lit l'issue + critères d'acceptation
   → Crée une branche : git checkout -b feature/<issue>-<slug>
   → TDD : RED (tests d'abord) → GREEN (implémentation) → REFACTOR
   → Lance : npm run typecheck && npm run test && npm run lint
   → Si échec : corrige, relance
   → git commit + push + gh pr create
   → Commente sur l'issue : "PR #XX ouverte"
   → Move Kanban → "In Review"
   ↓
4. LEAD-DEV-BOT (deepseek-v4-pro, toolset coding)
   → Lit la PR ouverte
   → gh pr diff → analyse le code
   → Checklist : conventions, tests, CI, secrets, spec, sécurité
   ├── APPROVE → gh pr merge --squash → close issue → "Done"
   └── REQUEST_CHANGES → commentaires file:line → "In Progress"
   ↓
5. NOTIFICATION Telegram
   → Tu reçois : "PR #XX merged ✅ / Issue #XX closed"
   → Ou : "PR #XX : changes requested (2 comments)"
```

---

## Exemples de tâches déléguables

### Exemple 1 — Feature nouvelle

**Telegram :**
```
/dev Ajoute un endpoint GET /api/jobs dans le projet qui liste toutes les
offres postées. Utilise Drizzle ORM. TDD strict. Tests dans tests/api/jobs.test.ts
```

### Exemple 2 — Bugfix

**GitHub Issue :**
```
[Bug] L'endpoint POST /api/tasks retourne 500 quand le body est vide
Steps to reproduce:
1. curl -X POST http://localhost:3000/api/tasks -H "Content-Type: application/json" -d '{}'
Expected: 400 Bad Request
Actual: 500 Internal Server Error
```

### Exemple 3 — Refactoring

**CLI :**
```bash
hermes chat \
  --profile dev-bot \
  --query "Refactorise lib/db/client.ts : extraire la configuration dans une fonction initDatabase() séparée. Garde tous les tests existants verts." \
  --toolsets coding
```

### Exemple 4 — Exploration + délégation

```bash
hermes chat \
  --profile dev-bot \
  --query "Analyse la structure du projet et délègue à un subagent la création des tests manquants pour les composants TodoList et TodoForm." \
  --toolsets coding,delegation
```

→ Le dev-bot utilise `delegate_task` pour spawn un subagent qui écrit les tests
en parallèle, pendant qu'il continue le reste.

---

## Délégation à des subagents (delegate_task)

Hermes permet à un agent de **spawner des subagents** pour paralléliser :

```python
delegate_task(
    goal="Implement Task 1: Create User model with email and password_hash fields",
    context="""
    TASK FROM PLAN:
    - Create: src/models/user.py
    - Add User class with email and password_hash fields

    FOLLOW TDD:
    1. Write failing test in tests/models/test_user.py
    2. Run: pytest tests/models/test_user.py -v (verify FAIL)
    3. Write minimal implementation
    4. Run: pytest tests/models/test_user.py -v (verify PASS)
    5. Commit: git add -A && git commit -m "feat: add User model"

    PROJECT CONTEXT:
    - TypeScript + Vitest project
    """,
    toolsets=['terminal', 'file']
)
```

### Profondeur de délégation
- `role="leaf"` : le subagent ne peut pas re-déléguer (default)
- `role="orchestrator"` : le subagent peut spinner ses propres subagents
- `max_spawn_depth=2` : max 2 niveaux de délégation

---

## Configuration Telegram (recommandé)

### 1. Créer le bot

1. Ouvrir Telegram → @BotFather
2. `/newbot` → nommer `ai_hirekit_bot`
3. Récupérer le token

### 2. Configurer sur le VPS

```bash
# SSH sur le VPS Contabo
hermes gateway setup
# → Telegram
# → Coller le token
# → Choisir ton user ID (pour que seul toi puisses commander le bot)
```

### 3. Variables d'environnement

```bash
# /opt/ai-hirekit/.env
TELEGRAM_BOT_TOKEN=your-bot-token
TELEGRAM_ALLOWED_USER_IDS=your-telegram-user-id
OLLAMA_CLOUD_API_KEY=your-key
GH_TOKEN_DEV=ghp_xxx
GH_TOKEN_REVIEW=ghp_xxx
```

### 4. Lancer le gateway

```bash
hermes gateway run
# → Le bot écoute les messages Telegram
# → Tu peux chatter avec lui directement
```

---

## Commands Telegram du bot

| Commande | Action |
|---|---|
| `/dev <description>` | Crée issue dev-task → dev-bot implémente |
| `/recon <url>` | Crée issue recon → recon-bot analyse le site |
| `/post <site>` | Crée issue posting → poster-bot poste l'offre |
| `/status` | Affiche le statut des issues/PRs en cours |
| `/review` | Force le review-bot à vérifier les PRs ouvertes |
| `/skills` | Liste les skills disponibles |
| `/cron list` | Liste les cron jobs actifs |
| `/help` | Aide |

---

## Cron jobs (automatisation)

```bash
# Dev-bot : lit les issues dev-task toutes les 30 min
hermes cron create "every 30m" \
  "Check GitHub issues labeled 'dev-task' in YoLoADR/ai-hirekit. For each, implement with TDD, run tests, open PR." \
  --skill dev-tdd \
  --name "Dev bot" \
  --deliver telegram

# Lead-dev-bot : lit les PRs ouvertes toutes les 30 min
hermes cron create "every 30m" \
  "Check open PRs in YoLoADR/ai-hirekit. Review with checklist. Approve or request changes." \
  --skill lead-review \
  --name "Lead review bot" \
  --deliver telegram

# Notification : informe Telegram des changements
hermes cron create "every 1h" \
  "Summarize GitHub activity in YoLoADR/ai-hirekit: new issues, PRs opened/merged, closed issues." \
  --name "Status report" \
  --deliver telegram
```

---

## Résumé : où trouver les agents et comment communiquer

| Canal | Comment | Quand |
|---|---|---|
| **Telegram** | Message au bot `@ai_hirekit_bot` | Quotidien, rapide |
| **GitHub Issues** | `gh issue create --label dev-task` | Tâches structurées |
| **CLI Hermes** | `hermes chat --profile dev-bot` | Tests, debug |
| **GitHub PRs** | Commentaire sur une PR | Review feedback |
| **Kanban Hermes** | `kanban_list` tool | Suivi multi-tâches |