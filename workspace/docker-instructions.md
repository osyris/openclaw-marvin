# Dockerfile Instructions — OpenClaw "Marvin" (Railway)

## 1. Дополнительные пакеты

В Dockerfile добавить:
```dockerfile
RUN apt-get update && apt-get install -y openssh-client && rm -rf /var/lib/apt/lists/*
```
> Если chromium ещё не в образе — добавить и его: `openssh-client chromium`

## 2. Entrypoint

Перед запуском OpenClaw добавить:
```bash
[ -f /data/.ssh/setup.sh ] && /data/.ssh/setup.sh
```
Скрипт создаёт `/root/.ssh/config` (ссылка на deploy key в `/data/.ssh/marvin_deploy`) и настраивает git identity (Marvin, marvin@konzhi.com).

## 3. Переменные окружения (Railway)

Оставить ТОЛЬКО:
```
OPENCLAW_STATE_DIR=/data/.openclaw
OPENCLAW_WORKSPACE_DIR=/data/workspace
RAILWAY_RUN_UID=0
```

**Удалить из env vars:** ANTHROPIC_API_KEY, TELEGRAM_BOT_TOKEN, DEEPGRAM_API_KEY, OPENCLAW_GATEWAY_TOKEN, AUTH_SECRET, PROXY_USER, PROXY_PASSWORD, SETUP_PASSWORD — они уже в openclaw.json на persistent volume.

## 4. Git repo

- Remote: `git@github.com:osyris/marvin.git`
- Branch: `main`
- Git init в `/data/` (корень persistent volume)
- Deploy key (ED25519, read+write) уже добавлен в GitHub и лежит в `/data/.ssh/marvin_deploy`

## 5. Persistent volume `/data`

```
/data/
├── .ssh/marvin_deploy          # SSH private key (НЕ в git)
├── .ssh/marvin_deploy.pub      # SSH public key
├── .ssh/setup.sh               # Startup script
├── .openclaw/openclaw.json     # Конфиг с секретами (НЕ в git)
├── .openclaw/openclaw.sanitized.json  # Конфиг без секретов (в git)
├── .git/                       # Git repo
├── .gitignore
└── workspace/                  # Рабочая директория агента
```

## 6. Автобэкап

Cron внутри OpenClaw: каждый день 03:00 Europe/Tallinn → `git add -A && git commit && git push`. Sanitized конфиг обновляется перед коммитом.
