# 🔐 Hardbot

> Multi-layer security-hardening blueprint for AI agent deployments

**Hardbot** is a comprehensive security-hardening specification for deploying AI agents (originally targeting MoltBot/ClawdBot) in hostile environments. It transforms a vulnerable agent deployment into a production-ready, multi-container architecture with defense-in-depth at every layer — from network isolation to prompt-injection filtering.

[![Status](https://img.shields.io/badge/status-blueprint-yellow.svg)]()
[![License](https://img.shields.io/badge/license-GPL--3.0-blue.svg)]()
[![Architecture](https://img.shields.io/badge/architecture-4--container-success.svg)]()

> **📝 Status:** This repository currently contains the **complete architectural specification** in `docs/`. Implementation is implementation-ready — designed to be fed to Claude Code or executed by a developer following the phase-by-phase tasks.

---

## 🎯 What Hardbot Solves

Most off-the-shelf AI agents (Signal/Telegram/Discord bots, Claude Code variants, RAG systems) ship with multiple unaddressed attack vectors:

| Attack | Hardbot defense |
|---|---|
| Authentication bypass | Constant-time token comparison + device pairing + rate limiting |
| Prompt injection | Multi-layer pattern detector + email summarizer + system prompt hardening |
| Tool execution exploits | Forced Docker sandbox + dangerous-pattern blocker + ephemeral containers |
| SSRF via web fetch | Domain allowlist + private IP blocking + size/timeout limits |
| Credential exposure | AES-256-GCM encryption at rest + isolated secrets manager |
| SQL injection | Validated queries + read-only roles + row-level security + audit triggers |
| Email-borne payloads | Dedicated summarizer container, never raw email to main agent |
| Network exposure | Tailscale VPN only, UFW firewall, no public ports |
| Session leakage | `per-channel-peer` scope + context size limits + 7-day expiration |
| Group spam / impersonation | Single-owner DM, groups disabled, pairing required |

---

## 🏗️ Architecture

A 4-container Docker deployment, each with a single responsibility:

```
                    ┌──────────────────┐
                    │   Tailscale VPN  │  ← only entry point
                    └────────┬─────────┘
                             │
        ┌────────────────────┴────────────────────┐
        │     UFW firewall (Tailscale-only)        │
        ├──────────────────────────────────────────┤
        │  Container 1: Hardbot Main               │
        │  • Signal gateway (owner DM only)        │
        │  • Tool orchestration                    │
        │  • Prompt-injection filter               │
        │  • Web allowlist enforcement             │
        ├──────────────────────────────────────────┤
        │  Container 2: AIWB Backend               │
        │  • AI Workbench API server               │
        │  • Encrypted secrets manager (AES-GCM)   │
        │  • IP allowlisted (only Container 1)     │
        ├──────────────────────────────────────────┤
        │  Container 3: PostgreSQL                 │
        │  • Read-only role for Hardbot            │
        │  • Read-write role for AIWB              │
        │  • Row-level security                    │
        │  • Audit-log triggers                    │
        ├──────────────────────────────────────────┤
        │  Container 4: Email Summarizer           │
        │  • Dedicated IMAP account                │
        │  • Prompt-injection scan                 │
        │  • Claude Haiku summarization            │
        │  • Never passes raw email upstream       │
        └──────────────────────────────────────────┘
```

All inter-container traffic on internal Docker network (`hardbot-secure`). External access is Tailscale-only.

---

## 🛡️ Defense Layers

### 1. Network
- Tailscale VPN as the only ingress
- UFW firewall blocks everything except Tailscale subnet
- No public ports; even SSH is Tailscale-gated
- Internal Docker network for inter-container communication

### 2. Authentication
- Random 64-character gateway token (rotated)
- Constant-time comparison (prevents timing attacks)
- Device pairing required for every connection
- Rate limit: 5 auth attempts/minute/IP
- All attempts logged

### 3. Tool Execution
- Forced Docker sandbox mode (no host execution)
- Dangerous command patterns blocked (`rm -rf`, `sudo`, etc.)
- Ephemeral containers (destroyed after each command)
- Resource limits (CPU, memory, timeout)

### 4. Web Access
- Dynamic domain allowlist (owner approves new domains via Signal slash commands)
- Private IP ranges blocked (SSRF prevention)
- Content scanned for prompt-injection patterns
- 10MB size limit, 10s timeout

### 5. Database
- Hardbot has read-only role
- AIWB has read-write role
- Row-level security policies
- All DML logged via triggers
- SQL injection patterns blocked at app layer

### 6. Email Pipeline
- Separate IMAP account (never shares credentials with main agent)
- Suspicious-pattern scan against `config/suspicious-patterns.json`
- Claude Haiku summarizes content
- Main agent receives only the summary, never raw email

### 7. Credentials
- AES-256-GCM encryption at rest
- Isolated secrets manager container
- Keys never exposed via API
- Single startup-time decryption into env vars

### 8. Sessions
- `per-channel-peer` scope (no cross-session leakage)
- Peer authorization validated on every access
- Context size cap (50,000 tokens)
- 7-day session expiration

---

## 📚 Documentation Structure

The `docs/` folder contains the complete refactoring specification:

| Document | Contents |
|---|---|
| `10-CLAUDE-CODE-REFACTORING.md` | Master spec — phases, tasks, architecture, configs, test suite |
| `Tamp.md` | Working notes |

The master document is structured for direct execution by Claude Code, broken into 11 phases:

1. Repository setup & analysis
2. Docker architecture
3. Core security hardening (auth, sandbox, web, sessions)
4. AIWB backend integration
5. Signal integration & pairing
6. Prompt injection defense
7. Database isolation
8. Email summarizer
9. Dynamic website allowlist
10. Configuration templates
11. Testing & validation

---

## 🚀 How to Use This

### Option A — Hand to Claude Code

The master document is written for AI execution:

```
1. Fork MoltBot to your repo
2. Open Claude Code in the fork
3. Provide docs/10-CLAUDE-CODE-REFACTORING.md as context
4. Execute phase by phase, testing after each
5. Run security audit at the end
```

### Option B — Manual implementation

The spec is also human-readable. Each phase is independent enough to implement manually if you prefer.

### Option C — Adopt as methodology

Even without using MoltBot specifically, the layered defense pattern applies to any AI agent deployment. Use the architecture, threat model, and configuration templates as a reference for your own stack.

---

## 🎯 Why Hardbot vs. Off-the-Shelf Hardened Agents

Even as the AI-agent security space matures with vendor-hardened offerings, Hardbot's specific value lies in:

- **AIWB-native integration** — designed from day one to use [AI Workbench](https://github.com/juanitto-maker/AIworkbench) as its tool backend, not as an afterthought
- **Mobile-first ownership** — Termux + Tailscale + Signal — the whole ops surface fits in your pocket
- **Self-hosted philosophy** — no vendor lock-in, no telemetry, runs on a single Ubuntu VPS
- **Off-grid friendly** — works on intermittent connections, low-bandwidth environments
- **Composable** — pair with [GuardOS](https://github.com/juanitto-maker/GuardOS) for air-gapped local AI work and [Clide](https://github.com/juanitto-maker/Clide) for additional channel coverage

---

## 📋 Threat Model

Hardbot assumes a hostile environment:

- ✅ The internet is hostile
- ✅ External content (emails, web pages) may be adversarial
- ✅ Vendor APIs may be compromised
- ✅ Other Docker containers on the host may be compromised
- ✅ Physical device may be lost or seized
- ❌ Owner's signing key remains uncompromised (out-of-scope)
- ❌ Tailscale's coordination server is trusted (mitigatable with self-hosted Headscale)

---

## ✅ Success Criteria

The deployment is considered hardened when:

1. ✅ Pass `clawdbot security audit --deep` with zero critical issues
2. ✅ Signal pairing works (QR scan, owner-only)
3. ✅ AIWB integration functional (full command set)
4. ✅ Database operations validated (no SQL injection vector)
5. ✅ Email summaries scrubbed (prompt injection filtered)
6. ✅ Website allowlist enforces owner approval
7. ✅ All credentials encrypted at rest
8. ✅ Comprehensive audit logs

---

## 🤝 Contributing

This is currently a personal blueprint. Contributions welcome if you:

- Implement a phase and want to share findings
- Discover additional attack vectors
- Adapt it to non-MoltBot agents (Discord, Matrix, etc.)
- Improve the test suite

Open an issue or PR.

---

## 📄 License

GPL-3.0 — same as MoltBot/ClawdBot upstream. The blueprint and any derived implementations remain open-source.

---

## 🔗 Related Projects

- [AI Workbench (AIWB)](https://github.com/juanitto-maker/AIworkbench) — the multi-AI orchestration backend Hardbot integrates with
- [Clide](https://github.com/juanitto-maker/Clide) — Rust-based Telegram/Matrix AI assistant (sister project)
- [GuardOS](https://github.com/juanitto-maker/GuardOS) — air-gapped personal AI OS (companion privacy stack)
- [virtOS](https://github.com/juanitto-maker/virtOS) — privacy-first virtualized environment
- [MoltBot upstream](https://github.com/moltbot/moltbot) — the unhardened base this spec hardens

---

Built with ❤️ by [juanitto-maker](https://github.com/juanitto-maker) — part of the [digitalAIventures](https://amber-kitten-4khe.here.now/) portfolio.
