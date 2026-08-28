# Vibe Security Agent

A scaffold for scoped, repeatable black-box penetration testing of web and mobile applications, where an AI planner proposes the test plan and a policy file constrains what may actually run.

## The problem

Application security testing does not scale with release velocity: an annual penetration test certifies a build that shipped forty times since. The usual answer, pointing scanners at everything continuously, fails differently — unscoped automated testing against production is itself an incident, and an AI planner given tool access will happily exceed the engagement scope it was never told about. What is missing is not another scanner but the boundary around one: an explicit, version-controlled statement of what may be tested, at what rate, with which accounts, and a plan that cannot name a target outside it.

## Quickstart

```bash
git clone https://github.com/Prompt-Shields/vibe-sec-agent.git && cd vibe-sec-agent
python3 -m venv .venv && source .venv/bin/activate && pip install --upgrade pip playwright requests && python -m playwright install --with-deps chromium
$EDITOR policy/scope.yaml   # required: allowed hosts, rate limits, test accounts
docker compose -f runners/docker-compose.yml up -d
python scripts/agent_orchestrator.py --target https://app.example.com && python scripts/merge_findings.py && python scripts/evidence_hasher.py
```

Requires Docker, Docker Compose, and Python 3.11 or later, on macOS, Linux, or WSL2. Editing `policy/scope.yaml` is not optional: the orchestrator refuses targets that are not in it. Results land in `out/report.json`, `out/merged_findings.json`, and `out/evidence_manifest.json`.

## How does it work?

**A scope policy is a version-controlled file listing the hosts that may be tested, the maximum request rate, the permitted time windows, and the test accounts to use — it is the authority on what is in bounds, not the planner.** **A constrained planner is a language model whose output must validate against a JSON schema of permitted actions, so that anything it proposes outside that vocabulary is rejected before execution rather than argued with.**

```
   policy/scope.yaml ......... allowed hosts, RPS caps, windows, test accounts
   policy/action_schema.json . the only actions that may be emitted
            |
            v
   +---------------------+
   |  agents/planner.py  |  LLM proposes a plan
   +----------+----------+
              | plan must validate against action_schema.json
              | targets must appear in scope.yaml
              v                          rejected -> never executed
   +--------------------------------------------------+
   | scripts/agent_orchestrator.py                    |
   +---------------+------------------+---------------+
                   |                  |
        web        |          mobile  |         LLM safety
   +---------------v----+   +---------v-------+  +--------------------+
   | ZAP  (baseline)    |   | MobSF (static)  |  | prompt injection   |
   | nuclei (allowlist) |   | jadx  (decomp)  |  | indirect injection |
   | httpx (recon)      |   | mitmproxy (dyn) |  | data boundary      |
   | Schemathesis (API) |   | Frida (dynamic) |  | profiles/          |
   | Playwright (auth)  |   +---------+-------+  +---------+----------+
   +---------------+----+             |                    |
                   +------------------+--------------------+
                                      v
                      +--------------------------------+
                      | merge_findings.py              |
                      | evidence_hasher.py             |
                      +---------------+----------------+
                                      v
                          out/report.json (finding_schema.json)
                          out/evidence_manifest.json  <- hashed
                                      |
                          human review before escalation
```

**Evidence hashing means each captured request and response is recorded with a cryptographic digest in a manifest, so a finding presented later can be shown to correspond to the traffic that produced it.** Every runner is a pinned container, so a rerun three months on uses the same tool version rather than whatever the registry serves that day.

Findings conform to `reporting/finding_schema.json`:

```json
{
  "id": "WEB-AUTHZ-001",
  "title": "IDOR in /api/v1/orders/{id}",
  "severity": "high",
  "cvss": "8.7",
  "evidence": {"request": "...", "response": "..."},
  "repro_steps": ["Login as user A", "Request user B's order", "Receive 200"],
  "fix": "Enforce object-level checks"
}
```

### Safe operation

Only test assets you own or hold written permission to test. Respect the rate caps and time windows in `scope.yaml`. Use test accounts, never production credentials. Keep an operator in the loop for anything destructive. None of this is enforced by the tooling beyond the scope file — it is your engagement letter that makes it lawful.

## What this does not do

This is a scaffold, and the README of a security tool is the wrong place to overstate one.

- **It does not replace a penetration test.** It runs known checks from known tools. It does not chain vulnerabilities, reason about business logic, or find the flaw that requires understanding what the application is for. A clean run means the automated checks passed, nothing more.
- **The scope file constrains targets, not consequences.** It stops the planner naming an out-of-scope host. It does not stop an in-scope test from writing data, exhausting a queue, locking accounts, or paging someone. Testing production remains a decision with an operational blast radius.
- **It confers no legal authorisation whatsoever.** A scope file is not permission. Running this against infrastructure you do not own or have written permission to test is unlawful in most jurisdictions, and the presence of an allowlist entry is not a defence.
- **The planner is not trustworthy and is not treated as such.** Constraining it to a schema bounds the damage from a bad or manipulated plan; it does not make the plan correct. A plan that validates can still be useless, redundant, or wrong about what it is testing.
- **Findings are unvalidated and include false positives.** Output is scanner output. Nothing here triages, deduplicates against known accepted risk, or confirms exploitability. Human review before escalation is mandatory, not advisory.
- **Evidence hashing is integrity, not chain of custody.** Digests detect later alteration of the manifest. They do not establish provenance, and the manifest is not tamper-evident against someone who can rewrite both artefact and hash.
- **Mobile dynamic testing needs a prepared device.** Frida and mitmproxy require a rooted or jailbroken device or emulator with a trusted CA installed. Certificate pinning defeats the proxy path until you handle it, and that setup is not automated here.
- **Coverage gaps are structural.** No authenticated ZAP context, no stateful Schemathesis checks, no GraphQL cost or depth analysis, and nuclei runs against an allowlist rather than the full template set. These are on the roadmap and absent today.
- **The README has outrun the repository.** There is no `LICENCE` file and no `.github/workflows/security_baseline.yml` in this repository, despite both having been referenced previously. Treat the continuous integration integration as unbuilt.

## Free versus Prompt Shields Cloud

This repository is free and Apache 2.0 licensed in full, and stays that way. It is offensive tooling and sits outside the Prompt Shields product, which is defensive; there is no paid edition of this scaffold. The boundary across the product line: **anything an individual engineer needs is free; anything an organisation or an auditor needs is paid.** No capability moves from the free side to the paid side.

| | Free — this repository | Prompt Shields Cloud |
|---|---|---|
| Scanners, planner, runners | Complete, no feature gating | Not offered; Cloud is a defensive product |
| LLM safety probes | Static payload suite in `profiles/` | Managed detection models, retrained continuously, running in production traffic |
| Findings | Local JSON, retained by you | Hosted retention, cross-project alerting, OWASP LLM Top 10 and MITRE ATLAS mapping |
| Governance | Per-repository scope file | Organisation-wide policy, versioning, approval workflows, SSO and SAML, SCIM, RBAC |
| Compliance evidence | Local hash manifest | Hash-chained tamper-evident audit logs, EU AI Act Article 12 exports, one-click incident reports |
| Support | Community issues, best effort | Service level agreements, named support, data processing agreement and penetration test report handling |

The relationship is sequential rather than tiered: this scaffold finds the prompt injection path; Prompt Shields is the runtime control that closes it.

## Links

- Documentation: [docs.promptshields.com](https://docs.promptshields.com); runner details in [runners/README.md](runners/README.md)
- Security policy: [SECURITY.md](SECURITY.md) — report vulnerabilities privately to security@promptshields.com, never via a public issue
- Contributing: [CONTRIBUTING.md](CONTRIBUTING.md)
- Code of conduct: [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md)
- Tools used: [OWASP ZAP](https://www.zaproxy.org/), [nuclei](https://nuclei.projectdiscovery.io/), [Schemathesis](https://schemathesis.readthedocs.io/), [MobSF](https://mobsf.github.io/docs/), [mitmproxy](https://mitmproxy.org/)

## Licence

Apache 2.0 — see [LICENSE](LICENSE) and [NOTICE](NOTICE).
