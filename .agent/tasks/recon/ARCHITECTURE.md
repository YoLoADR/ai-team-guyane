# Architecture — Où est l'équipe, où est le code

## Réponse courte

| Question | Réponse |
|---|---|
| Où est l'équipe de bots ? | **VM 102** (`ai-agents-vm`) sur Precision (Proxmox) |
| Où est le code de ai-hirekit local ? | **VM 102** `/home/hermes/projects/ai-hirekit/` |
| Où sur GitHub ? | **`YoLoADR/ai-hirekit`** |
| Le bot Telegram ? | **Contabo** (`100.98.194.18`) — `/opt/telegram-bot.py` |
| Est-ce mélangé avec ai-team ? | **NON** — repos séparés, chemins séparés, profils séparés |
| Est-ce mélangé avec d'autres projets ? | **NON** — ai-hirekit est isolé |

## Les 3 machines

```
Contabo (100.98.194.18)          Precision (100.111.21.3)         VM 102 (ai-agents-vm)
┌────────────────────┐           ┌────────────────────┐           ┌────────────────────────┐
│ ai-sales-vm        │           │ precision          │           │ ai-agents-vm            │
│                    │           │                    │           │ user: hermes            │
│ /opt/telegram-bot  │──SSH──→   │ Proxmox            │──pct──→   │                         │
│ .py                │           │ CT 101: openhands  │   102     │ ~/.hermes/profiles/     │
│                    │           │ CT 102: ai-agents  │           │   po-bot/ (ai-team)     │
│ Service:           │           │                    │           │   dev-bot/ (ai-team)    │
│ loop-engineering   │           │ (hyperviseur,      │           │   lead-dev-bot/ (team)  │
│ -bot.service       │           │  aucun agent ici)  │           │                         │
│                    │           │                    │           │ /home/hermes/repo/      │
│ Token Telegram     │           │                    │           │   → YoLoADR/ai-team     │
│ 8705...            │           │                    │           │                         │
│                    │           │                    │           │ /home/hermes/projects/  │
│ (PAS d'agents      │           │                    │           │   ai-hirekit/           │
│  Hermes ici)       │           │                    │           │   → YoLoADR/ai-hirekit  │
└────────────────────┘           └────────────────────┘           └────────────────────────┘
```

## Isolation projets

```
VM 102 (user hermes)
│
├── /home/hermes/repo/                    ← PROJET ai-team
│   └── .git → origin: YoLoADR/ai-team     (ne touche PAS ai-hirekit)
│
└── /home/hermes/projects/ai-hirekit/     ← PROJET ai-hirekit
    └── .git → origin: YoLoADR/ai-hirekit  (ne touche PAS ai-team)
```

### Profils Hermes actuels

| Profil | cwd | Projet | Statut |
|---|---|---|---|
| po-bot | `/home/hermes/repo/` | ai-team | ✅ actif |
| dev-bot | `/home/hermes/repo/` | ai-team | ✅ actif |
| lead-dev-bot | `/home/hermes/repo/` | ai-team | ✅ actif |
| hirekit-* (×5) | `/home/hermes/projects/ai-hirekit/` | ai-hirekit | ⏳ à créer |

## Ce qui reste à faire pour ai-hirekit

1. **Créer 5 profils Hermes sur VM 102** (hirekit-recon, poster, review, dev, leaddev)
   avec `cwd: /home/hermes/projects/ai-hirekit/`
2. **Déployer les skills** ai-hirekit sur VM 102 (`cp skills/* ~/.hermes/skills/`)
3. **Mettre à jour telegram-bot.py** (Contabo) pour les commandes `/hirekit`
4. **Créer les comptes recruteurs** (manuel, par Yohann)
5. **Tester** en envoyant `/hirekit dev <tâche>` au bot Telegram