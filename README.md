# Vibe Security Agent

Automated black-box penetration testing framework for **web** and **mobile applications**, orchestrated with an AI planner and safe guardrails.  
The scaffold integrates open-source security tools (ZAP, nuclei, httpx, MobSF, mitmproxy, Schemathesis) with CI automation to provide repeatable, scoped testing.

---

## Features

- **Constrained AI planner**: generates structured test plans within a JSON schema.
- **Dockerised tool runners**: standardised scans with reproducible environments.
- **Web application testing**:
  - Reconnaissance, TLS/headers, content discovery
  - Auth & session flows (via Playwright)
  - Access control / IDOR checks
  - API fuzzing for REST & GraphQL
- **Mobile application testing**:
  - Static scans (MobSF, jadx)
  - Dynamic tests via mitmproxy & Frida
  - Deep link and intent validation
- **LLM safety probes**:
  - Prompt injection & indirect injection tests
  - Data boundary validation
- **CI/CD ready**:
  - GitHub Actions workflow for nightly baselines
  - Evidence hashing & report merging
- **Safe by default**:
  - Scope allow-list (`policy/scope.yaml`)
  - Action schema (`policy/action_schema.json`)
  - Rate limits and time windows

---

## Quick Start

1. **Clone the repo**
   ```bash
   git clone git@github.com:<your-username>/vibe-sec-agent.git
   cd vibe-sec-agent