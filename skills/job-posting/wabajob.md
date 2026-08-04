# WabaJob.com — Procédure de posting

## URLs
- Connexion: https://www.wabajob.com/login
- Dashboard recruteur: https://www.wabajob.com/recruiter-dashboard

## Étapes

### 1. Charger cookies
```python
load_cookies(page, "wabajob")
```

### 2. Naviguer vers dashboard recruteur
```
browser_navigate("https://www.wabajob.com/recruiter-dashboard")
browser_snapshot()
```
→ Si redirection vers /login, cookies expirés.
→ Attendre 2-3s (SPA React) avant snapshot.

### 3. Accéder au formulaire de publication
Depuis le dashboard recruteur, chercher un bouton :
- "Publier une offre"
- "Nouvelle offre"
- "Créer une offre"
- Ou similaire

```
browser_click(ref="@eXX")  # bouton de création d'offre
browser_snapshot()
```

### 4. Formulaire de publication
Le formulaire exact n'est pas visible sans auth (recon limitée).
L'inscription étant en 4 étapes, le formulaire de publication peut aussi
être multi-étapes.

Champs probables :
- Titre du poste
- Description
- Catégorie/Secteur
- Type de contrat
- Lieu
- Salaire
- Expérience requise
- Niveau d'étude
- Compétences

### 5. Remplir avec job.md
Pour l'offre "Animateur·rice de quartier" :
- Titre: "Animateur·rice de quartier"
- Description: [contenu complet de job.md]
- Catégorie: Communication/Journalisme (ou Social)

### 6. Soumettre
- Cliquer le bouton de soumission
- Si multi-étapes : naviguer entre étapes avec "Suivant"/"Continuer"
- `browser_snapshot()` → vérifier confirmation

### 7. Vérifier
```
browser_navigate("https://www.wabajob.com/jobs")  # ou URL des offres
browser_snapshot()
```
→ Chercher "Animateur" dans la liste

## Notes
- WabaJob a un formulaire d'inscription en 4 étapes — le formulaire de
  publication peut être similaire (multi-étapes)
- SPA React : attendre hydration avant snapshot
- Google Sign-in disponible si cookies expirés
- ⚠️ Étape 4 inscription nécessite validation email — déjà géré si compte créé