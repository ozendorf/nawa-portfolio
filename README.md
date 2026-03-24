# Nawa — Arabic Learning Platform

A mobile language learning app for Arabic (MSA, Darija, Lebanese, etc.) with ~10k users. Built and operated end-to-end as a solo developer.

**Stack:** React Native (Expo), TypeScript, Fastify, Supabase, Google Gemini, OpenAI, ElevenLabs, Docker, Cloudflare, Tailscale, Grafana

---

## What It Does

- AI tutor chat and voice-based lessons (STT/TTS)
- Structured learning paths with real-time WebSocket sessions
- Writing correction with structured AI feedback (Gemini)

---

## Architecture

```
Mobile App (React Native)
    │
    ▼
Cloudflare (CDN/WAF)
    │
    ▼
Hetzner VPS
  ├── HAProxy (TLS termination)
  ├── Fastify Proxy (auth, credential injection, AI endpoints)
  └── Vector → Grafana Cloud (logs & metrics)
    │
    ▼
AI Providers: OpenAI · Google Gemini · ElevenLabs
Auth: Supabase
```

---

## Key Design Decisions

### Credential Isolation

No third-party API key reaches the mobile client. The Fastify proxy injects provider credentials server-side — a compromised client only exposes a scoped proxy token, not direct provider keys.

### Infrastructure Hardening

Production VPS provisioned via cloud-init: SSH key-only auth (Ed25519), non-root user, UFW default-deny firewall, management access exclusively through Tailscale VPN.

### Secure Containers

Multi-stage Docker builds — dev dependencies never reach the production image. Process runs as non-root with proper PID 1 signal handling (`dumb-init`).

### Structured AI Output

LLM responses are schema-constrained and validated server-side before reaching the client, reducing prompt injection risk.

### Observability

Docker and system logs ship through Vector to Grafana Cloud (Loki + Prometheus) with authenticated sinks.

### AI Data Governance

Users see a consent screen listing exactly what data is processed and by which providers before any AI interaction.

---

## Security Improvements Backlog

| Area | Planned |
|------|---------|
| Session storage | Migrate from AsyncStorage to `expo-secure-store` |
| Rate limiting | Add `@fastify/rate-limit` per-key |
| Certificate pinning | TLS pinning for proxy endpoint |
| Input validation | JSON schema validation on all endpoints |
| Dependency scanning | Add SCA to CI pipeline |
| Secrets scanning | Pre-commit hooks (gitleaks) |

---
