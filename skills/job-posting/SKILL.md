# Job Posting Skill

## Overview
Skill pour poster des offres d'emploi sur des sites de recrutement via
browser automation Hermes (Playwright). Utilise des sessions pré-authentifiées
(cookies JSON) pour contourner captchas et validations manuelles.

## When to Use
- Poster une offre d'emploi sur un site de recrutement
- Multi-posting (même offre sur plusieurs sites)
- "Don't use for": création de comptes (manuel), recherche de CV, navigation
  générale

## Prérequis
- Cookies de session pré-authentifiés dans `sites/<site>/cookies.json`
- Offre d'emploi formatée dans `job.md`
- Hermes Agent avec browser toolset activé

## Procédure générale

### 1. Charger les cookies
```python
import json
from playwright.sync_api import sync_playwright

def load_cookies(page, site):
    with open(f"sites/{site}/cookies.json") as f:
        cookies = json.load(f)
    page.context().add_cookies(cookies)
```

### 2. Naviguer sur le site
- `browser_navigate("https://<site>/publication-url")`
- `browser_snapshot()` → vérifier qu'on est bien sur le formulaire (pas
  redirigé vers login)

### 3. Remplir le formulaire
- `browser_snapshot()` → identifier les champs par leur ref
- `browser_type(ref="@eXX", text="valeur")` pour chaque champ
- Respecter l'ordre des champs du RECON.md du site

### 4. Soumettre
- `browser_click(ref="@eXX")` sur le bouton de soumission
- `browser_snapshot()` → vérifier le message de confirmation

### 5. Vérifier
- Naviguer vers la page des offres publiées
- Chercher le titre de l'offre
- Confirmer la visibilité publique

## Par site

Voir les fichiers détaillés :
- `bjemploi.md` — BJemploi.com (PHP classique, Bénin)
- `job2mada.md` — Job2mada.com (SPA React, Madagascar)
- `asako.md` — Asako.mg (SPA React + IA, Madagascar)
- `wabajob.md` — WabaJob.com (SPA React 4 étapes, Bénin)

## Common Pitfalls
1. **Cookies expirés** → re-login manuel requis, re-exporter les cookies
2. **SPA hydration lente** → attendre 2-3s après navigate avant snapshot
3. **Champs dynamiques** → le snapshot peut changer entre两次 lectures
4. **reCAPTCHA invisible** → si détecté, session pré-auth est la seule voie
5. **Rate limiting** → attendre 30-60s entre deux postings

## Verification Checklist
- [ ] Cookies chargés et valides (pas de redirection login)
- [ ] Tous les champs obligatoires remplis
- [ ] Bouton de soumission cliqué
- [ ] Message de confirmation affiché
- [ ] Offre visible sur la page publique des annonces

## One-Shot Recipe
```
Prompt: "Poste l'offre d'emploi de job.md sur <site>. Utilise les cookies
de sites/<site>/cookies.json. Suis la procédure de skills/job-posting/<site>.md.
Vérifie que l'offre est visible après publication."
```