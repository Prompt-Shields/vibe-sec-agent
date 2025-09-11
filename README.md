# Vibe Security Agent

Automated black-box penetration testing framework for **web** and **mobile applications**, orchestrated with an AI planner and safe guardrails.
The scaffold integrates open-source security tools (ZAP, nuclei, httpx, MobSF, mitmproxy, Schemathesis) with CI automation to provide repeatable, scoped testing.

---

## Features

* **Constrained AI planner**: generates structured test plans within a JSON schema.
* **Dockerised tool runners**: standardised scans with reproducible environments.
* **Web application testing**:

  * Reconnaissance, TLS/headers, content discovery
  * Auth & session flows (via Playwright)
  * Access control / IDOR checks
  * API fuzzing for REST & GraphQL
* **Mobile application testing**:

  * Static scans (MobSF, jadx)
  * Dynamic tests via mitmproxy & Frida
  * Deep link and intent validation
* **LLM safety probes**:

  * Prompt injection & indirect injection tests
  * Data boundary validation
* **CI/CD ready**:

  * GitHub Actions workflow for nightly baselines
  * Evidence hashing & report merging
* **Safe by default**:

  * Scope allow-list (`policy/scope.yaml`)
  * Action schema (`policy/action_schema.json`)
  * Rate limits and time windows

---

## Quick Start

1. **Clone the repo**

   ```bash
   git clone git@github.com:<your-username>/vibe-sec-agent.git
   cd vibe-sec-agent
   ```

2. **Define your scope**
   Edit `policy/scope.yaml` to include allowed domains, rate limits, and test accounts.

3. **Start supporting services**

   ```bash
   docker compose -f runners/docker-compose.yml up -d
   ```

4. **Capture login flows (optional)**

   ```bash
   python scripts/playwright_login.py --target https://app.example.com --user test@example.com --password test123
   ```

5. **Run baseline scan**

   ```bash
   python scripts/agent_orchestrator.py --target https://app.example.com
   ```

6. **View results**

   * Findings: `out/report.json`
   * Evidence: `out/evidence_manifest.json`
   * Merged artefacts: `out/merged_findings.json`

---

## Detailed Instructions

### Prerequisites

* macOS, Linux, or WSL2 on Windows
* 4 CPU cores, 8 GB RAM minimum
* Docker + Docker Compose
* Python 3.11+
* Git
* Optional: Java (jadx), Node.js 18+ (for Playwright)

Install Python packages:

```bash
python -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install playwright requests
python -m playwright install --with-deps chromium
```

### Repository bootstrap

```bash
git clone git@github.com:<your-username>/vibe-sec-agent.git
cd vibe-sec-agent
```

### Configure scope

Edit `policy/scope.yaml` with allowed targets, rate limits, legal references, and test accounts.

### Start long-lived services

```bash
docker compose -f runners/docker-compose.yml up -d
```

### Quick web baseline run

```bash
python scripts/agent_orchestrator.py --target https://app.vibecoding.com
python scripts/merge_findings.py
python scripts/evidence_hasher.py
```

Outputs in `out/` folder.

### Authenticated flows

```bash
python scripts/playwright_login.py --target https://app.vibecoding.com/login --user user@example.com --password pass
```

### API testing

```bash
python scripts/agent_orchestrator.py --target https://api.vibecoding.com --context '{"openapi_url":"https://api.vibecoding.com/openapi.json"}'
```

### Mobile application testing

Run MobSF static scan:

```bash
export MOBSF_URL=http://localhost:8000
export MOBSF_KEY=<api_key>
python scripts/mobsf_scan.py --url $MOBSF_URL --key $MOBSF_KEY --apk ./apks/vibe-release.apk
```

Dynamic tests: route emulator traffic through mitmproxy at `localhost:8080`.

### LLM safety probes

Review and use payloads in `profiles/llm-prompt-suite.txt` against user inputs or stored content.

### CI Integration

* Workflow in `.github/workflows/security_baseline.yml`
* Schedules weekday baselines (06:00 UTC)
* Uploads evidence as artefacts
* Add secrets (e.g. `MOBSF_KEY`, test creds) in GitHub repo settings

Run manually via Actions tab.

### Reporting

* Findings in `reporting/finding_schema.json`
* Example:

```json
{
  "id": "WEB-AUTHZ-001",
  "title": "IDOR in /api/v1/orders/{id}",
  "severity": "high",
  "cvss": "8.7",
  "evidence": {"request":"...","response":"..."},
  "repro_steps": ["Login as user A","Request user B's order","Receive 200"],
  "fix": "Enforce object-level checks"
}
```

### Safe operation checklist

* Only scan allowed targets
* Respect RPS caps and time windows
* Use test accounts only
* Keep operator in loop for destructive tests
* Never include production secrets

### Troubleshooting

* **Docker pull fails**: check network or `docker login`
* **ZAP empty output**: site may block scans, adjust UA
* **Schemathesis fails**: validate schema URL, add auth headers
* **MobSF 403**: check API key and container health
* **Mitmproxy no traffic**: check device proxy config and CA install

### Next steps

* Wire authenticated ZAP context
* Add operator approval in CI
* Store artefacts in evidence bucket
* Build dashboard for trends

---

## Safety and Legal

* Only run against assets you own or have written permission to test
* Stick to scope and rate limits
* Use test accounts, never production
* Human review required before escalation

---

## Roadmap

* Authenticated ZAP integration
* Broaden nuclei coverage
* Schemathesis stateful checks
* Automated GraphQL cost/depth checks
* Add Frida/Objection scripts for deeper mobile testing

---

## Licence

MIT (see `LICENCE` file).
