# Moltbot-Setup-Guide-with-API-to-Save-You-Cost
Walks you through installing Moltbot (formerly Clawdbot), completing onboarding, and wiring it to use your own API service to save AI cost.

## Table of contents

- Overview
- Requirements
- Install
- First-time onboarding
- Configure the unified API gateway (CrazyRouter)
- Restart and verify
- Validation checks
- Common pitfalls
- FAQ
- Command cheat sheet
- File locations
- Security tips
- Summary

---

## Overview

Moltbot (formerly Clawdbot) is an open-source local AI assistant that works through messaging apps (Telegram, WhatsApp, Discord, and more) or a Web UI. You can run it locally and connect it to any compatible AI API provider by configuring the gateway correctly.

What this guide covers:
- Installing Node.js 22+ and Moltbot
- Onboarding and optional channel setup
- Configuring the API gateway with a custom base URL
- Validating the setup and troubleshooting common issues

---

## Requirements

Required:
- OS: macOS, Linux, or Windows
- Node.js: 22.0.0 or newer
- Package manager: pnpm (recommended) or npm

Optional:
- Xcode on macOS (if you build native apps)
- Messaging platform account (Telegram, Discord, etc.)

---

## Install

### 1) Install Node.js 22

If you use nvm:

```bash
nvm install 22
nvm use 22
nvm alias default 22
node --version
```

### 2) Install Moltbot

Option A: npm global install (recommended)

```bash
npm install -g moltbot@latest
```

Option B: one-line install script

```bash
curl -fsSL https://molt.bot/install.sh | bash
```

Option C: build from source

```bash
git clone https://github.com/moltbot/moltbot.git
cd moltbot
pnpm install
pnpm build
npm link
```

---

## First-time onboarding

Start the guided setup:

```bash
moltbot onboard
```

During onboarding:
- Confirm the security prompt.
- Choose the model provider and authentication method.
- Optionally set up a channel (Telegram example below).

### Auth method quick comparison

- setup-token: best for Claude Max or Pro users
- Claude Code CLI token: if you already use Claude Code locally
- API key: standard pay-as-you-go credentials

If you need a setup-token:

```bash
claude setup-token
```

Paste the token when prompted.

### Telegram channel (optional)

To create a Telegram bot token:
1) Chat with @BotFather
2) Run /newbot
3) Name the bot and copy the token

After onboarding, you will see the local Web UI URL and gateway address.

If you see a pairing code, approve it like this:

```bash
moltbot pairing approve telegram <CODE>
```

---

## Configure the unified API gateway (CrazyRouter)

Use the following settings for your unified AI API gateway:

- API base URL: https://crazyrouter.com/
- Sample API key: sk-xxx
- Recommended purchase link: https://crazyrouter.com/register?aff=WyfY (this link includes a 2 USD gift credit)
- Recommended third-party API: https://crazyrouter.com/

Important: Moltbot reads custom provider base URLs from the config file under models.providers (not from provider env vars). For anthropic-messages providers, the client appends /v1 automatically, so set baseUrl to the root endpoint (no /v1).

### 1) Back up the config file

```bash
cp ~/.clawdbot/moltbot.json ~/.clawdbot/moltbot.json.bak
```

### 2) Edit the config file

```bash
nano ~/.clawdbot/moltbot.json
```

Add or update the models.providers section:

```json
{
  "models": {
    "mode": "merge",
    "providers": {
      "anthropic": {
        "baseUrl": "https://crazyrouter.com/",
        "apiKey": "sk-xxx",
        "api": "anthropic-messages",
        "models": []
      }
    }
  }
}
```

Field notes:
- mode: use "merge" to keep the default catalog and add your provider
- baseUrl: your API gateway endpoint (for anthropic-messages, omit /v1)
- apiKey: your gateway key
- api: must be anthropic-messages
- models: must exist even if empty

### 3) Full example (optional)

```json
{
  "meta": {
    "lastTouchedVersion": "2026.1.25",
    "lastTouchedAt": "2026-01-27T01:05:21.233Z"
  },
  "models": {
    "mode": "merge",
    "providers": {
      "anthropic": {
        "baseUrl": "https://crazyrouter.com/",
        "apiKey": "sk-xxx",
        "api": "anthropic-messages",
        "models": []
      }
    }
  },
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-sonnet-4-5"
      },
      "workspace": "/Users/yourname/clawd",
      "maxConcurrent": 4
    }
  },
  "gateway": {
    "port": 18789,
    "mode": "local",
    "bind": "loopback",
    "auth": {
      "mode": "token",
      "token": "your_gateway_token"
    }
  },
  "channels": {
    "telegram": {
      "enabled": false
    }
  }
}
```

---

## Restart and verify

Restart the gateway:

```bash
moltbot gateway restart
```

Check status:

```bash
moltbot channels status
```

If everything is correct you should see that the gateway is reachable.

---

## Validation checks

1) Open the Web UI (from the onboarding output). Example:

```
http://127.0.0.1:18789/?token=your-token
```

2) Send a test message in Chat.
3) Confirm you receive a response and the status is healthy.

If you need logs:

```bash
moltbot logs --follow
```

---

## Common pitfalls

1) Using environment variables
- Problem: provider env vars do not change the gateway base URL
- Fix: update ~/.clawdbot/moltbot.json under models.providers

2) Missing the models field
- Problem: config validation fails
- Fix: include "models": [] in the provider config

3) Telegram connect failures
- Symptom: gateway keeps restarting
- Fix: disable telegram temporarily

```bash
moltbot config set channels.telegram.enabled false
moltbot gateway restart
```

4) Node.js too old
- Symptom: version error on startup
- Fix: upgrade to Node.js 22+

5) Forgot to restart
- Fix: run moltbot gateway restart after edits

---

## FAQ

Q: Web UI shows disconnected (1006)
A:
- Check the gateway process
- Check port conflicts
- Validate the config JSON
- Restart the gateway

Q: API requests fail
A:
- Verify baseUrl and apiKey in the config
- Confirm api is anthropic-messages
- Restart the gateway

Q: No response in chat
A:
- Confirm you edited the config file (not env vars)
- Restart the gateway
- Review logs for errors

Q: Where are logs stored
A:
- /tmp/moltbot/moltbot-YYYY-MM-DD.log
- /tmp/moltbot/moltbot-gateway.log (launchd/macOS gateway)
- Use moltbot logs --follow for streaming output

Q: How do I reset everything
A:
- Stop the gateway
- Back up and remove ~/.clawdbot
- Rerun moltbot onboard

---

## Command cheat sheet

Gateway:
```bash
moltbot channels status
moltbot status --deep
moltbot gateway restart
```

Config:
```bash
moltbot configure
moltbot config set gateway.mode local
moltbot config set channels.telegram.enabled false
cat ~/.clawdbot/moltbot.json
```

Logs:
```bash
moltbot logs --follow
```

Diagnostics:
```bash
moltbot doctor
moltbot doctor --fix
```

---

## File locations

```
~/.clawdbot/
├── moltbot.json
├── credentials/
├── sessions/
├── logs/
└── agents/

~/Library/LaunchAgents/
└── bot.molt.gateway.plist (legacy installs may show com.clawdbot.gateway.plist)

/tmp/moltbot/
├── moltbot-YYYY-MM-DD.log
└── moltbot-gateway.log
```

---

## Security tips

- Keep API keys out of git.
- Rotate API keys regularly.
- Do not share Web UI token URLs.
- Keep the gateway bound to localhost unless you protect it with VPN.
- Back up configs before major changes.

---

## Summary

You now have Moltbot installed, configured, and connected to a unified AI API gateway at https://crazyrouter.com/. The key steps are updating the models.providers section, keeping the required fields (baseUrl, apiKey, api, models), and restarting the gateway after any edit.

Recommended purchase link (2 USD gift credit): https://crazyrouter.com/register?aff=WyfY

