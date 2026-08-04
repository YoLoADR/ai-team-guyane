# Session Log — ai-hirekit (2026-08-03)

## Modèles ayant conduit cette session

Cette session a été conduite avec **Kimi (kimi-k2.7-code)** et
**GLM (glm-5.2)** via Ollama Cloud :
- **GLM-5.2** : analyse initiale, recon des 4 sites via Playwright,
  recherche Context7 sur Hermes Agent, rédaction de toute la documentation,
  configuration Hermes sur le VPS, routing Telegram
- **Kimi-k2.7-code** : code (profils Hermes, docker-compose, scripts
  shell, skills), structuration des fichiers, déploiement sur le VPS

Les modèles minimax-m3 et deepseek-v4-pro sont assignés à des rôles
spécifiques pour l'A/B test mais n'ont pas participé à la construction.

---

## Journal complet de la session de conversation

---

## 1. Demande initiale de l'utilisateur

> J'ai besoin de poster des offres d'emploi (un·e animateur·rice de quartier :
> /Users/YoLoADR/Factory/ai-team/job.md) sur BJemploi.com, job2mada.com,
> asako.mg, wabajob.com ; est-ce possible de demander à des un agent hermes
> utilisant playwright ? Car dans ces taches certains agent on fait des choses
> remarquable : /Users/YoLoADR/Factory/ai-team/.agent/tasks ; Si oui
> j'aimerai créer un ou des agents Hermes sur contabo, avec des configurations
> différents afin de voir le ou lesquel reussise à poste des jobs

### Contexte lu
- `/Users/YoLoADR/Factory/ai-team/job.md` — offre d'emploi "Animateur·rice
  de quartier" pour PRESSE SAINT LOUIS by Cagpemini
- `/Users/YoLoADR/Factory/ai-team/.agent/tasks/` — 3 dossiers de loop
  engineering (PO/Dev/Lead Dev bots)
- `/Users/YoLoADR/Factory/ai-team/.agent/tasks/loop-engineering-todo-app-v2/LOOP_ENGINEERING_PLAN.md`
  — Plan v2 avec OpenHands, 3 profils LLM (glm-5.2 PO, kimi-k2.7-code Dev,
  qwen3.5:397b Lead-Review), VPS Contabo "carapace"
- `/Users/YoLoADR/Factory/ai-team/.agent/tasks/todo-app-loop-engineering/PLAN.md`
  — Plan v1 avec Hermes Agent, 3 profils (minimax-m3 PO, kimi-k2.7-code Dev,
  deepseek-v4-pro Lead Dev)
- `/Users/YoLoADR/Factory/ai-team/infra/hermes/config.yaml` — config Hermes
  existante (Ollama Cloud, toolsets github/git/shell/file)
- `/Users/YoLoADR/Factory/ai-team/infra/hermes/profiles.yaml` — profils
  existants (po, dev, leaddev)

### Note sur "Pourquoi pas Hermes"
Extrait du plan v2 :
> Hermes Agent (Nous Research) est un agent mono-utilisateur avec
> auto-apprentissage de skills. Son "loop engineering" est un mécanisme
> d'auto-amélioration des skills de l'agent, pas un workflow multi-agent
> PO→Dev→LeadDev. Pour notre cas (3 rôles isolés qui interagissent via GitHub),
> OpenHands est adapté car :
> - Support natif multi-agent avec workspaces isolés (Docker volumes séparés)
> - Plugin pr-review GitHub Actions documenté
> - Endpoint LLM OpenAI-compatible (base_url configurable → Ollama Cloud)
> - Automations event-driven (issues.labeled, pull_request.opened)
> - Config par agent via config.toml

Mais pour le cas du posting d'offres d'emploi, Hermes est **adapté** car :
- C'est une tâche mono-agent (pas besoin de multi-agent)
- Hermes a un browser toolset intégré (Playwright)
- browse.sh skills pour automatisation site-spécifique
- Cron jobs pour tâches récurrentes
- Skills custom créables

---

## 2. Recherche Context7 sur Hermes Agent

### 2.1. Resolve library ID
Bibliothèque trouvée : `/nousresearch/hermes-agent`
- 33151 code snippets
- Source Reputation: High
- Benchmark Score: 81.61
- Versions: v2026.4.8, v2026.4.16, v2026.6.5

Autres bibliothèques pertinentes :
- `/llmstxt/hermes-agent_nousresearch_llms_txt` (4868 snippets, score 79.01)
- `/cclank/hermes-wiki` (1952 snippets, architecture wiki, score 68.19)
- `/fathah/hermes-desktop` (592 snippets, app desktop, score 66.56)
- `/ksimback/hermes-ecosystem` (19727 snippets, Atlas, score 67)

### 2.2. Query : Browser Toolset
Source : https://github.com/nousresearch/hermes-agent/blob/main/website/docs/reference/tools-reference.md

#### Tools disponibles
- **browser_back**: Navigate back to the previous page in browser history.
- **browser_click**: Click on an element identified by its ref ID (e.g., '@e5').
- **browser_console**: Get browser console output and JavaScript errors.
- **browser_get_images**: Get a list of all images on the current page with
  their URLs and alt text.
- **browser_navigate**: Navigate to a URL in the browser. Must be called
  before other browser tools.
- **browser_press**: Press a keyboard key.
- **browser_scroll**: Scroll the page in a direction.
- **browser_snapshot**: Get a text-based snapshot of the current page's
  accessibility tree.
- **browser_type**: Type text into an input field identified by its ref ID.
- **browser_vision**: Take a screenshot of the current page for visual
  inspection.

#### Automate Custom Page Actions with Playwright and DynamicFetcher
Source : https://github.com/nousresearch/hermes-agent/blob/main/optional-skills/research/scrapling/SKILL.md

```python
from playwright.sync_api import Page
from scrapling.fetchers import DynamicFetcher

def scroll_and_click(page: Page):
    page.mouse.wheel(0, 3000)
    page.wait_for_timeout(1000)
    page.click('button.load-more')
    page.wait_for_selector('.extra-results')

page = DynamicFetcher.fetch('https://example.com', page_action=scroll_and_click)
results = page.css('.extra-results .item::text').getall()
```

#### web_search vs web_extract vs browser vs computer-use
Source : https://github.com/nousresearch/hermes-agent/blob/main/tools/web_tools.py

- **web_search** : retourne seulement les métadonnées (URLs, titres, descriptions)
- **web_extract** : retourne le contenu complet d'une page spécifique
- **browser** : automation interactive complète (navigate, click, type, scroll)
- **computer_use** : contrôle desktop en arrière-plan (screenshots, mouse,
  keyboard) — pas pour le web browsing

### 2.3. Query : Docker + shm-size
Source : https://github.com/nousresearch/hermes-agent/blob/main/website/docs/user-guide/docker.md

```sh
docker run -d \
  --name hermes \
  --shm-size=1g \
  -v ~/.hermes:/opt/data \
  nousresearch/hermes-agent gateway run
```

→ **shm-size=1g obligatoire** pour Playwright browser tools (mémoire partagée
pour Chrome).

### 2.4. Query : Skills + browse.sh
Source : https://github.com/nousresearch/hermes-agent/blob/main/website/docs/user-guide/features/skills.md

#### browse.sh
Catalogue de 200+ skills pré-faits pour automatiser des sites spécifiques.

```bash
hermes skills search airbnb --source browse-sh
hermes skills inspect browse-sh/airbnb.com/search-listings-ddgioa
hermes skills install browse-sh/airbnb.com/search-listings-ddgioa
```

#### Commands skills
```bash
hermes skills browse                      # list everything available
hermes skills browse --source official     # browse only official optional skills
hermes skills search <query>              # find skills by keyword
hermes skills install <identifier>         # install one (runs a security scan first)
hermes skills config                       # Enable/disable skills per platform
hermes skills check|update|uninstall|publish PATH  # Manage skill lifecycle
hermes skills tap add REPO                 # Add a GitHub repo as a skill source
hermes bundles                             # Skill bundles (one /<name> alias loads several skills)
```

→ **Action à faire** : lancer `hermes skills search emploi --source browse-sh`
  pour vérifier si des skills existent pour les 4 sites cibles.

### 2.5. Query : Custom skill creation
Source : https://github.com/nousresearch/hermes-agent/blob/main/skills/software-development/hermes-agent-skill-authoring/SKILL.md

#### Template SKILL.md
```markdown
# <Title>

## Overview
One or two paragraphs: what and why.

## When to Use
- Bulleted triggers
- "Don't use for:" counter-triggers

## <Topic sections specific to the skill>
- Quick-reference tables are common
- Code blocks with exact commands
- Hermes-specific recipes (tests via scripts/run_tests.sh, ui-tui paths, etc.)

## Common Pitfalls
Numbered list of mistakes and their fixes.

## Verification Checklist
- [ ] Checkbox list of post-action verifications

## One-Shot Recipes (optional)
Named scenarios → concrete command sequences.
```

### 2.6. Query : Cron jobs
Source : https://github.com/nousresearch/hermes-agent/blob/main/website/docs/user-guide/features/cron.md

#### Create a scheduled task
```python
cronjob(
    action="create",
    skill="blogwatcher",
    prompt="Check the configured feeds and summarize anything new.",
    schedule="0 9 * * *",
    name="Morning feeds",
)
```

#### Via chat
```bash
/cron add 30m "Remind me to check the build"
/cron add "every 2h" "Check server status"
/cron add "every 1h" "Summarize new feed items" --skill blogwatcher
```

#### Schedule formats supportés
Source : https://github.com/nousresearch/hermes-agent/blob/main/cron/jobs.py

```python
def parse_schedule(schedule: str) -> Dict[str, Any]:
    """
    Examples:
        "30m"              → once in 30 minutes
        "2h"               → once in 2 hours
        "every 30m"        → recurring every 30 minutes
        "every 2h"         → recurring every 2 hours
        "0 9 * * *"        → cron expression
        "2026-02-03T14:00" → once at timestamp
    """
```

### 2.7. Query : Automating Web Form Submission
Source : https://github.com/nousresearch/hermes-agent/blob/main/website/docs/user-guide/features/browser.md

#### Workflow type
```text
User: Sign up for an account on example.com with my email john@example.com

Agent workflow:
1. browser_navigate("https://example.com/signup")
2. browser_snapshot()  → sees form fields with refs
3. browser_type(ref="@e3", text="john@example.com")
4. browser_type(ref="@e5", text="SecurePass123")
5. browser_click(ref="@e8")  → clicks "Create Account"
6. browser_snapshot()  → confirms success
```

→ **C'est exactement le pattern** qu'on utilisera pour le posting d'offres.

### 2.8. Query : Browser Provider architecture
Source : https://github.com/cclank/hermes-wiki/blob/master/concepts/browser-tool-architecture.md

> Browser Tool est dans `tools/browser_tool.py`, fournit une automation
> multi-backend. Tous les backends exposent la même interface (navigate, click,
> type, scroll, vision, console, pdf). Basé sur l'accessibility tree
> (ariaSnapshot) — représentation textuelle de la page, l'LLM n'a pas besoin
> de vision pour opérer.

Source : https://github.com/cclank/hermes-wiki/blob/master/changelog/2026-05-19-update.md

```python
class BrowserProvider(abc.ABC):
    @abstractmethod
    async def navigate(self, url: str, **kwargs) -> str: ...
    @abstractmethod
    async def execute_script(self, script: str, **kwargs) -> str: ...
```

### 2.9. Query : MCP server config
Source : https://github.com/nousresearch/hermes-agent/blob/main/website/docs/user-guide/features/codex-app-server-runtime.md

```toml
[mcp_servers.hermes-tools]
command = "/path/to/python"
args = ["-m", "agent.transports.hermes_tools_mcp_server"]
env = { HERMES_HOME = "/your/.hermes", PYTHONPATH = "...", HERMES_QUIET = "1" }
startup_timeout_sec = 30.0
tool_timeout_sec = 600.0
```

### 2.10. Query : Browser CDP Passthrough
Source : https://github.com/cclank/hermes-wiki/blob/master/changelog/2026-05-16-update.md

> Browser CDP evaluations share a single WebSocket connection to Chrome,
> significantly speeding up page interactions from seconds to milliseconds.

### 2.11. Query : Plugin System Browser Provider
Source : https://github.com/cclank/hermes-wiki/blob/master/changelog/2026-05-17-update.md

```python
# plugins.py:613
def register_browser_provider(self, provider: Type[BrowserProvider]) -> None:
    self.browser_registry.register(provider)
```

---

## 3. Réponse initiale — Analyse et questions

### Réponse donnée à l'utilisateur

**Oui, c'est possible** — Hermes Agent a un browser toolset intégré (basé
Playwright) qui permet de naviguer, remplir des formulaires et poster sur des
sites web.

### Ce que Hermes peut faire
- Browser tools natifs : navigate, snapshot, click, type, scroll, vision
- browse.sh : catalogue de 200+ skills pré-faits
- Cron jobs : programmation de tâches récurrentes
- Skills custom : SKILL.md avec instructions de posting

### Ce qu'il faut vérifier AVANT de créer les agents
1. Les 4 sites ont-ils des API publiques ? → Non, browser automation seule voie
2. Les 4 sites nécessitent-ils un compte ? → Oui, comptes recruteurs requis
3. Captcha / anti-bot ? → À vérifier (recon)
4. browse.sh skills existants ? → À lancer sur le VPS

### Risques majeurs identifiés
| Risque | Mitigation |
|---|---|
| Captchas (reCAPTCHA, hCaptcha) | Sessions pré-auth + cookies |
| Changement de structure des sites | browser_snapshot s'adapte |
| Rate limiting / IP bannie | Proxies différents par agent + delays |
| Cookies de session expirés | Skill avec étape de re-login |

### Questions posées à l'utilisateur

3 questions avec options :

#### Q1 : Comptes existants
> As-tu déjà des comptes recruteurs sur ces 4 sites ?
Options :
- Comptes déjà créés
- À créer manuellement (Recommandé)
- L'agent doit les créer

#### Q2 : Infrastructure
> Quel est ton serveur Contabo ? As-tu déjà un VPS Contabo avec Hermes installé ?
Options :
- Hermes déjà installé
- VPS vierge
- Pas encore de VPS

#### Q3 : Stratégie captcha
> Pour le risque captcha, quelle approche préfères-tu ?
Options :
- Sessions pré-authentifiées (Recommandé)
- Service de résolution (2captcha/CapSolver)
- Mode headful + manuel

---

## 4. Réponses de l'utilisateur

### Q1 : Comptes
> L'agent doit les créer

### Q2 : Infrastructure
> Je souhaite créer un agent dédié sur contabo ; isolé pour éviter les
> conflits avec mes autres process

### Q3 : Captcha
> Sessions pré-authentifiées (Recommandé)

→ Décision : créer un VPS Contabo **dédié** pour ce projet, isolé des autres
process existants (carapace, etc.)

---

## 5. Phase de recon (Playwright browser MCP)

### 5.1. BJemploi.com — Accueil
- Navigate to https://www.bjemploi.com
- Titre : "BJemploi.com 1er site d'offres d'emploi, d'appels d'offre..."
- Header : logo, recherche, réseaux sociaux (Facebook, X, WhatsApp, Telegram)
- Nav : Espace Candidats, Espace Employeurs, Ressources, Répertoires, Appels
  d'offre, Contact, Newsletter
- Footer : liens Espace Employeurs (Connexion, Inscription, Publier une
  annonce, Recherche CV, Mot de passe, Les demandes)
- 14 annonces récentes visibles
- Dernière mise à jour : 03-08-2026 07:27:44
- Sites partenaires : BFemploi.com, MLemploi.com, NigerEmploi.com,
  SesameHosting.com

### 5.2. BJemploi.com — Inscription Employeur
- Click sur "Inscription" (footer Espace Employeurs)
- URL : https://www.bjemploi.com/espace-employeurs-inscription
- Titre : "BJemploi.com - Inscription Employeurs"

#### Texte d'introduction
> L'inscription est 100% GRATUITE! Après votre inscription, vous pouvez gérer
> votre espace "Recruteur". Vous pourrez ainsi consulter les CV des candidats
> qui souhaitent travailler dans votre secteur d'activité. Plus généralement,
> vous pouvez faire des recherches de CV selon vos critères. Votre inscription
> vous permet aussi de déposer des offres d'emploi et les candidatures pourront
> être faites directement sur notre site si vous le souhaitez. Ces candidatures
> seront automatiquement enregistrées dans votre espace. Nous pouvons aussi
> vous les envoyer par email ou par la poste.
>
> Vous êtes dans l'espace d'enregistrement des EMPLOYEURS (RECRUTEURS). Avant
> d'être active, votre demande d'inscription sera évaluée et les informations
> que vous allez saisir seront vérifiées. Si votre profil n'est pas celui d'un
> recruteur et/ou les informations ne permettent pas de vous identifer comme
> tel, nous allons rejeter votre demande.
>
> Les champs marqués par * sont obligatoires.

#### Champs identifiés (snapshot complet)
1. Email professionnel* — textbox, ref=f1e77, placeholder "Votre adresse email
   professionnelle*", tooltip "Nous vous conseillons de mettre une adresse email
   associée à votre Structure (c'est à dire une adresse Pro)"
2. Structure (nom société, ONG)* — textbox, ref=f1e85, placeholder "Votre
   structure: nom de société, ONG, ...*", tooltip "Le nom exact de votre
   Structure"
3. Nom* — textbox, ref=f1e93, placeholder "Votre nom*"
4. Prénom* — textbox, ref=f1e99, placeholder "Votre prénom*"
5. Poste* — textbox, ref=f1e105, placeholder "Votre poste comme DRH, DG, ...*",
   tooltip "Poste que vous occupez au sein de votre Structure"
6. Ville* — textbox, ref=f1e113, placeholder "La ville où se trouve votre
   structure*"
7. Pays* — combobox, ref=f1e119, défaut "Bénin", ~200 pays (Afghanistan à
   Zimbabwe), tooltip "Pays où se trouve votre structure"
8. Adresse postale* — textbox, ref=f1e127, placeholder "Adresse postale de
   votre structure*"
9. Téléphone* — textbox, ref=f1e133, placeholder "Téléphone au format
   international (+ind numéro)*"
10. Site web — textbox, ref=f1e139, placeholder "Site web de votre structure"
    (optionnel)
11. Conditions d'utilisation — checkbox, ref=f1e144
12. Bouton "Inscription Employeur" — ref=f1e149 (disabled tant que checkbox
    non cochée)
13. Bouton "Annuler" — ref=f1e150

#### Iframe
Une iframe (ref=f1e153) est présente après le formulaire — possiblement
reCAPTCHA invisible ou analytics.

#### Captcha
❌ Aucun captcha visible dans le snapshot

#### ⚠️ Validation admin
Le site indique clairement que l'inscription sera évaluée manuellement avant
activation.

### 5.3. Job2mada.com — Accueil
- Navigate to https://www.job2mada.com
- Titre : "Job2mada - Plateforme d'emploi Madagascar | Offres et recrutement"
- Header : logo, navigation, boutons Connexion + Inscription
- Hero : "Trouvez le Job de Vos Rêves à Madagascar"
- "plus de 6 offres disponibles"
- Sections : offres d'emploi, services, "Pourquoi choisir Job2Mada"
- Pas de mention "recruteur" sur la page d'accueil

### 5.4. Job2mada.com — Inscription
- Navigate to https://www.job2mada.com/register
- Titre : "Job2mada - Offres d'emploi à Madagascar | Recrutement"
- Heading : "Créer un compte"
- Sous-titre : "Rejoignez la communauté Job2mada"
- Bouton "Continuer avec Google"

#### Toggle type de compte
- "Je suis un(e)" avec 2 options : Candidat / Employeur
- Click sur "Candidat" d'abord (ref=f10e30) — pas de changement visible
- Click sur "Employeur" (ref=f10e38) — champs supplémentaires apparaissent

#### Champs mode Candidat
1. Nom complet* — textbox (ref=f10e47)
2. Email* — textbox (ref=f10e51)
3. Mot de passe* — password (ref=f10e56), min 6 caractères, bouton afficher
4. Confirmer mot de passe* — password (ref=f10e64), bouton afficher

#### Champs mode Employeur (supplémentaires)
5. Nom de l'entreprise* — textbox (ref=f10e79), placeholder "Nom de votre
   entreprise"
6. Catégorie de l'entreprise* — combobox (ref=f10e82), 15 options :
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
7. Adresse de l'entreprise — textbox (ref=f10e86), placeholder "Adresse
   complète" (optionnel)
8. Siège social — textbox (ref=f10e89), placeholder "Ville du siège" (optionnel)
9. Téléphone — textbox (ref=f10e93), placeholder "+261 XX XX XXX XX" (optionnel)
10. Site web — textbox (ref=f10e96), placeholder
    "https://www.entreprise.com" (optionnel)
11. Mot de passe* — password (ref=f10e56), min 6 caractères
12. Confirmer mot de passe* — password (ref=f10e64)
13. Bouton "Créer mon compte" (disabled tant que nom complet vide)
14. Texte : "En créant un compte, vous acceptez nos Conditions d'utilisation
    et notre Politique de confidentialité"
15. Lien "Déjà un compte ? Se connecter" → /login

#### Captcha
❌ Aucun captcha visible

### 5.5. Job2mada.com — Connexion
- Navigate to https://www.job2mada.com/login
- Heading : "Connexion"
- Sous-titre : "Connectez-vous à votre compte Job2mada"
- Bouton "Continuer avec Google"
- Email (textbox, ref=f24e31)
- Mot de passe (ref=f24e34)
- Bouton "Se connecter" (ref=f24e40)
- Lien "Mot de passe oublié ?" → /forgot-password
- Lien "Pas encore de compte ? Créer un compte" → /register

### 5.6. Asako.mg — Accueil
- Navigate to https://www.asako.mg
- Titre : "Asako.mg — Offres d'emploi à Madagascar 🇲🇬"
- Header : logo Asako, navigation (Offres d'emploi, Emploi CDD, Freelance,
  Offre de stage, Offres urgentes), boutons Connexion + Créer mon profil
- Find "Inscription" → 6 matches trouvés
- Footer : liens Candidats (Offres d'emploi, Créer mon profil, Mon Asako,
  Blog), Recruteurs (Publier une offre, Asako Recruteur, Tarifs, Annuaire
  entreprises)
- "© 2026 Asako.mg - Le premier site d'emploi à Madagascar"

### 5.7. Asako.mg — Inscription Recruteur
- Navigate to https://www.asako.mg/inscription/recruteur
- Titre : "Recrutez vos talents à Madagascar — Asako.mg"
- Heading : "Le recrutement à Madagascar, enfin simple"
- Sous-titre : "Publiez une offre, recevez des candidatures, suivez chaque
  profil dans votre pipeline — le tout depuis un seul outil, pensé pour vous."

#### Formulaire (intégré à la landing page)
- Heading : "Créez votre espace recruteur"
- "Gratuit, prêt en 30 secondes"

#### Champs
1. Nom de l'entreprise* — textbox (ref=f14e71), placeholder "Ex: TeknetGroup",
   validation "Le nom de l'entreprise est requis"
2. Prénom* — textbox (ref=f14e76), placeholder "Rivo", validation "Le prénom
   est requis"
3. Nom* — textbox (ref=f14e80), placeholder "Rakoto", validation "Le nom est
   requis"
4. Email professionnel* — textbox (ref=f14e84), placeholder
   "vous@entreprise.mg", validation "L'adresse email est requise"
5. Mot de passe* — password (ref=f14e88), placeholder "Minimum 8 caractères",
   bouton "Afficher le mot de passe", validation "Le mot de passe est requis"
6. Website — textbox (ref=f14e93) (champ honeypot anti-spam ?)

#### Bouton
"Publier ma première offre" (ref=f14e94)

#### Captcha
❌ Aucun captcha visible

#### Sections de la landing page
- "500+ entreprises malgaches nous font confiance" : GROUPE ALTHEA, RADISSON
  Hotels, ABL OUTSOURCING, Passion Gourmande, VALIHA HOTEL, ScaleMyCrew,
  Assistant Ventures Network, KALDERA
- "Tout ce dont vous avez besoin, rien de superflu"
- "L'IA rédige, vous validez" : "Décrivez le poste en une phrase — l'IA génère
  une offre complète et structurée."
- "Visible là où ça compte" : Google Jobs, communauté Facebook 100k+, 5000
  visiteurs/jour
- "Rejoignez les recruteurs qui ont simplifié leur process"
- Stats : 100k+ communauté Facebook, 40 000+ CVs, ~30 candidatures/offre,
  500+ entreprises
- "Prêt en 3 étapes" : 1. Créez votre compte (30 sec), 2. Publiez votre offre
  (2 min), 3. Recevez et suivez (Automatique)
- Pricing : mensuel / trimestriel -15%, plans gratuits + payants

### 5.8. Asako.mg — Connexion
- Navigate to https://www.asako.mg/connexion (redirigée depuis
  /recruteur/offres/nouvelle)
- Heading : "Content de vous revoir"
- Email (textbox, ref=f33e32)
- Mot de passe (ref=f33e37)
- Bouton "Se connecter" (ref=f33e43)
- Liens : "Je cherche un emploi Créez votre profil gratuitement" →
  /inscription/candidat, "Je recrute Publiez votre première offre" →
  /inscription/recruteur

### 5.9. WabaJob.com — Accueil
- Navigate to https://www.wabajob.com
- Titre : "WabaJob - Offres d'emploi au Benin | Recrutement et Carrieres"
- Header : logo W WABAJOB., navigation (dashboard, recruiter-dashboard,
  trainings, recruiters, Blog, Contact), boutons S'inscrire + Login
- Find "recruteur" → 3 matches
- Section "Pour les Recruteurs" : "Publication d'offres d'emploi",
  "Messagerie intégrée" (communication directe avec candidats)
- Lien "Créer mon compte recruteur" → /register

### 5.10. WabaJob.com — Inscription
- Navigate to https://www.wabajob.com/register
- Titre : "Inscription - WabaJob | Creez votre compte"
- Heading : "Créer un compte"
- Sous-titre : "Accédez aux meilleures opportunités professionnelles au Bénin"

#### Stepper (4 étapes)
1. Compte
2. Informations
3. Récapitulatif
4. Vérification

#### Sélection type de compte
- Bouton "Chercheur d'emploi" (ref=f16e92) : "Je cherche de nouvelles
  opportunités professionnelles"
- Bouton "Recruteur" (ref=f16e102) : "Je souhaite publier des offres et
  recruter"

#### Click sur "Recruteur"
- Bouton actif = "Recruteur"

#### Inscription Rapide
- Bouton "S'inscrire avec Google" (ref=f16e114)

#### Champs étape 1
1. Email — textbox (ref=f16e129), placeholder "votre.email@exemple.com"
2. Mot de passe — password (ref=f16e137), bouton afficher (ref=f16e138)
3. Confirmer mot de passe — password (ref=f16e149), bouton afficher
   (ref=f16e150)
4. Bouton "Continuer" (ref=f16e156)

#### Liens
- "Vous avez déjà un compte ? Se connecter" → /login
- "Besoin d'aide ? Contactez-nous" → /contact

#### Captcha
❌ Aucun captcha visible sur étape 1

#### ⚠️ Étape 4 — Vérification
Non visible sans compléter les étapes 1-3. Probablement validation par email.

### 5.11. WabaJob.com — Connexion
- Navigate to https://www.wabajob.com/login
- Heading : "Connexion à votre compte"
- Email professionnel — textbox (ref=f34e84)
- Mot de passe — password (ref=f34e93), bouton "Afficher le mot de passe"
- Checkbox "Se souvenir de moi" (ref=f34e100)
- Bouton "Se connecter" (ref=f34e103)
- Bouton "Continuer avec Google" (ref=f34e110)
- Lien "Mot de passe oublié ?" → /forgotPassword
- Lien "Vous n'avez pas de compte ? Créer un compte" → /register

### 5.12. BJemploi.com — Connexion
- Navigate to https://www.bjemploi.com/espace-employeurs-connexion
- Titre : "BJemploi.com - Connexion Employeur"
- Heading : "Connexion Espace Recruteur"
- Iframe présente (ref=f27e85) — possiblement reCAPTCHA invisible
- Texte : "Se connecter, Mot de passe oublié ?, Pas encore inscrit? Cliquez
  ici"

### 5.13. BJemploi.com — Publication d'annonce
- Navigate to https://www.bjemploi.com/depot-annonce.html
- Message : "Vous devez vous connecter à votre espace personnel Employeur
  pour pouvoir publier vous même une annonce. Vous pouvez vous identifier en
  cliquant ici. Si vous n'avez pas encore de compte Employeur, veuillez vous
  inscrire en cliquant ici."
- Iframe (ref=f17e133) présente — possiblement reCAPTCHA
- Formulaire non visible sans authentification

### 5.14. Job2mada.com — Tentative publication
- Navigate to https://www.job2mada.com/jobs/new
- Réponse : "Offre non trouvée" + "Erreur lors du chargement de l'offre"
- Bouton "Retour aux offres"
- → Formulaire non accessible sans auth

---

## 6. Décisions prises pendant la session

### 6.1. Architecture agents
- **4 profils Hermes séparés**, 1 par site, avec des modèles LLM différents
  pour A/B test
- Modèles choisis d'après les contextes existants (ai-team) :
  - bjemploi-poster → kimi-k2.7-code (précis formulaires)
  - job2mada-poster → glm-5.2 (polyvalent)
  - asako-poster → minimax-m3 (grand contexte)
  - wabajob-poster → deepseek-v4-pro (raisonnement multi-étapes)

### 6.2. Authentification
- Sessions pré-authentifiées (cookies JSON) — choisi par l'utilisateur
- L'agent ne crée pas les comptes (captchas éventuels, validation admin)
- L'utilisateur crée les comptes manuellement
- Cookies exportés via Playwright `page.context().cookies()`

### 6.3. Infrastructure
- VPS Contabo **dédié** (nouveau, isolé des autres process)
- Docker Compose avec 4 conteneurs Hermes
- `shm-size=1g` obligatoire pour Playwright/Chrome

### 6.4. Template reproductible
- Projet nommé `ai-hirekit` (suffixe -kit pour template)
- Structure reproductible pour autres pays/langues
- 3 noms suggérés : ai-hirekit, ai-recruitkit, ai-scoutkit
- Choix de l'utilisateur : **ai-hirekit**

### 6.5. Création du projet
- Chemin : `/Users/YoLoADR/Factory/ai-hirekit/`
- Structure complète créée avec tous les fichiers
- Git initialisé

---

## 7. Fichiers créés dans ai-hirekit/

| Fichier | Rôle |
|---|---|
| `README.md` | Présentation + usage template |
| `job.md` | Offre d'emploi + comptes recruteurs (à renseigner) |
| `docker-compose.yml` | 4 conteneurs Hermes (1 par site) |
| `.gitignore` | Cookies, .env, secrets exclus |
| `.agent/tasks/recon/context.md` | Contexte du projet |
| `.agent/tasks/recon/PLAN.md` | Plan détaillé (phases, architecture) |
| `.agent/tasks/recon/SESSION_NOTES.md` | Notes de recon (synthèse) |
| `.agent/tasks/recon/SESSION_LOG.md` | Ce fichier — journal complet |
| `.agent/tasks/recon/insights.md` | Hypothèses A/B test |
| `.agent/tasks/recon/todos.md` | Tâches par phase |
| `sites/bjemploi/RECON.md` | Recon BJemploi |
| `sites/job2mada/RECON.md` | Recon Job2mada |
| `sites/asako/RECON.md` | Recon Asako |
| `sites/wabajob/RECON.md` | Recon WabaJob |
| `infra/hermes/config.yaml` | Config globale Hermes |
| `infra/hermes/profiles.yaml` | 4 profils (kimi, glm, minimax, deepseek) |
| `skills/job-posting/SKILL.md` | Skill principal |
| `skills/job-posting/bjemploi.md` | Procédure posting BJemploi |
| `skills/job-posting/job2mada.md` | Procédure posting Job2mada |
| `skills/job-posting/asako.md` | Procédure posting Asako |
| `skills/job-posting/wabajob.md` | Procédure posting WabaJob |

---

## 8. Prochaines étapes identifiées

### Immédiat (utilisateur)
1. Créer le `.env` manuellement : `echo 'OLLAMA_CLOUD_API_KEY=ta-key' > .env`
2. Compléter `job.md` → remplacer tous les `[À RENSEIGNER]`
3. Créer les comptes sur les 4 sites (URLs dans job.md)

### Intervention manuelle requise
| Site | Étape | Action |
|---|---|---|
| BJemploi | Post-inscription | ⏳ Attendre validation admin (1-48h) |
| WabaJob | Étape 4 inscription | ✋ Cliquer lien confirmation email |
| Tous | Post-inscription | 🔄 Exporter cookies JSON (Playwright snippet) |

### Après création des comptes
1. Provisionner VPS Contabo dédié
2. Installer Docker + Hermes Agent
3. Créer skill job-posting définitif
4. Tester en one-shot avec job.md
5. A/B comparer les 4 agents

---

## 9. Deuxième partie de session — Délégation, déploiement, Telegram

### 9.1. Demande de délégation de développements

L'utilisateur a demandé : "Si je souhaite déléguer des développements à
cette équipe ; où puis je les trouver et comment puis je communiquer et
leur déléguer des implémentations ?"

**Raisonnement** :
- Hermes a un outil `delegate_task` (subagents isolés) + gateway Telegram
  + cron jobs → l'équipe peut recevoir du dev
- 3 canaux identifiés : Telegram (mobile, recommandé), GitHub Issues
  (structuré), CLI Hermes (direct)
- Ajout de 2 rôles : dev-bot (kimi-k2.7-code, TDD) + lead-dev-bot
  (deepseek-v4-pro, review)
- Skill `dev-tdd` (tests d'abord, code ensuite, PR) + `lead-review`
  (checklist, approve/request_changes)

### 9.2. Offre Telegram de l'utilisateur

L'utilisateur : "J'ai un compte télégram si tu veux"

**Questions posées** :
- Q1 : Bot déjà créé via @BotFather ? → "Bot à créer"
- Q2 : VPS existant ? → "VPS existant"

**Raisonnement** :
- Le VPS Contabo (100.98.194.18) est accessible via SSH (user yohann)
- Hermes v0.19.0 déjà installé sur le VPS (pip global)
- Ollama Cloud API key trouvée dans `/Factory/sellkit/.env`
- Pas besoin de nouveau VPS — utiliser l'existant

### 9.3. Déploiement en autonomie

L'utilisateur a demandé de chercher les credentials dans les sous-projets
de /Factory et de tout faire en autonomie.

**Étapes effectuées** :
1. Recherche `.env` dans `/Factory/` → trouvé OLLAMA_CLOUD_API_KEY dans
   `sellkit/.env`
2. Recherche VPS SSH → trouvé `VM_SSH_HOST=100.98.194.18` et
   `VM_SSH_USER=yohann` dans `sellkit/.env`
3. Test SSH → VPS accessible (`hostname: ai-sales-vm`)
4. Hermes v0.19.0 confirmé installé (`/usr/local/bin/hermes`)
5. Configuration Hermes (`hermes config set model.provider custom`,
   `model.base_url https://ollama.com/v1`, `model.default glm-5.2`,
   `model.api_key f3bf...`)
6. Test "Hermes online" → ✅ réponse reçue
7. Création repo GitHub `YoLoADR/ai-hirekit` (le owner GitHub est
   `YoLoADR`, pas `yohannravino` — découvert via `gh api user`)
8. Push du code vers le repo
9. Exécution `setup-github.sh` → 15 labels créés
10. Création 4 issues recon (#1-#4) + 1 issue dev (#5)
11. Installation des 6 skills sur le VPS
12. Démarrage gateway Hermes (systemd user service)
13. Création 3 cron jobs (recon, dev, review — every 30m)

### 9.4. Création du bot Telegram via Playwright

L'utilisateur s'est connecté à Telegram Web (web.telegram.org/k/) et m'a
donné la main.

**Étapes effectuées via Playwright** :
1. Navigation vers https://web.telegram.org/k/
2. Recherche "@BotFather" dans la barre de recherche
3. Clic sur BotFather dans les résultats → conversation ouverte
4. Clic sur bouton "START"
5. Envoi `/newbot` via le contenteditable input
6. BotFather demande le nom → réponse: "ai-hirekit bot"
7. BotFather demande le username → réponse: "loop_engineering_team_bot"
8. BotFather retourne le token : `8705472898:AAHBC3KCff188YRewHDYOLDGq3FyBja-lc8`

**Note** : Le navigateur Playwright a crashé plusieurs fois sur Telegram
Web K (SPA trop lourd). L'utilisateur a fourni le playbook Playwright
exact à suivre. Le token a été récupéré.

### 9.5. Récupération du User ID Telegram

Via @userinfobot (Playwright) :
- User ID : `7211240214`
- First : Loïc
- Last : Philogène

### 9.6. Configuration Telegram sur le VPS

**Problèmes rencontrés** :
1. Service systemd user ne charge pas les env vars → création
   `~/.config/environment.d/ai-hirekit.conf`
2. Dépendances Telegram manquantes → `pip install python-telegram-bot`
3. Conflit de polling Telegram → "terminated by other getUpdates request"
   → un service root `hermes-gateway-telegram-gateway` tournait déjà

**Découverte clé** : Un script `/opt/telegram-bot.py` existait déjà sur
Contabo, créé par une session précédente. Ce script :
- Route les messages Telegram vers VM 102 (Precision → Proxmox → CT 102)
- Utilise le même token `8705472898:AAH...`
- A 3 bots (po, dev, lead) qui travaillent sur `YoLoADR/ai-team`
- Service systemd : `loop-engineering-bot.service` (actif)

**Le bot `@loop_engineering_team_bot` était déjà en ligne.**

### 9.7. Audit complet de l'architecture existante

**Découvertes** :
- **Contabo** (100.98.194.18) : gateway Telegram uniquement
  (`/opt/telegram-bot.py` → service `loop-engineering-bot.service`)
- **Precision** (100.111.21.3) : Proxmox avec 2 CTs
  - CT 101 `openhands-vm` (autre projet)
  - CT 102 `ai-agents-vm` (agents Hermes, user `hermes`)
- **VM 102** : 3 profils Hermes actifs
  - po-bot (minimax-m3) → cwd `/home/hermes/repo/` → `YoLoADR/ai-team`
  - dev-bot (kimi-k2.7-code) → cwd `/home/hermes/repo/` → `YoLoADR/ai-team`
  - lead-dev-bot (deepseek-v4-pro) → cwd `/home/hermes/repo/` → `YoLoADR/ai-team`
- **ai-hirekit déjà cloné** sur VM 102 : `/home/hermes/projects/ai-hirekit/`
  (remote: `YoLoADR/ai-hirekit`, 2 commits)

**Raisonnement pour l'isolation** :
- Les bots ai-team ont `cwd: /home/hermes/repo/` (repo ai-team)
- Les bots ai-hirekit auront `cwd: /home/hermes/projects/ai-hirekit/`
- Repos GitHub séparés, chemins séparés, profils séparés
- Zéro interférence

### 9.8. Documentation de l'architecture

L'utilisateur s'est plaint de confusion sur l'emplacement de l'équipe.
J'ai créé :
- `README.md` mis à jour avec cartographie exacte (3 machines, isolation)
- `docs/ARCHITECTURE.md` avec diagrammes de flux complets
- `.agent/tasks/recon/ARCHITECTURE.md` version courte dans la tâche

**Nettoyage effectué** :
- Service user `hermes-gateway` désactivé sur Contabo (inactif, source de
  confusion)
- Service system `hermes-gateway-telegram-gateway` identifié comme échec
  (conflit avec `loop-engineering-bot.service`)

### 9.9. Demande de raisonnement

L'utilisateur a demandé : "N'oublie pas dans le readme et les autres
documents le raisonnement qui à mener aux choix"

**Raisonnement documenté** :
- D1 : Hermes > OpenHands (browser toolset Playwright)
- D2 : 4 modèles A/B test (pas de retour d'expérience)
- D3 : Cookies pré-auth (contournent captchas + validation admin)
- D4 : VPS Contabo dédié → en réalité VM 102 existante
- D5 : Template -kit (reproductible)
- D6 : 3 rôles d'agents (recon/poster/review) + 2 pour le dev
- D7 : Telegram (mobilité, gateway natif, notifications)
- D8 : GitHub Issues/PRs (stateless Hermes, persistance, Kanban)
- D9 : Pas de Docker Compose (agents natifs sur VM 102, pas en conteneurs)

### 9.10. État final du déploiement

| Composant | Statut | Où |
|---|---|---|
| Repo GitHub `YoLoADR/ai-hirekit` | ✅ | GitHub |
| Code sur VM 102 | ✅ | `/home/hermes/projects/ai-hirekit/` |
| Bot Telegram `@loop_engineering_team_bot` | ✅ actif | Contabo |
| Bots ai-team (po/dev/lead) | ✅ actifs | VM 102 `~/.hermes/profiles/` |
| Bots ai-hirekit (hirekit-*) | ⏳ à créer | VM 102 |
| Skills ai-hirekit sur VM 102 | ⏳ à déployer | VM 102 |
| Comptes recruteurs | ⏳ à créer | Manuel (Yohann) |
| Cron jobs (recon/dev/review) | ✅ créés | VPS Contabo (Hermes user) |
| Issues GitHub (#1-#5) | ✅ créées | GitHub `YoLoADR/ai-hirekit` |
| Labels (15) | ✅ créés | GitHub |