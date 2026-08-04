---
name: dev-tdd
description: Developer TDD workflow — tests first, implement, PR
---
# Dev TDD Workflow

## Procédure TDD
1. Lire l'issue GitHub assignée + critères d'acceptation
2. Créer branche : `git checkout -b feature/<issue-number>-<slug>`
3. **RED** : Écrire les tests d'abord (tests/api/, tests/components/)
4. Run : `npm run test -- --run` → vérifier que les tests échouent (RED)
5. **GREEN** : Implémenter le code minimum pour faire passer les tests
6. Run : `npm run test -- --run` → vérifier que les tests passent (GREEN)
7. **REFACTOR** : Améliorer le code sans casser les tests
8. Run : `npm run typecheck && npm run lint && npm run test -- --run`
9. Si échec : corriger, relancer
10. `git add -A && git commit -m "feat: <description>"`
11. `git push origin feature/<issue-number>-<slug>`
12. `gh pr create --title "feat: <description>" --body "Closes #<issue>"`
13. Commenter sur l'issue : "PR #XX ouverte, en attente de review"

## Délégation (optionnelle)

Si la tâche se décompose en sous-tâches indépendantes :

```python
delegate_task(
    goal="Write tests for TodoForm component",
    context="Tests in tests/components/TodoForm.test.tsx. Use React Testing Library.",
    toolsets=['terminal', 'file']
)
```

## Stack
- Next.js 14+ App Router
- TypeScript strict
- Vitest + React Testing Library + MSW
- Drizzle ORM + Better-SQLite3
- Zod pour validation
- Tailwind CSS + shadcn/ui

## Règles
- TDD strict : tests écrits AVANT le code, jamais l'inverse
- Chaque PR doit passer la CI (lint + typecheck + test)
- Pas de secrets/credentials dans le code
- Commits conventionnels : feat:, fix:, refactor:, test:, docs:
- Une PR = une issue = une feature