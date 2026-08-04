# Asako.mg — Procédure de posting

## URLs
- Connexion: https://www.asako.mg/connexion
- Publication: https://www.asako.mg/recruteur/offres/nouvelle

## Étapes

### 1. Charger cookies
```python
load_cookies(page, "asako")
```

### 2. Naviguer vers publication
```
browser_navigate("https://www.asako.mg/recruteur/offres/nouvelle")
browser_snapshot()
```
→ Si redirection vers /connexion, cookies expirés.
→ Attendre 2-3s (SPA React) avant snapshot.

### 3. Formulaire de publication
Asako propose une **IA de génération d'offre** à partir d'une phrase.
Deux approches possibles :

#### Option A — Utiliser l'IA Asako
- Décrire le poste en une phrase : "Animateur·rice de quartier pour écoute
  sociale et médiation"
- Laisser l'IA générer l'offre
- Relire et corriger avec le contenu de job.md
- Valider

#### Option B — Remplir manuellement
Champs probables :
- Titre du poste
- Description
- Catégorie
- Type de contrat
- Lieu
- Salaire
- Expérience
- Compétences

### 4. Remplir avec job.md
Pour l'offre "Animateur·rice de quartier" :
- Titre: "Animateur·rice de quartier"
- Description: [contenu complet de job.md]
- Catégorie: Médias & Communication (ou Social)

### 5. Soumettre
- Cliquer "Publier" ou équivalent
- `browser_snapshot()` → vérifier confirmation

### 6. Vérifier
```
browser_navigate("https://www.asako.mg/emploi")
browser_snapshot()
```
→ Chercher "Animateur" dans la liste

## Notes
- Asako est le plus moderne des 4 sites
- L'IA intégrée peut faciliter le posting (Option A)
- Asako indexé sur Google Jobs → bonne visibilité
- Plans gratuits disponibles
- SPA React : attendre hydration avant snapshot
- Pipeline de candidatures intégré (post-dépôt)