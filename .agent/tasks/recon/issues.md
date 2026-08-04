# Issues initiales — ai-hirekit

## Issue #1 — Recon BJemploi.com

```bash
gh issue create -R YoLoADR/ai-hirekit \
  --title "[Recon] BJemploi.com — Cartographier le formulaire de posting" \
  --label "recon,site-bjemploi" \
  --body "## Recon BJemploi.com (Bénin)

### Objectif
Cartographier le formulaire de publication d'offre sur BJemploi.com.

### URL
https://www.bjemploi.com

### Checklist
- [x] Page d'accueil visitée (recon initiale faite)
- [x] Page d'inscription recruteur identifiée
- [x] Champs d'inscription documentés (11 champs)
- [x] Page de connexion identifiée
- [ ] Page de publication d'offre identifiée (login requis)
- [ ] Champs du formulaire de posting documentés
- [x] Captcha: aucun visible (iframe possible reCAPTCHA invisible)
- [ ] RECON.md complété dans sites/bjemploi/

### Notes
- Site PHP classique
- ⚠️ Validation admin requise après inscription (1-48h)
- Pas de Google Sign-in
- URL inscription: /espace-employeurs-inscription
- URL connexion: /espace-employeurs-connexion
- URL posting: /depot-annonce.html (login requis)
"
```

## Issue #2 — Recon Job2mada.com

```bash
gh issue create -R YoLoADR/ai-hirekit \
  --title "[Recon] Job2mada.com — Cartographier le formulaire de posting" \
  --label "recon,site-job2mada" \
  --body "## Recon Job2mada.com (Madagascar)

### Objectif
Cartographier le formulaire de publication d'offre sur Job2mada.com.

### URL
https://www.job2mada.com

### Checklist
- [x] Page d'accueil visitée
- [x] Page d'inscription identifiée (toggle Employeur)
- [x] Champs d'inscription documentés (12 champs mode Employeur)
- [x] Page de connexion identifiée
- [ ] Page de publication d'offre identifiée (login requis)
- [ ] Champs du formulaire de posting documentés
- [x] Captcha: aucun visible
- [ ] RECON.md complété dans sites/job2mada/

### Notes
- SPA React/Next.js
- Google Sign-in disponible
- 14 catégories d'entreprise
- URL inscription: /register
- URL connexion: /login
- URL posting: /jobs/new (login requis)
"
```

## Issue #3 — Recon Asako.mg

```bash
gh issue create -R YoLoADR/ai-hirekit \
  --title "[Recon] Asako.mg — Cartographier le formulaire de posting" \
  --label "recon,site-asako" \
  --body "## Recon Asako.mg (Madagascar)

### Objectif
Cartographier le formulaire de publication d'offre sur Asako.mg.

### URL
https://www.asako.mg

### Checklist
- [x] Page d'accueil visitée
- [x] Page d'inscription recruteur identifiée (intégrée à la landing page)
- [x] Champs d'inscription documentés (6 champs)
- [x] Page de connexion identifiée
- [ ] Page de publication d'offre identifiée (login requis)
- [ ] Champs du formulaire de posting documentés
- [x] Captcha: aucun visible
- [ ] RECON.md complété dans sites/asako/

### Notes
- SPA React/Next.js moderne
- ⚡ IA de génération d'offre intégrée (peut faciliter le posting)
- 500+ entreprises inscrites
- Indexé sur Google Jobs
- URL inscription: /inscription/recruteur
- URL connexion: /connexion
- URL posting: /recruteur/offres/nouvelle (login requis)
"
```

## Issue #4 — Recon WabaJob.com

```bash
gh issue create -R YoLoADR/ai-hirekit \
  --title "[Recon] WabaJob.com — Cartographier le formulaire de posting" \
  --label "recon,site-wabajob" \
  --body "## Recon WabaJob.com (Bénin)

### Objectif
Cartographier le formulaire de publication d'offre sur WabaJob.com.

### URL
https://www.wabajob.com

### Checklist
- [x] Page d'accueil visitée
- [x] Page d'inscription identifiée (4 étapes, type Recruteur)
- [x] Champs étape 1 documentés (4 champs: type, email, password, confirm)
- [ ] Champs étapes 2-4 documentés (nécessite completion étape 1)
- [x] Page de connexion identifiée
- [ ] Page de publication d'offre identifiée (login requis)
- [ ] Champs du formulaire de posting documentés
- [x] Captcha: aucun visible sur étape 1
- [ ] RECON.md complété dans sites/wabajob/

### Notes
- SPA React/Next.js
- Inscription multi-étapes (4 steps)
- ⚠️ Étape 4 = Vérification (probablement email)
- Google Sign-in disponible
- URL inscription: /register
- URL connexion: /login
- URL dashboard: /recruiter-dashboard (login requis)
"
```

## Issues #5-#8 — Comptes recruteurs (après recon)

Créées automatiquement par le recon-bot après validation des issues #1-#4,
avec le label "comptes".

## Issues #9-#24 — Posting (A/B test)

Pour chaque site (4) × chaque modèle (4) = 16 issues de posting :

```bash
# Exemple pour BJemploi × kimi
gh issue create -R YoLoADR/ai-hirekit \
  --title "[Posting] BJemploi × kimi-k2.7-code — Animateur·rice de quartier" \
  --label "posting,site-bjemploi,model-kimi" \
  --body "## Posting task

### Site
BJemploi.com

### Modèle
kimi-k2.7-code

### Cookies
sites/bjemploi/cookies.json

### URL posting
https://www.bjemploi.com/depot-annonce.html

### Offre
Voir job.md — 'Animateur·rice de quartier'

### Prérequis
- [ ] Compte recruteur créé
- [ ] Cookies exportés
- [ ] RECON.md complété

### Critères d'acceptation
- [ ] L'agent navigue sans redirection login
- [ ] Tous les champs obligatoires remplis
- [ ] Formulaire soumis
- [ ] Offre visible sur la page publique
"
```