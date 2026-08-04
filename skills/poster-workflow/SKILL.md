---
name: poster-workflow
description: Poster workflow — poste une offre d'emploi sur un site via browser automation
---
# Poster Workflow

## Procédure
1. Lire l'issue GitHub assignée avec le label "posting"
2. Extraire : site cible, modèle LLM, chemin cookies, URL posting
3. Charger les cookies de session :
   ```
   file_read("sites/<site>/cookies.json")
   ```
4. `browser_navigate(posting_url)`
5. `browser_snapshot()` → vérifier qu'on n'est pas redirigé vers login
   - Si redirect → commenter issue "cookies expirés" → label "blocked"
6. Identifier les champs du formulaire via `browser_snapshot()`
7. Pour chaque champ, `browser_type(ref="@eXX", text="valeur")` avec le contenu de job.md
8. Si combobox → `browser_click(ref="@eXX")` puis sélectionner l'option
9. Si checkbox conditions → `browser_click(ref="@eXX")`
10. Si multi-étapes → `browser_click` sur "Continuer"/"Suivant" entre étapes
11. `browser_click(ref="@eXX")` sur le bouton de soumission
12. `browser_snapshot()` → vérifier le message de confirmation
13. `browser_navigate(url_offres_publiques)` → vérifier visibilité
14. `browser_vision()` → screenshot de l'offre publiée
15. Ouvrir une PR avec :
    - URL de l'offre
    - Screenshot
    - Logs des actions effectuées
16. Commenter sur l'issue avec le résultat

## Mapping des champs (depuis job.md)

| Champ générique | Valeur pour "Animateur·rice de quartier" |
|---|---|
| Titre | "Animateur·rice de quartier" |
| Secteur | Communication-Journalisme (ou Social) |
| Contrat | CDD (ou CDI) |
| Lieu | Cotonou (BJ) / Antananarivo (MG) |
| Structure | PRESSE SAINT LOUIS by Cagpemini |
| Description | [Contenu complet de job.md sections POURQUOI/MISSION/PROFIL/CONDITIONS] |
| Rémunération | 1 800 €/mois + primes |
| Date clôture | +30 jours |

## Gestion des erreurs

| Erreur | Action |
|---|---|
| Redirection login | Commenter "cookies expirés" → label "blocked" |
| Captcha apparaît | Commenter "captcha détecté" → label "blocked-captcha" |
| Champ non trouvé | `browser_snapshot()` à nouveau → chercher alternative |
| Soumission échoue | `browser_console()` → commenter l'erreur JS |
| Offre non visible | Attendre 30s → `browser_navigate` à nouveau → si toujours invisible, commenter |

## Vérification finale
- [ ] `browser_navigate` vers la page des offres publiques
- [ ] `browser_find(text="Animateur")` → l'offre doit apparaître
- [ ] `browser_vision()` → screenshot pour la PR