# BJemploi.com — Procédure de posting

## URLs
- Connexion: https://www.bjemploi.com/espace-employeurs-connexion
- Publication: https://www.bjemploi.com/depot-annonce.html

## Étapes

### 1. Charger cookies
```python
load_cookies(page, "bjemploi")
```

### 2. Naviguer vers publication
```
browser_navigate("https://www.bjemploi.com/depot-annonce.html")
browser_snapshot()
```
→ Vérifier qu'on n'est pas redirigé vers connexion. Si redirigé :
cookies expirés, intervention manuelle requise.

### 3. Formulaire de publication
Le formulaire exact n'est pas visible sans authentification (recon limitée).
Après login, identifier les champs probablement :
- Titre de l'offre
- Secteur (combobox — voir liste des secteurs ci-dessous)
- Type de contrat (combobox — CDD, CDI, etc.)
- Lieu d'affectation
- Structure/Recruteur
- Date de clôture
- Description (textarea)
- Possiblement logo/image upload

### Secteurs disponibles (combobox)
- Administration-Gestion
- Agriculture-Elevage-Mde Rural
- Auxiliaires de service
- BTP-Immobilier
- Communication-Journalisme
- Comptabilité-Finance
- Consultation
- Défense-Sécurité
- Direction-Management
- Droit-Juridique
- Economie-Finance
- Education
- Enseignement-Recherches
- Entretien-Nettoyage
- Environnement - Dev. durable
- Formation-Coaching
- Humanitaire
- Hygiène Eau et Assainissement
- Informatique-TIC-Télécom
- Logistique-Transport
- Marketing-Vente
- Prestation de service
- Ressources humaines
- Santé-Médecine-Pharmacie
- Sciences-Techniques
- Secrétariat
- Sécurité-Gardiennage
- Social
- Social-Bénévolat-Associatif
- Statistique
- Suivi-évaluation
- Tourisme-Hôtellerie-Restauration
- Autre

### Types de contrat (combobox)
- appel d'offre (AO)
- contrat à durée déterminée (CDD)
- contrat à durée déterminée renouvelable (CDD/R)
- contrat à durée indéterminée (CDI)
- consultation (Consultation)
- formation (Formation)
- intérim (Intérim)
- appel à manifestation d'intérêt (MI)
- stage (Stage)

### 4. Remplir avec job.md
Pour l'offre "Animateur·rice de quartier" :
- Titre: "Animateur·rice de quartier"
- Secteur: Communication-Journalisme (ou Social)
- Contrat: CDD (ou CDI, selon le choix)
- Lieu: Cotonou (Bénin)
- Structure: PRESSE SAINT LOUIS by Cagpemini
- Description: [contenu complet de job.md sections POURQUOI/MISSION/PROFIL/CONDITIONS]

### 5. Soumettre
- Cliquer le bouton de soumission
- `browser_snapshot()` → vérifier confirmation

### 6. Vérifier
```
browser_navigate("https://www.bjemploi.com/emplois-annonces.html")
browser_snapshot()
```
→ Chercher "Animateur" dans la liste des annonces

## Notes
- Site PHP classique, pas de hydration delay
- Si reCAPTCHA invisible dans l'iframe → cookies pré-auth sont la seule voie
- Validation admin peut être requise pour l'annonce elle-même (à confirmer)