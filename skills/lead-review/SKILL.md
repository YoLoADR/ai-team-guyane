---
name: lead-review
description: Lead Developer code review — checklist, approve or request changes
---
# Lead Dev Review Workflow

## Procédure
1. Lire les PRs ouvertes : `gh pr list --state open`
2. Pour chaque PR :
   - `gh pr diff <pr>` → analyser le code
   - `gh pr view <pr> --json body` → lire le contexte
   - Vérifier l'issue liée (Closes #XX)

## Checklist de review

### 1. Conventions
- [ ] TypeScript strict (pas de `any`, types explicites)
- [ ] Next.js App Router respecté (route handlers, server components)
- [ ] Nommage cohérent (camelCase fonctions, PascalCase composants)
- [ ] Pas de console.log en production

### 2. Tests (TDD)
- [ ] Tests présents pour les nouveaux composants/routes
- [ ] Tests écrits AVANT le code (vérifier l'ordre des commits)
- [ ] Couverture suffisante (cas nominaux + edge cases)
- [ ] Tests passent : `npm run test -- --run`

### 3. CI
- [ ] `npm run lint` passe
- [ ] `npx tsc --noEmit` passe
- [ ] `npm run test -- --run` passe

### 4. Sécurité
- [ ] Pas de secrets/credentials dans le code
- [ ] Validation des inputs (Zod côté API)
- [ ] Pas d'injection SQL (utiliser Drizzle query builder)
- [ ] Pas de XSS (échapper les sorties)

### 5. Spec
- [ ] Critères d'acceptation de l'issue vérifiés
- [ ] Pas de feature non demandée (scope creep)

### 6. Performance
- [ ] Pas de N+1 queries
- [ ] Pas de gros bundles inutiles
- [ ] Pas de re-renders inutiles (React.memo, useCallback si besoin)

## Décision

### SI OK (tous les critères validés)
```bash
gh pr review <pr> --approve \
  --body "✅ Code review approuvé. Tests, CI, sécurité et spec tous conformes."
gh pr merge <pr> --squash
gh issue close <issue> --comment "Implémenté et mergé dans la PR #<pr>."
```
→ Label "success" sur l'issue

### SI KO (problème détecté)
```bash
gh pr review <pr> --request-changes \
  --body "
## ❌ Problèmes détectés

1. **<file>:<line>** : <problème> → <suggestion>
2. **<file>:<line>** : <problème> → <suggestion>

## Actions requises
- Corriger les points ci-dessus
- Relancer les tests
- Re-pousser sur la même branche
"
```
→ Label "needs-work" sur l'issue

## Format des commentaires
Toujours : `file:line → problème → suggestion`

Exemple :
```
src/app/api/tasks/route.ts:42 → pas de validation Zod sur le body → ajouter taskInsertSchema.safeParse(body)
tests/api/tasks.test.ts:15 → test manquant pour le cas 400 → ajouter test avec body invalide
```