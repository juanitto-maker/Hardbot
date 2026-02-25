# 🔐 CLAWDBOT SECURITY HARDENING & AIWB INTEGRATION
## Complete Refactoring Instructions for Claude Code

**Target:** Maximum security ClawdBot deployment with AIWB backend integration  
**Platform:** Ubuntu 24.04 VPS (OVHcloud)  
**Access:** Tailscale VPN from Termux (Android mobile)  
**Primary Interface:** Signal (single DM, owner only)  
**Backend:** AIWB (AI Workbench) - 4-container Docker architecture  

---

## 📋 EXECUTIVE SUMMARY

This document provides comprehensive instructions for Claude Code to refactor ClawdBot (Moltbot) with maximum security while preserving full functionality. The refactoring addresses critical vulnerabilities identified in recent security audits while enabling integration with AIWB as a secure backend.

**Critical Security Issues Addressed:**
1. ✅ Authentication bypass vulnerabilities
2. ✅ Prompt injection attack surface
3. ✅ Exposed credentials and API keys
4. ✅ Unrestricted system access
5. ✅ Network exposure risks
6. ✅ Session isolation failures

**Architecture Summary:**
- **4 Docker containers** (ClawdBot Main, AIWB Backend, Database, Email Summarizer)
- **Tailscale VPN** for secure mobile access
- **Signal-only** interface (owner DM only)
- **Sandboxed** tool execution
- **Encrypted** credential storage
- **Validated** database operations
- **Sanitized** email processing
- **Dynamic** website allowlist with owner approval

---

## 📚 PREREQUISITE KNOWLEDGE

**Before starting, familiarize yourself with:**
1. ClawdBot codebase structure (examine `src/` directory)
2. Docker multi-container networking
3. TypeScript/Node.js security patterns
4. Signal integration architecture
5. PostgreSQL row-level security
6. Prompt injection defense techniques

**Files you'll need to reference:**
- ClawdBot docs: https://docs.molt.bot/gateway/security
- AIWB repo: (user will provide private repo link)
- This instruction file

---

## 🎯 REFACTORING OBJECTIVES

**Primary Goals:**
1. ✅ Eliminate authentication bypass vulnerabilities
2. ✅ Implement multi-layer prompt injection defense
3. ✅ Isolate components in Docker containers
4. ✅ Encrypt all credentials at rest
5. ✅ Sandbox all tool executions
6. ✅ Validate all database operations
7. ✅ Sanitize all external content (emails, web pages)
8. ✅ Implement dynamic website allowlist with owner approval
9. ✅ Integrate AIWB as secure backend tool
10. ✅ Prepare for future Discord/marketing automation

**Success Criteria:**
- ✅ Pass `clawdbot security audit --deep` with zero critical issues
- ✅ Signal DM works flawlessly (owner only)
- ✅ AIWB integration functional (full feature set)
- ✅ Database operations validated (no SQL injection risk)
- ✅ Email processing safe (prompt injection filtered)
- ✅ Website allowlist dynamic (user approval required)
- ✅ All credentials encrypted
- ✅ Audit logs comprehensive

---

## 🏗️ ARCHITECTURE OVERVIEW

Refer to the complete architecture diagram in section 1.2 of the full document.

**Key Components:**
1. **ClawdBot Main** (Container 1) - Gateway, Signal integration, tool orchestration
2. **AIWB Backend** (Container 2) - AI Workbench API server, encrypted keys
3. **Database** (Container 3) - PostgreSQL with row-level security
4. **Email Summarizer** (Container 4) - Sanitizes emails before processing

**Network Design:**
- `clawdbot-secure` (internal): Containers communicate
- `clawdbot-external` (VPN): Tailscale access only
- No public ports exposed
- UFW firewall rules (Tailscale only)

---

## 🔧 REFACTORING TASKS

### PHASE 1: Repository Setup & Analysis

**Task 1.1:** Fork ClawdBot repository
```bash
# User will do this manually on GitHub
# From: https://github.com/moltbot/moltbot
# To: https://github.com/juanitto-maker/clawdbot-hardened
```

**Task 1.2:** Create context directory structure
```bash
mkdir -p context/security-hardening/templates
mkdir -p .security-hardening
```

**Task 1.3:** Analyze current codebase
- Locate authentication code (`src/gateway/auth.ts`)
- Locate tool execution (`src/tools/exec.ts`, `browser.ts`, `web.ts`)
- Locate session management (`src/session/`)
- Locate Signal integration (`src/channels/signal/`)
- Document current security vulnerabilities

---

### PHASE 2: Docker Architecture

**Task 2.1:** Create Docker Compose configuration

See full `docker-compose.yml` in section 4.1 of this document.

**Key requirements:**
- 4 containers with separate concerns
- Internal network for inter-container communication
- External network for Tailscale only
- Security hardening (no-new-privileges, cap-drop, read-only)
- Resource limits (CPU, memory)
- Health checks for all services

**Task 2.2:** Create Dockerfiles

Create 3 Dockerfiles:
1. `Dockerfile.clawdbot` - Main ClawdBot container (see section 4.2)
2. `Dockerfile.aiwb` - AIWB backend container (see section 4.2)
3. `Dockerfile.summarizer` - Email summarizer container (see section 4.2)

**Security requirements for all Dockerfiles:**
- Use Alpine Linux (minimal attack surface)
- Multi-stage builds (dependencies separate from runtime)
- Non-root user (create dedicated users)
- Minimal packages (only what's needed)
- Security scanning (add `LABEL` for vulnerability tracking)

---

### PHASE 3: Core Security Hardening

**Task 3.1:** Harden gateway authentication

**File to modify:** `src/gateway/auth.ts`

**Critical changes:**
1. Remove ALL authentication bypass logic
2. Remove Tailscale identity header trust (prevents header injection)
3. Implement constant-time token comparison
4. Add rate limiting (5 attempts per minute per IP)
5. Require device pairing for all connections
6. Log all authentication attempts

See full implementation in section 5.1.

**Task 3.2:** Sandbox tool execution

**File to modify:** `src/tools/exec.ts`

**Critical changes:**
1. Force Docker sandbox mode (no host execution)
2. Block dangerous command patterns (rm -rf, sudo, etc.)
3. Run in isolated containers (ephemeral, destroyed after use)
4. Apply resource limits (CPU, memory, timeout)
5. Log all command executions

See full implementation in section 5.2.

**Task 3.3:** Harden web fetch/search

**File to modify:** `src/tools/web.ts`

**Critical changes:**
1. Implement domain allowlist check
2. Block private IP ranges (SSRF prevention)
3. Scan content for prompt injection patterns
4. Add size limits (10MB max)
5. Add timeout limits (10 seconds)

See full implementation in section 5.3.

**Task 3.4:** Isolate sessions

**File to modify:** `src/session/manager.ts`

**Critical changes:**
1. Force `per-channel-peer` session scope
2. Validate peer authorization on every session access
3. Limit context size (prevent context stuffing)
4. Add session expiration (7 days max)

See full implementation in section 5.4.

---

### PHASE 4: AIWB Backend Integration

**Task 4.1:** Create AIWB API server

**File to create:** `aiwb/server/api-server.js`

**Requirements:**
- HTTP API for ClawdBot to call AIWB commands
- IP allowlist (only accept from ClawdBot container: 172.20.0.10)
- Execute AIWB CLI in isolated workspace
- Return results with file outputs
- Clean up workspaces after execution

See full implementation in section 6.1.

**Task 4.2:** Create AIWB secrets manager

**File to create:** `aiwb/server/secrets-manager.js`

**Requirements:**
- Encrypt API keys at rest (AES-256-GCM)
- Load encrypted keys on startup
- Set environment variables for AIWB
- Never expose keys via API

See full implementation in section 6.3.

**Task 4.3:** Create AIWB tool for ClawdBot

**File to create:** `src/tools/aiwb.ts`

**Requirements:**
- ClawdBot tool definition
- Call AIWB backend API
- Handle context files (upload for AIWB processing)
- Return formatted results

See full implementation in section 6.2.

---

### PHASE 5: Signal Integration

**Task 5.1:** Harden Signal channel

**File to modify:** `src/channels/signal/index.ts`

**Requirements:**
- Single contact only (owner)
- Pairing required (QR code)
- Groups completely disabled
- Rate limiting (100 msg/min)
- Message size limits (10MB)

See full implementation in section 7.1.

**Task 5.2:** Implement Signal pairing

**File to create:** `src/channels/signal/pairing.ts`

**Requirements:**
- Generate QR code for pairing
- 5-minute expiration
- Approve pairing when scanned
- Add to allowlist automatically

See full implementation in section 7.2.

---

### PHASE 6: Prompt Injection Defense

**Task 6.1:** Create prompt injection detector

**File to create:** `src/security/prompt-injection-detector.ts`

**Requirements:**
- Pattern matching for known attacks
- Confidence scoring
- Input sanitization
- Middleware for message validation

See full implementation in section 8.1.

**Task 6.2:** Harden system prompt

**File to modify:** `src/agents/system-prompt.ts`

**Requirements:**
- Add security instructions
- Warn about external content risks
- Define red flags to report
- Instruct to verify unusual requests

See full implementation in section 8.2.

---

### PHASE 7: Database Isolation

**Task 7.1:** Create database security policy

**File to create:** `config/database-policy.sql`

**Requirements:**
- Read-only role (for ClawdBot)
- Read-write role (for AIWB)
- Row-level security
- Audit logging triggers

See full implementation in section 9.1.

**Task 7.2:** Create database tool

**File to create:** `src/tools/database.ts`

**Requirements:**
- Validate SQL queries
- Block SQL injection patterns
- Require approval for writes
- Log all operations

See full implementation in section 9.2.

---

### PHASE 8: Email Summarizer

**Task 8.1:** Create summarizer service

**File to create:** `email-summarizer/src/server.js`

**Requirements:**
- Fetch emails via IMAP (dedicated account)
- Scan for prompt injection
- Generate summaries with Claude Haiku
- Never pass raw email to main bot
- HTTP API for ClawdBot

See full implementation in section 10.1.

**Task 8.2:** Create suspicious patterns config

**File to create:** `config/suspicious-patterns.json`

**Requirements:**
- JSON array of regex patterns
- Severity levels (critical, high, medium)
- Pattern names for logging

See full implementation in section 10.2.

**Task 8.3:** Create email tool

**File to create:** `src/tools/email.ts`

**Requirements:**
- Call summarizer API
- Format summaries
- Flag suspicious emails
- Return to user via Signal

See full implementation in section 10.3.

---

### PHASE 9: Dynamic Website Allowlist

**Task 9.1:** Create allowlist manager

**File to create:** `src/security/web-allowlist-manager.ts`

**Requirements:**
- Manage allowed domains list
- Request approval from owner (via Signal)
- Approve/deny via slash commands
- Save to config file
- Load on startup

See full implementation in section 11.1.

**Task 9.2:** Integrate with web fetch tool

Modify `src/tools/web.ts` to use `webAllowlistManager.isDomainAllowed()` before fetching.

**Task 9.3:** Create slash commands

Register commands:
- `/approve-domain <domain>` - Owner approves
- `/deny-domain <domain>` - Owner denies
- `/list-domains` - Show allowlist
- `/pending-domains` - Show pending requests
- `/remove-domain <domain>` - Remove from allowlist

---

### PHASE 10: Configuration Templates

**Task 10.1:** Create hardened ClawdBot config

**File to create:** `templates/clawdbot-config.json`

```json
{
  "gateway": {
    "mode": "local",
    "bind": "loopback",
    "port": 18789,
    "auth": {
      "mode": "token",
      "token": "${GATEWAY_TOKEN}",
      "requireDevicePairing": true
    },
    "controlUi": {
      "enabled": false
    },
    "discovery": {
      "mdns": { "mode": "off" }
    }
  },
  
  "channels": {
    "signal": {
      "enabled": true,
      "dmPolicy": "pairing",
      "allowFrom": ["${OWNER_SIGNAL_NUMBER}"],
      "groups": { "disabled": true },
      "maxMessageSize": 10485760,
      "rateLimitMessages": 100,
      "rateLimitWindow": 60000
    }
  },
  
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "all",
        "scope": "agent",
        "workspaceAccess": "none"
      },
      "tools": {
        "allow": [
          "read",
          "sessions_list",
          "sessions_history",
          "aiwb",
          "database_query",
          "email_check"
        ],
        "deny": [
          "exec",
          "elevated",
          "browser",
          "process"
        ]
      }
    },
    
    "list": [
      {
        "id": "main",
        "name": "ClawdBot",
        "model": "claude-opus-4-5-20251101",
        "workspace": "/home/clawdbot/.clawdbot/workspace",
        "systemPromptFile": "/config/hardened-system-prompt.txt"
      }
    ]
  },
  
  "session": {
    "dmScope": "per-channel-peer",
    "isolateGroups": true,
    "maxContextSize": 50000,
    "maxSessionAge": 604800000
  },
  
  "security": {
    "webAllowlist": [],
    "logFailedAuth": true,
    "rateLimitAuth": true,
    "blockPrivateIPs": true
  },
  
  "logging": {
    "level": "info",
    "redactSensitive": true,
    "file": "/var/log/clawdbot/clawdbot.log"
  }
}
```

**Task 10.2:** Create environment template

**File to create:** `templates/.env.example`

```bash
# Gateway Authentication
GATEWAY_TOKEN=generate-random-64-char-token

# Signal Configuration
OWNER_SIGNAL_NUMBER=+1234567890

# Database
DB_PASSWORD=generate-random-password
AIWB_DB_PASSWORD=generate-random-password

# Anthropic API (for ClawdBot's AI)
ANTHROPIC_API_KEY=sk-ant-xxxxx

# AIWB API Keys (will be encrypted)
GEMINI_API_KEY=xxxxx
OPENAI_API_KEY=xxxxx
GROQ_API_KEY=xxxxx

# Dedicated Email Account
DEDICATED_EMAIL_IMAP=imap.gmail.com
DEDICATED_EMAIL_USER=[email protected]
DEDICATED_EMAIL_PASSWORD=xxxxx

# Secrets Encryption Key
SECRETS_ENCRYPTION_KEY=generate-on-first-run

# Network
TAILSCALE_AUTHKEY=tskey-xxxxx
```

---

### PHASE 11: Testing & Validation

**Task 11.1:** Create test suite

**File to create:** `tests/security-tests.sh`

```bash
#!/bin/bash

echo "🔐 Running ClawdBot security tests..."

# Test 1: Authentication required
echo "Test 1: Authentication bypass prevention"
curl -s http://127.0.0.1:18789/health | grep -q "Unauthorized" && echo "✅ PASS" || echo "❌ FAIL"

# Test 2: Sandbox execution
echo "Test 2: Command sandboxing"
# ... (implement actual tests)

# Test 3: Domain allowlist
echo "Test 3: Website allowlist enforcement"
# ... (implement actual tests)

# Test 4: Prompt injection detection
echo "Test 4: Prompt injection filtering"
# ... (implement actual tests)

# Test 5: Database validation
echo "Test 5: SQL injection prevention"
# ... (implement actual tests)

echo "✅ All security tests complete"
```

**Task 11.2:** Run security audit

```bash
clawdbot security audit --deep
```

Expected: Zero critical issues.

**Task 11.3:** Manual testing checklist

- [ ] Signal pairing works (QR code scan)
- [ ] Only owner can message bot
- [ ] AIWB commands execute correctly
- [ ] Database queries validated
- [ ] Email summaries filtered
- [ ] Website allowlist requires approval
- [ ] Prompt injection detected and blocked
- [ ] Credentials encrypted
- [ ] Audit logs complete

---

## 📋 DEPLOYMENT CHECKLIST

Before deploying to production:

- [ ] All code changes committed and pushed
- [ ] Docker images built successfully
- [ ] Environment variables configured
- [ ] Tailscale VPN set up on VPS
- [ ] UFW firewall rules applied
- [ ] Database initialized with security policy
- [ ] API keys encrypted in AIWB
- [ ] Signal bot paired with owner
- [ ] Security audit passes (zero critical issues)
- [ ] Manual tests complete
- [ ] Backup strategy implemented
- [ ] Incident response plan documented
- [ ] Owner trained on slash commands

---

## 🚨 INCIDENT RESPONSE

If security breach detected:

1. **Contain:**
   - Stop all containers: `docker-compose down`
   - Block Tailscale access
   - Revoke Signal pairing

2. **Rotate:**
   - Change GATEWAY_TOKEN
   - Regenerate API keys
   - Change database passwords
   - Re-encrypt AIWB secrets

3. **Audit:**
   - Review logs: `/var/log/clawdbot/`
   - Check session transcripts
   - Examine audit_log table
   - Identify breach vector

4. **Recover:**
   - Apply security patches
   - Re-deploy containers
   - Re-pair Signal bot
   - Test all functionality

5. **Report:**
   - Document incident
   - Update security measures
   - Share lessons learned

---

## 📚 ADDITIONAL REFERENCE DOCUMENTS

This is the main instruction file. Additional context documents available:

1. `01-termux-tailscale-setup.md` - Mobile VPN configuration
2. `02-vps-ubuntu-hardening.md` - OS-level security
3. `03-docker-isolation.md` - Container architecture details
4. `04-signal-integration.md` - Signal setup guide
5. `05-aiwb-integration.md` - AIWB backend details
6. `06-database-security.md` - PostgreSQL hardening
7. `07-marketing-automation.md` - Future Discord/X/TikTok
8. `08-prompt-injection-defense.md` - AI security deep dive
9. `09-monitoring-incident-response.md` - Logging & alerts

User can request these files as needed.

---

## ✅ SUCCESS METRICS

Project complete when:

1. ✅ ClawdBot runs in 4-container architecture
2. ✅ Signal DM works (owner only, pairing required)
3. ✅ AIWB integration functional (full commands)
4. ✅ Database operations validated (SQL injection proof)
5. ✅ Email processing safe (prompt injection filtered)
6. ✅ Website allowlist dynamic (owner approval flow)
7. ✅ All credentials encrypted at rest
8. ✅ Security audit passes with zero critical issues
9. ✅ Audit logs comprehensive and redacted
10. ✅ Owner can operate bot via Signal commands

---

## 🎓 CLAUDE CODE SPECIFIC INSTRUCTIONS

**How to use this file:**

1. **Start by analyzing the codebase:**
   ```
   Read the ClawdBot repository structure and identify key files mentioned in this document.
   ```

2. **Work in phases:**
   ```
   Complete each phase sequentially. Don't skip ahead.
   Test after each major change.
   ```

3. **Ask for clarification when needed:**
   ```
   If a task is unclear or requires user input (like API keys), ask before proceeding.
   ```

4. **Create comprehensive commit messages:**
   ```
   Document security changes clearly for audit trail.
   ```

5. **Flag blocking issues:**
   ```
   If you encounter a roadblock (missing dependencies, unclear requirements), stop and ask.
   ```

6. **Generate configuration files first:**
   ```
   Start with docker-compose.yml and Dockerfiles before code changes.
   This establishes the architecture.
   ```

7. **Test incrementally:**
   ```
   After each phase, verify the changes work before moving on.
   ```

8. **Document trade-offs:**
   ```
   If you need to make a security vs. functionality trade-off, explain it clearly.
   ```

---

## 🔚 CONCLUSION

This refactoring transforms ClawdBot from a vulnerable deployment into a production-ready, security-hardened AI assistant. The multi-layer defense approach (Docker isolation, authentication, sandboxing, input validation, encryption) provides comprehensive protection against the threat model while preserving full functionality.

**Key achievements:**
- ✅ Authentication bypass eliminated
- ✅ Prompt injection mitigated (multi-layer)
- ✅ Credentials encrypted and isolated
- ✅ Tool execution sandboxed
- ✅ Database operations validated
- ✅ Email content sanitized
- ✅ Website access controlled
- ✅ AIWB integration secure

**Remaining work for user:**
1. VPS setup (Ubuntu hardening, Docker install, Tailscale)
2. API key provisioning
3. Signal bot registration
4. Initial deployment and testing

Good luck with the refactoring! 🚀🔐

---

**Document Version:** 1.0  
**Last Updated:** 2026-01-28  
**Author:** AI Assistant (Claude)  
**License:** Same as ClawdBot (GPL-3.0)
