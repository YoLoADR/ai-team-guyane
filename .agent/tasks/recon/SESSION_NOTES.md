# Session Notes — Recon des 4 sites (2026-08-03)

## Méthode

Recon via Playwright (browser MCP) sur les 4 sites cibles. Pour chaque site :
- Page d'accueil
- Page d'inscription recruteur
- Page de connexion
- Page de publication d'offre (si accessible sans login)

## 1. BJemploi.com (Bénin)

### Inscription Recruteur — `https://www.bjemploi.com/espace-employeurs-inscription`

**Champs du formulaire :**
| # | Champ | Requis | Type | Notes |
|---|---|---|---|---|
| 1 | Email professionnel | ✅ | textbox | "Votre adresse email professionnelle*" |
| 2 | Structure (nom société, ONG) | ✅ | textbox | "Votre structure: nom de société, ONG, ...*" |
| 3 | Nom | ✅ | textbox | "Votre nom*" |
| 4 | Prénom | ✅ | textbox | "Votre prénom*" |
| 5 | Poste (DRH, DG...) | ✅ | textbox | "Votre poste comme DRH, DG, ...*" |
| 6 | Ville | ✅ | textbox | "La ville où se trouve votre structure*" |
| 7 | Pays | ✅ | combobox | Défaut: Bénin. 150+ pays dans le dropdown |
| 8 | Adresse postale | ✅ | textbox | "Adresse postale de votre structure*" |
| 9 | Téléphone | ✅ | textbox | "Téléphone au format international (+ind numéro)*" |
| 10 | Site web | ❌ | textbox | Optionnel |
| 11 | Conditions d'utilisation | ✅ | checkbox | À cocher pour activer le bouton |

**Bouton :** `Inscription Employeur` (disabled tant que champs obligatoires vides)

**Captcha :** ❌ Aucun captcha visible

**⚠️ Validation admin :** Le site précise :
> "Avant d'être active, votre demande d'inscription sera évaluée et les
> informations que vous allez saisir seront vérifiées. Si votre profil n'est
> pas celui d'un recruteur et/ou les informations ne permettent pas de vous
> identifier comme tel, nous allons rejeter votre demande."

→ **Intervention manuelle requise** : attendre la validation (1-48h estimé).

### Connexion — `https://www.bjemploi.com/espace-employeurs-connexion`

- Email + mot de passe
- Lien "Mot de passe oublié ?"
- Pas de captcha visible
- Iframe présente (possiblement pour sécurité additionnelle)

### Publication d'annonce — `https://www.bjemploi.com/depot-annonce.html`

- Redirige vers page de connexion si non authentifié
- Message : "Vous devez vous connecter à votre espace personnel Employeur pour
  pouvoir publier vous même une annonce."
- Formulaire de publication non visible sans authentification

### Architecture technique
- Site PHP classique (URLs `.html`, tables HTML, iframes)
- Pas de SPA (pas de React/Vue)
- Pas de captcha visible (mais iframe possiblement reCAPTCHA invisible)
- Dropdown pays avec ~200 options
- Pas de Google Sign-in

---

## 2. Job2mada.com (Madagascar)

### Inscription Recruteur — `https://www.job2mada.com/register`

**Toggle type de compte :** Candidat / Employeur (boutons radio)

**Champs (mode Employeur) :**
| # | Champ | Requis | Type | Notes |
|---|---|---|---|---|
| 1 | Nom complet | ✅ | textbox | "Votre nom complet" |
| 2 | Email | ✅ | textbox | "votre@email.com" |
| 3 | Nom de l'entreprise | ✅ | textbox | "Nom de votre entreprise" |
| 4 | Catégorie entreprise | ✅ | combobox | 14 options (voir ci-dessous) |
| 5 | Adresse entreprise | ❌ | textbox | "Adresse complète" |
| 6 | Siège social | ❌ | textbox | "Ville du siège" |
| 7 | Téléphone | ❌ | textbox | "+261 XX XX XXX XX" |
| 8 | Site web | ❌ | textbox | "https://www.entreprise.com" |
| 9 | Mot de passe | ✅ | textbox (password) | Min 6 caractères |
| 10 | Confirmer mot de passe | ✅ | textbox (password) | |
| 11 | Conditions d'utilisation | ✅ | checkbox (implicite) | |

**Catégories d'entreprise (combobox) :**
- Sélectionnez une catégorie (défaut)
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

**Bouton :** `Créer mon compte` (disabled tant que nom complet vide)

**Google Sign-in :** ✅ Bouton "Continuer avec Google"

**Captcha :** ❌ Aucun captcha visible

### Connexion — `https://www.job2mada.com/login`

- Email + mot de passe
- Bouton "Continuer avec Google"
- Lien "Mot de passe oublié ?" → `/forgot-password`
- Pas de captcha visible

### Publication d'offre — `https://www.job2mada.com/jobs/new`

- Redirige/affiche "Offre non trouvée" sans authentification
- Le formulaire complet n'est visible qu'après connexion

### Architecture technique
- SPA (React/Next.js probable, many console errors = React hydration)
- Notifications system (region "Notifications F8")
- Pas de captcha visible
- Google Sign-in disponible

---

## 3. Asako.mg (Madagascar)

### Inscription Recruteur — `https://www.asako.mg/inscription/recruteur`

**Le formulaire est intégré à la page landing recruteur** (pas de page séparée).

**Champs :**
| # | Champ | Requis | Type | Notes |
|---|---|---|---|---|
| 1 | Nom de l'entreprise | ✅ | textbox | Placeholder: "Ex: TeknetGroup" |
| 2 | Prénom | ✅ | textbox | Placeholder: "Rivo" |
| 3 | Nom | ✅ | textbox | Placeholder: "Rakoto" |
| 4 | Email professionnel | ✅ | textbox | Placeholder: "vous@entreprise.mg" |
| 5 | Mot de passe | ✅ | textbox (password) | Min 8 caractères |
| 6 | Site web | ❌ | textbox | Champ "Website" |

**Bouton :** `Publier ma première offre`

**Captcha :** ❌ Aucun captcha visible

**Validation messages :** Champs avec validation inline (messages d'erreur
sous chaque champ : "Le nom de l'entreprise est requis", etc.)

### Connexion — `https://www.asako.mg/connexion`

- Email + mot de passe
- Pas de Google Sign-in sur la page de connexion
- Pas de captcha visible

### Publication d'offre — `https://www.asako.mg/recruteur/offres/nouvelle`

- Redirige vers `/connexion?next=%2Frecruteur%2Foffres%2Fnouvelle` si non auth
- **Fonctionnalité IA** : Asako propose une IA qui génère l'offre à partir
  d'une phrase — pourrait faciliter le posting

### Architecture technique
- SPA moderne (React/Next.js)
- Design soigné, UX polishée
- Pas de captcha visible
- Pas de Google Sign-in (sur la page recruteur)
- Système de pricing (gratuit + plans payants)
- "500+ entreprises malgaches"
- Fonctionnalité IA de génération d'offres
- Pipeline de candidatures intégré

---

## 4. WabaJob.com (Bénin)

### Inscription Recruteur — `https://www.wabajob.com/register`

**Inscription en 4 étapes :**
1. **Compte** — Type (Candidat/Recruteur) + Email + Mot de passe + Confirmation
2. **Informations** — (non visible sans compléter l'étape 1)
3. **Récapitulatif** — Validation
4. **Vérification** — Probablement email de confirmation

**Étape 1 — Champs :**
| # | Champ | Requis | Type | Notes |
|---|---|---|---|---|
| 1 | Type de compte | ✅ | button toggle | Chercheur d'emploi / Recruteur |
| 2 | Email | ✅ | textbox | "votre.email@exemple.com" |
| 3 | Mot de passe | ✅ | textbox (password) | Bouton afficher/masquer |
| 4 | Confirmer mot de passe | ✅ | textbox (password) | Bouton afficher/masquer |

**Bouton :** `Continuer` (étape 1)

**Google Sign-in :** ✅ Bouton "S'inscrire avec Google"

**Captcha :** ❌ Aucun captcha visible sur l'étape 1

**⚠️ Étape 4 — Vérification :** Probablement validation par email.
→ **Intervention manuelle可能 requise** : cliquer sur lien de confirmation email.

### Connexion — `https://www.wabajob.com/login`

- Email professionnel + mot de passe
- Checkbox "Se souvenir de moi"
- Bouton "Continuer avec Google"
- Lien "Mot de passe oublié ?" → `/forgotPassword`
- Pas de captcha visible

### Architecture technique
- SPA moderne (React/Next.js)
- Inscription multi-étapes (4 steps)
- Google Sign-in disponible
- Pas de captcha visible sur étape 1

---

## Comparaison synthétique

| Critère | BJemploi | Job2mada | Asako | WabaJob |
|---|---|---|---|---|
| Pays | Bénin | Madagascar | Madagascar | Bénin |
| Techno | PHP classique | SPA React | SPA React | SPA React |
| Captcha | ❌ | ❌ | ❌ | ❌ |
| Google Sign-in | ❌ | ✅ | ❌ (recruteur) | ✅ |
| Validation admin | ✋ Oui | Non visible | Non visible | Non visible |
| Multi-étapes inscription | Non | Non | Non | Oui (4 étapes) |
| IA génération offre | ❌ | ❌ | ✅ | ❌ |
| Forms visibles sans login | Inscription seule | Inscription seule | Inscription seule | Inscription seule |
| Difficulté posting | Moyenne (PHP) | Facile (SPA) | Facile (IA assist) | Moyenne (4 étapes) |

## Interventions manuelles identifiées

| Site | Étape | Action manuelle |
|---|---|---|
| BJemploi | Post-inscription | ⏳ Attendre validation admin (1-48h) |
| WabaJob | Étape 4 inscription | ✋ Cliquer lien confirmation email |
| Tous | Post-inscription | 🔄 Exporter cookies de session (Playwright snippet) |

## Prochaines étapes

1. ✅ Recon des 4 sites — TERMINÉE
2. ⏳ Créer les comptes recruteurs (manuel, par Yohann)
3. ⏳ Exporter les cookies de session après création
4. ⏳ Provisionner VPS Contabo dédié
5. ⏳ Installer Hermes Agent + Docker
6. ⏳ Créer skill job-posting (SKILL.md)
7. ⏳ Créer 4 profils Hermes
8. ⏳ Tester en one-shot avec job.md