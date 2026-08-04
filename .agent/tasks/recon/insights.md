# Insights — ai-hirekit

## 2026-08-03 — Session initiale

### Découverte
- Les 4 sites n'ont **aucun captcha visible** sur leurs formulaires
  d'inscription recruteur. C'est une bonne surprise — la création de compte
  manuelle sera simple.
- BJemploi.com est un site PHP classique (tables HTML, iframes) — probablement
  le plus facile à automatiser pour un agent browser, car la structure est
  simple et statique.
- Asako.mg est le plus moderne (SPA React) et propose une **IA de génération
  d'offre** intégrée — l'agent Hermes pourrait potentiellement l'utiliser pour
  gagner du temps.
- WabaJob.com a une inscription en **4 étapes** — l'agent devra gérer les
  transitions entre étapes, ce qui teste la robustesse du modèle.

### Hypothèses A/B test
| Modèle | Hypothèse | Risque |
|---|---|---|
| kimi-k2.7-code | Bon pour formulaires structurés (BJemploi PHP) | Peut manquer de contexte sur les SPA |
| glm-5.2 | Polyvalent, bon sur job2mada | Moins précis que kimi sur les forms |
| minimax-m3 | Grand contexte pour Asako (longs formulaires IA) | Peut être trop créatif |
| deepseek-v4-pro | Bon raisonnement pour WabaJob (4 étapes) | Peut sur-analyser |

### Points d'attention
- BJemploi nécessite une **validation admin manuelle** après inscription.
  Il faut créer le compte le plus tôt possible.
- WabaJob a une **étape 4 de vérification** (probablement email).
  Intervention manuelle à prévoir.
- Les cookies de session devront être exportés après création des comptes.
  Playwright peut le faire via `page.context().cookies()`.
- Aucun des 4 sites n'expose d'API publique pour le posting d'offres.
  Browser automation est la seule voie.

### À surveiller
- reCAPTCHA invisible (iframe sur BJemploi connexion) — si présent, les
  sessions pré-auth contournent le problème.
- Changement de structure des formulaires — les SPAs (job2mada, asako,
  wabajob) peuvent changer entre la recon et le test réel.
- Expiration des cookies — durée de vie inconnue, prévoir re-login.