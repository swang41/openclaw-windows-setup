# OpenClaw on Windows 10 — Initial Setup Guide  
Docker Desktop + WSL2 + Telegram + Anthropic (Claude 4.5 Series)

This guide documents a simple, working setup for running OpenClaw locally on a Windows 10 desktop using Docker and WSL2.  
I created this mainly to help others who want to experiment with OpenClaw before deciding whether to invest in a Mac mini or other dedicated hardware.

---

## 1. Prerequisites

Install:

- Docker Desktop
- WSL2 (Ubuntu recommended)
- Git
- Node.js 18+ (inside WSL)
- Telegram bot token (via @BotFather)
- Anthropic API key

Enable WSL integration in Docker Desktop:

```
Settings → Resources → WSL Integration → Enable Ubuntu
```

---

## 2. Clone OpenClaw (inside WSL)

```bash
git clone https://github.com/OpenClawAI/openclaw.git
cd openclaw
```

---

## 3. Create `.env`

Create a file named `.env` in the OpenClaw folder:

```
ANTHROPIC_API_KEY=your_key_here
TELEGRAM_BOT_TOKEN=your_telegram_bot_token
OPENCLAW_GATEWAY_TOKEN=any_random_string
```

---

## 4. Update `docker-compose.yml`

Make sure the gateway service includes the environment file:

```yaml
env_file: ./env
```

Example block:

```yaml
services:
  gateway:
    build:
      context: .
    image: openclaw:local
    env_file: ./env
    volumes:
      - ~/.openclaw:/home/node/.openclaw
    ports:
      - "18789:18789"
```

This ensures the container receives your API keys.

---

## 5. Build the OpenClaw Image

Run the correct docker compose build command:

```bash
docker compose build openclaw:local
```

---

## 6. Run the Onboarding UI  
(Required for Anthropic authentication)

```bash
docker compose run --rm opencli onboard
```

During onboarding:

1. Choose **Anthropic**
2. Enter your API key
3. Save and exit

This generates:

```
~/.openclaw/agents/main/agent/auth-profiles.json
```

OpenClaw will not load Anthropic models without this file.

---

## ⚠️ Anthropic Model Notes

Some Anthropic model names currently return **404 (model not found)** inside OpenClaw.

Models that usually fail:
- `claude-3-5-sonnet-20241022`
- `claude-3-opus`
- `claude-3-haiku`
- all Claude 2.x models

Working models:
- `anthropic/claude-haiku-4-5`
- `anthropic/claude-sonnet-4-5`

You can configure these later in `openclaw.json`.

---

## 7. Start the OpenClaw Gateway

```bash
docker compose up -d gateway
```

Check system status:

```bash
openclaw doctor
```

You should see valid configuration and no missing keys.

---

## 8. Configure Telegram Access

Find your Telegram numeric user ID (via `@userinfobot`).

Edit:

```
~/.openclaw/openclaw.json
```

Add or update:

```json
{
  "channels": {
    "telegram": {
      "enabled": true,
      "dmPolicy": "allowlist",
      "allowFrom": [YOUR_TELEGRAM_NUMERIC_ID],
      "groupPolicy": "disabled",
      "streamMode": "partial"
    }
  }
}
```

Restart the gateway:

```bash
docker restart openclaw-gateway
```

You should now be able to send a message to your Telegram bot and get a reply.

---

## 9. Minimal Agent Identity Prompt

Edit:

```
~/.openclaw/agents/main/agent/agent.json
```

Add a simple identity prompt:

```
You are a helpful AI assistant running locally via OpenClaw.
Use Anthropic models for reasoning.
Provide concise and structured responses.
```

This is enough for basic usage.

---

## 10. (Optional) Quiet-Hour Guardrails

Create:

```
~/.openclaw/agents/main/agent/agent-policy.json
```

Add:

```json
{
  "policies": {
    "message_limits": {
      "unsolicited_daily_max": 1,
      "quiet_hours": {
        "enabled": true,
        "start": "22:00",
        "end": "07:00",
        "timezone": "America/Los_Angeles"
      }
    }
  }
}
```

Restart:

```bash
docker restart openclaw-gateway
```

---

## ✔ Setup Complete

You now have a working OpenClaw installation on Windows 10 using Docker + WSL2, with Anthropic models and Telegram messaging enabled.

This guide is intended as a simple starting point for Windows users who want to try OpenClaw locally before investing in more dedicated hardware.
