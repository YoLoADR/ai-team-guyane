# Contexte — ai-hirekit

## Objectif

Poster automatiquement une offre d'emploi sur 4 sites de recrutement
(BJemploi.com, job2mada.com, asako.mg, wabajob.com) en utilisant des agents
Hermes avec browser automation (Playwright), avec des configurations
différentes pour identifier laquelle réussit à poster de façon autonome.

## Périmètre

| Inclus | Exclu |
|---|---|
| Posting sur 4 sites via browser automation | API propriétaire (pas d'API publique) |
| Sessions pré-authentifiées (cookies) | Création de comptes par l'agent (captchas) |
| 4 profils Hermes (1 par site, modèle différent) | Multi-offres simultanées (v1 = 1 offre) |
| Skill custom job-posting | Cron/récurrence (v1 = one-shot) |
| A/B test des modèles Ollama Cloud | Monitoring temps réel |
| Template reproductible (autres pays/langues) | |

## Stack technique

| Couche | Choix |
|---|---|
| Agent | Hermes Agent (Nous Research) |
| Browser | Playwright (intégré Hermes, Docker shm-size=1g) |
| LLMs | Ollama Cloud (4 modèles testés en A/B) |
| Hébergement | VPS Contabo dédié (Docker) |
| Auth | Sessions pré-authentifiées (cookies JSON) |
| Suivi | Pattern Merenza (.agent/tasks/) |

## Modèles Ollama Cloud retenus (A/B test)

| Profil | Modèle | Justification |
|---|---|---|
| bjemploi-poster | kimi-k2.7-code | Précis sur formulaires structurés |
| job2mada-poster | glm-5.2 | Modèle alternatif polyvalent |
| asako-poster | minimax-m3 | Grand contexte (longs formulaires IA) |
| wabajob-poster | deepseek-v4-pro | Bon raisonnement multi-étapes |

## Sites cibles

| Site | Pays | Captcha | Validation admin | Multi-étapes |
|---|---|---|---|---|
| BJemploi.com | Bénin | ❌ | ✋ Oui (1-48h) | Non |
| Job2mada.com | Madagascar | ❌ | Non visible | Non |
| Asako.mg | Madagascar | ❌ | Non visible | Non |
| WabaJob.com | Bénin | ❌ | Non visible | Oui (4 étapes) |

## Workflow

```
1. Opérateur crée les comptes recruteurs manuellement (captchas éventuels)
   ↓
2. Opérateur exporte les cookies de session (Playwright code snippet)
   ↓
3. Agent Hermes charge les cookies + navigue sur le site
   ↓
4. Agent remplit le formulaire de publication d'offre avec le contenu de job.md
   ↓
5. Agent soumet le formulaire
   ↓
6. Vérification : l'offre est-elle visible sur le site ?
```

## Règles dures

- 🔒 Credentials (emails, mots de passe, cookies) → jamais commités dans git
- 🔒 Cookies de session stockés dans `sites/<site>/cookies.json` (gitignored)
- ✅ Sessions pré-authentifiées uniquement (l'agent ne crée pas de comptes)
- ✅ Un profil Hermes par site (isolation totale)
- 🚫 Pas de Claude/GPT : uniquement modèles Ollama Cloud
- 🚫 Pas de service de résolution de captcha (v1)

## Raisonnement et décisions

### Préambule — Modèles ayant construit cette équipe

Cette équipe et toute la documentation du projet ai-hirekit ont été
**construites avec Kimi (kimi-k2.7-code) et GLM (glm-5.2)** via Ollama Cloud,
lors d'une session unique le 2026-08-03 :
- **GLM-5.2** : analyse, recon des 4 sites, rédaction de la documentation,
  routing Telegram, configuration Hermes
- **Kimi-k2.7-code** : code, profils Hermes, docker-compose, scripts,
  structuration des skills

Les 2 autres modèles (minimax-m3, deepseek-v4-pro) sont assignés à des
rôles spécifiques (Asako, WabaJob) pour l'A/B test mais n'ont pas
participé à la construction de l'équipe.

### D1 — Pourquoi Hermes Agent et pas OpenHands ?

**Contexte** : Le projet ai-team avait testé deux approches pour le loop
engineering :
- v1 : Hermes Agent (3 profils, minimax/kimi/deepseek)
- v2 : OpenHands (3 profils, glm/kimi/qwen) — choisi pour ai-team car
  support natif multi-agent avec workspaces Docker isolés

**Raisonnement** : Pour le posting d'offres d'emploi, le besoin est
différent de ai-team :
- C'est une tâche **mono-agent** (1 agent poste sur 1 site à la fois),
  pas un workflow PO→Dev→LeadDev
- Hermes a un **browser toolset intégré** (Playwright) — c'est le critère
  décisif : OpenHands n'a pas de browser automation natif
- Hermes propose **browse.sh** (200+ skills pré-faits pour automatiser des
  sites spécifiques)
- Hermes a des **cron jobs** intégrés pour la récurrence
- Les skills custom (`SKILL.md`) permettent de documenter les formulaires
  de posting par site

**Décision** : Hermes Agent, pas OpenHands.

### D2 — Pourquoi 4 modèles LLM différents (A/B test) ?

**Raisonnement** : Aucun retour d'expérience n'existe sur l'automatisation
de posting d'offres d'emploi par des LLM sur ces sites malgaches/béninois.
Plutôt que de parier sur un seul modèle, tester 4 modèles en parallèle
permet de :
- Identifier quel modèle gère le mieux les formulaires (précision vs
  raisonnement)
- Comparer la robustesse face aux SPA React (hydration, champs dynamiques)
- Mesurer le coût réel en tokens par posting
- Diversifier les risques (un modèle peut être indisponible)

**Choix des modèles** (basé sur les profils existants dans ai-team) :
| Modèle | Hypothèse | Site assigné |
|---|---|---|
| kimi-k2.7-code | Spécialisé code → précis sur formulaires structurés | BJemploi (PHP classique) |
| glm-5.2 | Polyvalent → bon sur SPA React standard | Job2mada (SPA simple) |
| minimax-m3 | Grand contexte (16384 tokens) → longs formulaires IA | Asako (IA génération offre) |
| deepseek-v4-pro | Bon raisonnement → gère les multi-étapes | WabaJob (4 étapes) |

### D3 — Pourquoi sessions pré-authentifiées (cookies) et pas création de comptes par l'agent ?

**Raisonnement** :
- Les 4 sites nécessitent des comptes recruteurs
- BJemploi a une **validation admin manuelle** (1-48h) — l'agent ne peut
  pas attendre
- WabaJob a une **vérification email** (étape 4) — nécessite intervention
  humaine
- Des captchas invisibles (iframe sur BJemploi) peuvent bloquer l'agent
- Les cookies de session contournent tout ça — l'agent démarre déjà
  connecté

**Décision** : L'opérateur (Yohann) crée les comptes manuellement, exporte
les cookies via Playwright, l'agent utilise les cookies pour les sessions.

### D4 — Pourquoi un VPS Contabo dédié isolé ?

**Raisonnement** :
- Le VPS Contabo (`ai-sales-vm`, 100.98.194.18) héberge déjà d'autres
  services (ai-sales, tgc-rag, squad, maya)
- Isoler ai-hirekit évite les conflits de ressources (Playwright/Chrome
  est gourmand en mémoire, `shm-size=1g`)
- L'isolation évite que les cron jobs d'ai-hirekit interfèrent avec les
  autres processus

**Décision finale** : En réalité, après audit, les agents Hermes tournent
déjà sur **VM 102 (Precision/Proxmox)** — pas sur Contabo. Contabo ne sert
que de gateway Telegram. Voir `ARCHITECTURE.md` pour le détail.

### D5 — Pourquoi un template reproductible (-kit) ?

**Raisonnement** : Le besoin initial (poster sur BJemploi, Job2mada, Asako,
WabaJob) est un cas spécifique, mais le pattern est reproductible :
- Autres pays : mêmes sites avec d'autres domaines (CIemploi, SNemploi…)
- Autres domaines : pas que l'emploi (immobilier, annonces, marketplace)
- Le suffixe "-kit" signale un template, pas un projet one-shot

**Décision** : Nommer le projet `ai-hirekit` (hire + kit), structuré pour
être copié-collé et adapté.

### D6 — Pourquoi 3 rôles d'agents (recon, poster, review) en plus des 4 posters ?

**Raisonnement** : Le loop engineering de ai-team (PO→Dev→LeadDev) a
montré que la séparation des rôles améliore la qualité :
- **recon-bot** : Analyse les formulaires avant posting (évite les erreurs
  de champs manquants)
- **poster-bot** : Poste l'offre (4 variantes pour A/B test)
- **review-bot** : Vérifie que l'offre est visible et correcte (attrape
  les erreurs d'encodage, champs vides, offres non publiées)

Ajoutés ensuite pour la délégation de dev :
- **dev-bot** : Implémente du code (TDD) pour des évolutions du projet
- **lead-dev-bot** : Review le code des PRs

### D7 — Pourquoi Telegram comme interface principale ?

**Raisonnement** :
- Hermes a un **gateway Telegram natif** (plus stable que Discord/Slack
  pour un usage personnel)
- Telegram permet la **mobilité** (commander depuis le téléphone)
- Le bot peut **push des notifications** (PR mergée, offre publiée, erreur)
- Un script Python (`/opt/telegram-bot.py`) route les commandes vers
  les bots sur VM 102 via SSH
- L'authentification par user ID (7211240214) sécurise l'accès

### D8 — Pourquoi GitHub Issues + PRs comme orchestrateur ?

**Raisonnement** :
- Les agents Hermes sont **stateless** entre les sessions — GitHub
  persiste l'état (issues = tâches, PRs = résultats)
- Les cron jobs d'Hermes (every 30m) lissent les issues → pas besoin
  de webhook temps réel
- Le Kanban Project V2 donne une **vue d'ensemble visuelle** du pipeline
- Les labels permettent le **filtrage par site, modèle, phase, statut**
- Les PRs avec template (preuves de posting, screenshots) donnent un
  audit trail
- L'automatisation Kanban (`.github/workflows/kanban.yml`) déplace les
  issues automatiquement sur les events PR/label

### D9 — Pourquoi ne pas déployer Docker Compose (9 conteneurs) ?

**Raisonnement initial** : Le `docker-compose.yml` prévoit 9 conteneurs
(4 posters + recon + review + dev + leaddev + telegram).

**Décision finale** : En réalité, après audit de l'infra existante :
- Les agents Hermes tournent déjà sur VM 102 **sans Docker** (install
  pip global, profils dans `~/.hermes/profiles/`)
- Le gateway Telegram tourne sur Contabo en **service systemd** (pas
  Docker)
- Docker Compose reste documenté dans le repo pour référence, mais
  l'exécution réelle utilise les profils Hermes natifs + cron jobs

## Fichiers liés

- Offre + comptes : `job.md` (racine)
- Recon détaillée : `.agent/tasks/recon/SESSION_NOTES.md`
- Recon par site : `sites/<site>/RECON.md`
- Plan infra : `.agent/tasks/recon/PLAN.md`
- Config Hermes : `infra/hermes/config.yaml`
- Profils : `infra/hermes/profiles/*.yaml`
- Skill : `skills/job-posting/SKILL.md`
- Projet d'origine : `/Users/YoLoADR/Factory/ai-team/job.md`