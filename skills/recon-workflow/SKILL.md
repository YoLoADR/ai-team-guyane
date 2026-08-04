---
name: recon-workflow
description: Recon workflow — analyse les formulaires de posting sur les sites de recrutement
---
# Recon Workflow

## Procédure
1. Lire l'issue GitHub assignée avec le label "recon"
2. Extraire l'URL du site cible
3. `browser_navigate(url)` sur la page d'accueil
4. `browser_snapshot()` → identifier les liens Espace Employeurs
5. Naviguer vers la page d'inscription recruteur
6. `browser_snapshot()` → identifier tous les champs (textbox, combobox, checkbox)
7. Naviguer vers la page de connexion
8. `browser_snapshot()` → identifier les champs de connexion
9. Naviguer vers la page de publication (si accessible)
10. `browser_snapshot()` → identifier les champs du formulaire de posting
11. Détecter les captchas (iframes, reCAPTCHA, hCaptcha)
12. Détecter les multi-étapes (steppers, wizards)
13. Commit `sites/<site>/RECON.md` avec tous les champs documentés
14. Commenter sur l'issue GitHub avec le résumé
15. Ajouter le label "comptes" sur l'issue

## Format RECON.md
```markdown
# <site> — Recon
## URLs
## Inscription recruteur
### Champs (numérotés avec refs, types, placeholders, tooltips)
## Connexion
## Publication d'offre
## Captcha
## Architecture technique
```

## Detection de captchas
- reCAPTCHA: iframe src contenant `recaptcha`, div class `g-recaptcha`
- hCaptcha: iframe src contenant `hcaptcha`, div class `h-captcha`
- Captcha invisible: iframe sans dimensions visibles

## Détection multi-étapes
- Stepper: éléments avec "1", "2", "3" en heading ou generic
- Wizard: boutons "Suivant", "Précédent", "Continuer"