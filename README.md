# Deploy and Host Sim on Railway

[![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/deploy/sim-production?referralCode=sLjDXb&utm_medium=integration&utm_source=button&utm_campaign=sim)

[Sim](https://sim.ai) is an open-source platform for building and running AI agent workflows. You lay out agents, tools, conditions, loops and code on a canvas, connect any model provider and hundreds of integrations (Slack, Gmail, GitHub, Notion, databases and more), then run each workflow on a schedule, from a webhook, as an API, or as a chat. It is Apache-2.0 licensed and one of the most-starred agent builders on GitHub.

## About Hosting Sim

This template runs the full Sim stack, version 0.9.26, as six Railway services from Sim's official images:

| Service | What it does |
|---|---|
| **Sim** | The web app and API. Its volume holds uploaded files. |
| **Realtime** | Live canvas sync over WebSockets, so edits and run status appear instantly and teammates can edit together. |
| **Migrations** | Brings the database schema up to date, then exits. |
| **Cron** | Runs scheduled workflows and polling triggers (Gmail, RSS and others) every minute. |
| **Postgres** | PostgreSQL 17 with pgvector, which knowledge bases need. On a volume. |
| **Redis** | Run-status streaming and caches. |

What's set up for you:

- **Only you can sign up.** Sim's default lets anyone with the URL create an account. Here, only the email you enter at deploy time (`SIM_ADMIN_EMAIL`) can sign up or sign in. Everyone else gets "Access restricted".
- **All secrets are generated.** That covers session signing, credential encryption, API-key encryption, the internal service secret and the cron secret. The services that need a secret reference it from Sim, so nothing has to be copied by hand.
- **Pinned versions.** All four Sim images are pinned to `v0.9.26`, so a redeploy never upgrades one service ahead of the others.

## Common Use Cases

- AI agents that act on your tools: triage email, answer Slack questions, enrich CRM records, summarize tickets
- RAG over your documents with built-in knowledge bases (pgvector), exposed as a chat or an API
- Scheduled and webhook-driven automations that mix LLM steps with plain code and HTTP calls

## Dependencies for Sim Hosting

- PostgreSQL 17 with pgvector and Redis 7 (both included)
- An API key from the model provider you want to use (OpenAI, Anthropic, Google, Groq, Mistral, OpenRouter, a local Ollama and more)

### Deployment Dependencies

- [Sim documentation](https://docs.sim.ai)
- [Sim on GitHub](https://github.com/simstudioai/sim)
- [Template source on GitHub](https://github.com/nomideusz/sim-railway)

### Implementation Details

**First sign-in.** Open the Sim service's Railway domain, click **Sign up**, and register with the email you entered as `SIM_ADMIN_EMAIL`. No verification email is sent, so you go straight to your workspace.

**Add teammates.** On the Sim service, add their addresses to `ALLOWED_LOGIN_EMAILS` (comma-separated), or set `ALLOWED_LOGIN_DOMAINS=yourcompany.com` to allow a whole domain. Deploy, then invite them under Settings → Teammates. Without email set up, open the invite's **⋯** menu and choose **Copy invite link** to send it yourself.

**Add model keys.** Paste a provider key into an Agent block's API key field. To reuse it, save it once under Settings → Secrets and reference it as `{{OPENAI_API_KEY}}`. Secrets are encrypted with `ENCRYPTION_KEY`. Knowledge bases create embeddings with OpenAI, so add `OPENAI_API_KEY` to the Sim service before using them.

**Run workflows from outside.** Click **Deploy** on a workflow, create a key under Settings → Sim API keys, then:

```bash
curl -X POST "https://<your-sim>.up.railway.app/api/workflows/<workflow-id>/execute" \
  -H "X-API-Key: <your-sim-api-key>" -H "Content-Type: application/json" \
  -d '{"input": "Hello"}'
```

Schedule and webhook triggers work as soon as a workflow is deployed. The Cron service handles schedules and polling triggers.

**The "New chat" assistant.** The chat on Sim's home screen (Copilot) runs on Sim's hosted agent service and needs a `COPILOT_API_KEY` from [sim.ai](https://sim.ai). Everything else, including building and running workflows and the Agent block, works without it.

**Custom domain.** Add it to the Sim service, then set `NEXT_PUBLIC_APP_URL` on both Sim and Realtime to `https://your.domain` and deploy both.

**Email (optional).** Add `SMTP_HOST`, `SMTP_PORT`, `SMTP_USER`, `SMTP_PASS` and `FROM_EMAIL_ADDRESS` (or `RESEND_API_KEY`) to the Sim service for password resets and email notifications.

**Upgrade.** Change the image tag on Sim, Realtime, Migrations and Cron to the same new version (see [releases](https://github.com/simstudioai/sim/releases)), and deploy Migrations first. On first boot Realtime restarts a couple of times until Migrations has finished. That's expected.

**Variables to know.**

- `ENCRYPTION_KEY`, `API_ENCRYPTION_KEY`: never change these after the first deploy, or saved credentials and API keys can't be decrypted.
- `ALLOWED_LOGIN_EMAILS` / `ALLOWED_LOGIN_DOMAINS`: who can sign up and sign in. Remove both to open sign-up to anyone.
- `OLLAMA_URL`: point it at an Ollama server to use local models.

**Memory.** Measured on Railway: about 1.5 GB idle across all six services, most of it the Sim app (about 1.2 GB), with a brief peak near 2.5 GB while it boots. Use the Hobby plan or above.

## Why Deploy Sim on Railway?

Railway is a singular platform to deploy your infrastructure stack. Railway will host your infrastructure so you don't have to deal with configuration, while allowing you to vertically and horizontally scale it.

By deploying Sim on Railway, you are one step closer to supporting a complete full-stack application with minimal burden. Host your servers, databases, AI agents, and more on Railway.
