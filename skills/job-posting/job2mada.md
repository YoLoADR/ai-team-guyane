# Job2mada.com — Procédure de posting

## URLs
- Connexion: https://www.job2mada.com/login
- Publication: https://www.job2mada.com/jobs/new (URL exacte à confirmer après login)

## Étapes

### 1. Charger cookies
```python
load_cookies(page, "job2mada")
```

### 2. Naviguer vers publication
```
browser_navigate("https://www.job2mada.com/jobs/new")
browser_snapshot()
```
→ Si redirection vers /login, cookies expirés.
→ Attendre 2-3s (SPA React hydration) avant snapshot.

### 3. Formulaire de publication
Le formulaire exact n'est pas visible sans auth (recon limitée).
Après login, identifier les champs probablement :
- Titre du poste
- Description
- Catégorie/Secteur
- Type de contrat
- Lieu
- Salaire
- Expérience requise
- Niveau d'étude
- Compétences requises

### Catégories d'entreprise (inscriptions)
- Technologie
- Finance & Banque
- Santé & Médical
- Éducation
- Commerce & Retail
- Industrie & Manufacturing
- Construction & BTP
- Tourisme & Hôtellerie
- Transport & Logistique
- Énergie & Environnement
- Agriculture & Agroalimentaire
- Médias & Communication
- Consulting & Services
- Autre

### 4. Remplir avec job.md
Pour l'offre "Animateur·rice de quartier" :
- Titre: "Animateur·rice de quartier"
- Catégorie: Médias & Communication (ou Autre)
- Description: [contenu complet de job.md]

### 5. Soumettre
- Cliquer le bouton de soumission
- `browser_snapshot()` → vérifier confirmation

### 6. Vérifier
```
browser_navigate("https://www.job2mada.com/jobs")
browser_snapshot()
```
→ Chercher "Animateur" dans la liste

## Notes
- SPA React : attendre 2-3s après chaque navigate avant snapshot
- Google Sign-in disponible si cookies expirés (mais nécessite intervention)
- Notifications system peut afficher des popups (region "Notifications F8")