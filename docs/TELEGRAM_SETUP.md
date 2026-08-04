# Setup Telegram Bot — Équipe Guyane 🇬🇫 (ai-team-guyane)

> **Mis à jour** le 2026-08-04 — le bot Telegram est centralisé dans
> `ai-team-cuba/telegram-bot.py` et déployé sur Contabo. Les commandes
> Guyane sont préfixées `/guyane_*`.

## 1. Créer le bot via @BotFather (2 min)

1. Ouvrir Telegram → chercher **@BotFather**
2. Envoyer `/newbot`
3. BotFather demande un nom → `ai-hirekit bot`
4. BotFather demande un username (unique, doit finir par `bot`) → `ai_hirekit_bot`
5. BotFather répond avec :
   ```
   Done! Congratulations on your new bot. ...
   Use this token to access the HTTP API:
   1234567890:ABCdefGHIjklMNOpqrsTUVwxyz
   ```
6. **Copier le token** (ne pas le partager)

## 2. Récupérer ton Telegram User ID

Pour que seul toi puisses commander le bot :

1. Ouvrir Telegram → chercher **@userinfobot**
2. Envoyer `/start`
3. Le bot répond avec :
   ```
   Id: 123456789
   First: Yohann
   ...
   ```
4. **Copier ton User ID**

## 3. Configurer sur le VPS Contabo

```bash
# SSH sur le VPS
ssh user@<vps-contabo-ip>

# Cloner le repo
git clone git@github.com:YoLoADR/ai-hirekit.git
cd ai-hirekit

# Créer le .env (manuellement — garde-fou sécurité)
cat > .env << 'EOF'
# Ollama Cloud
OLLAMA_CLOUD_API_KEY=ta-key-ollama

# Telegram
TELEGRAM_BOT_TOKEN=1234567890:ABCdefGHIjklMNOpqrsTUVwxyz
TELEGRAM_ALLOWED_USER_IDS=ton-user-id

# GitHub tokens machine
GH_TOKEN_RECON=ghp_xxx
GH_TOKEN_POSTER=ghp_xxx
GH_TOKEN_REVIEW=ghp_xxx
GH_TOKEN_DEV=ghp_xxx
GH_TOKEN_LEADDEV=ghp_xxx
GH_TOKEN_GATEWAY=ghp_xxx
EOF

# Lancer le setup Hermes
hermes gateway setup
# → Choisir Telegram
# → Le token est lu depuis .env automatiquement
```

## 4. Démarrer le gateway

### Option A — Docker Compose (recommandé)

```bash
docker-compose up -d hermes-telegram
```

### Option B — Manuel

```bash
hermes gateway run
# → Le bot écoute les messages Telegram
# → Tu peux chatter avec lui directement
```

## 5. Tester

Ouvre Telegram → trouve `@ai_hirekit_bot` → envoie :

```
/help
```

Le bot doit répondre avec la liste des commandes.

```
/status
```

Le bot doit afficher le statut des issues/PRs.

```
/dev Crée une fonction isEmail dans lib/validators.ts avec tests Vitest. TDD strict.
```

Le bot crée une issue GitHub `dev-task` et confirme.

## 6. Sécurité

- Le bot ne répond qu'aux users dont l'ID est dans `TELEGRAM_ALLOWED_USER_IDS`
- Le token Telegram n'est jamais commité (`.env` dans `.gitignore`)
- Les tokens GitHub sont séparés par rôle (principe du moindre privilège)

## 7. Commands disponibles

| Commande | Action |
|---|---|
| `/dev <description>` | Crée issue dev-task → dev-bot implémente |
| `/recon <url>` | Crée issue recon → recon-bot analyse |
| `/post <site>` | Crée issue posting → poster-bot poste |
| `/status` | Statut des issues/PRs en cours |
| `/review` | Force review sur les PRs ouvertes |
| `/skills` | Liste les skills |
| `/cron list` | Liste les cron jobs |
| `/help` | Aide |