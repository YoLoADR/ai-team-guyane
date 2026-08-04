---
name: review-workflow
description: Lead Review workflow — vérifie les postings d'offres d'emploi et approuve ou refuse
---
# Lead Review Workflow

## Procédure
1. Lire les PRs ouvertes dans YoLoADR/ai-hirekit
2. Pour chaque PR, extraire l'URL de l'offre publiée
3. `browser_navigate(url_offre)`
4. `browser_snapshot()` → analyser le contenu de l'offre
5. `browser_vision()` → screenshot pour vérification visuelle

## Checklist de review

### 1. Titre
- [ ] Le titre contient "Animateur·rice de quartier"
- [ ] Pas de caractère d'encodage cassé (Ã©, â€", etc.)
- [ ] Pas de balise HTML visible dans le titre

### 2. Description
- [ ] Section "POURQUOI CE POSTE" présente
- [ ] Section "VOTRE MISSION" présente (2 phases)
- [ ] Section "PROFIL RECHERCHÉ" présente
- [ ] Section "CONDITIONS" présente
- [ ] Section "CONFIDENTIALITÉ" présente
- [ ] Section "À PROPOS" présente
- [ ] Pas de champ vide
- [ ] Pas d'erreur d'encodage

### 3. Champs structurés
- [ ] Secteur correct (Communication-Journalisme ou Social)
- [ ] Type de contrat correct (CDD/CDI)
- [ ] Lieu correct (Cotonou pour BJ, Antananarivo pour MG)
- [ ] Structure : "PRESSE SAINT LOUIS by Cagpemini"
- [ ] Rémunération mentionnée

### 4. Visibilité
- [ ] L'offre est visible sans login
- [ ] L'offre apparaît dans la liste des annonces
- [ ] L'URL est accessible publiquement

### 5. Sécurité
- [ ] Pas de credentials exposés dans l'offre
- [ ] Pas d'informations personnelles (email/tel de Yohann)
- [ ] Le mot "Cagpemini" est bien distinct de "Capgemini"

## Décision

### SI OK (tous les critères validés)
```bash
gh pr review <pr> --approve --body "✅ Offre vérifiée et publiée correctement sur <site>. Titre, description, champs structurés et visibilité tous conformes."
gh pr merge <pr> --squash
gh issue close <issue> --comment "Offre publiée avec succès sur <site>."
```
→ Label "success" sur l'issue
→ Move → "Done"

### SI KO (problème détecté)
```bash
gh pr review <pr> --request-changes --body "
## ❌ Problèmes détectés

1. **<champ>** : <problème> → <suggestion>
2. **<champ>** : <problème> → <suggestion>

## Actions requises
- Corriger les champs ci-dessus
- Re-soumettre le formulaire
- Vérifier à nouveau
"
```
→ Label "needs-work" sur l'issue
→ Move → "Posting In Progress"

## Format des commentaires
Toujours utiliser le format :
```
<champ>: <problème> → <suggestion>
```

Exemple :
```
Titre: Contient "AnimateurÂ·rice" (encodage cassé) → Utiliser "Animateur·rice"
Description: Section "PROURQUOI" au lieu de "POURQUOI" → Corriger la typo
Secteur: "Autre" au lieu de "Communication-Journalisme" → Changer pour le secteur correct
```