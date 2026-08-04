# Todos — ai-hirekit

## Phase 0 — Setup projet
- [x] Créer `ai-hirekit/` avec structure de dossiers
- [x] Rédiger `job.md` (offre + champs comptes)
- [x] Rédiger `context.md` + `SESSION_NOTES.md` + `SESSION_LOG.md` + `PLAN.md`
- [x] Créer `.github/` (workflows, ISSUE_TEMPLATE, PR_TEMPLATE)
- [x] Créer `docs/GITHUB_SETUP.md` (repo, App, Project V2, secrets)
- [x] Créer skills loop engineering (recon, poster, review)
- [x] Créer `setup-github.sh` (automatisation setup)
- [x] Créer `.agent/tasks/recon/issues.md` (4 issues recon initiales)
- [x] Mettre à jour `profiles.yaml` (6 profils: 4 posters + recon + review)
- [x] Mettre à jour `docker-compose.yml` (6 conteneurs)
- [ ] Initialiser git + premier commit
- [ ] Exécuter `setup-github.sh` (créer repo + labels)
- [ ] Créer GitHub App "ai-hirekit-poster"
- [ ] Créer GitHub Project V2 + colonnes
- [ ] Créer les 4 issues recon initiales (#1-#4)

## Phase 1 — Comptes recruteurs (opérateur = Yohann)
- [ ] Créer compte BJemploi → https://www.bjemploi.com/espace-employeurs-inscription
- [ ] ⏳ Attendre validation admin BJemploi (1-48h)
- [ ] Créer compte Job2mada → https://www.job2mada.com/register (toggle Employeur)
- [ ] Créer compte Asako → https://www.asako.mg/inscription/recruteur
- [ ] Créer compte WabaJob → https://www.wabajob.com/register (4 étapes)
- [ ] ⏳ Cliquer lien confirmation email WabaJob (étape 4)
- [ ] Renseigner credentials dans `job.md` (sections [À RENSEIGNER])
- [ ] Exporter cookies JSON pour chaque site (Playwright snippet)
- [ ] Créer issues #5-#8 (comptes créés) avec label "comptes"

## Phase 2 — Infrastructure VPS Contabo
- [ ] Provisionner VPS Contabo dédié
- [ ] Installer Docker + Docker Compose
- [ ] Cloner repo `ai-hirekit`
- [ ] Créer `.env` avec OLLAMA_CLOUD_API_KEY + GH_TOKEN_*
- [ ] Copier cookies JSON sur le VPS (sites/*/cookies.json)
- [ ] Tester connectivité Ollama Cloud
- [ ] Lancer `docker-compose up -d` (6 conteneurs)
- [ ] Vérifier que Hermes démarre correctement

## Phase 3 — Recon-bot (automatique)
- [ ] recon-bot lit issues #1-#4
- [ ] recon-bot complète RECON.md pour chaque site
- [ ] recon-bot documente les champs du formulaire de posting (après login)
- [ ] recon-bot ajoute label "comptes" → transition Kanban

## Phase 4 — Poster-bot (A/B test, 16 issues)
- [ ] Créer 16 issues de posting (4 sites × 4 modèles)
- [ ] poster-bot (bjemploi) × kimi-k2.7-code → posting
- [ ] poster-bot (bjemploi) × glm-5.2 → posting
- [ ] poster-bot (bjemploi) × minimax-m3 → posting
- [ ] poster-bot (bjemploi) × deepseek-v4-pro → posting
- [ ] poster-bot (job2mada) × 4 modèles → posting
- [ ] poster-bot (asako) × 4 modèles → posting
- [ ] poster-bot (wabajob) × 4 modèles → posting
- [ ] Chaque posting ouvre une PR avec preuves

## Phase 5 — Review-bot (automatique)
- [ ] review-bot lit les PRs ouvertes
- [ ] review-bot visite l'URL de chaque offre publiée
- [ ] review-bot vérifie titre, description, champs, visibilité
- [ ] review-bot approve ou request-changes
- [ ] PRs approuvées → merge → close issue → label "success"

## Phase 6 — A/B test et optimisation
- [ ] Comparer les 16 postings (taux succès, qualité, robustesse)
- [ ] Identifier le meilleur modèle LLM par site
- [ ] Optimiser le system_prompt du gagnant
- [ ] Documenter dans `insights.md` + `results.md`