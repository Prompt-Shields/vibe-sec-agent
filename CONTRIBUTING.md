# Contributing to Vibe Security Agent

This is offensive tooling. Contributions are held to a higher bar than usual,
because a defect here can cause an incident on someone else's infrastructure.

## Before you start

Open an issue first. In particular, discuss before adding any check that writes
data, consumes quota, or can lock an account.

## Development setup

Requires Docker, Docker Compose, and Python 3.11 or later.

```bash
python3 -m venv .venv && source .venv/bin/activate
pip install --upgrade pip playwright requests
python -m playwright install --with-deps chromium
docker compose -f runners/docker-compose.yml up -d
```

## Checks that must pass

There is no test suite in this repository yet, and no continuous integration.
Adding either alongside a change is welcome. State in the pull request what you
ran the scaffold against — which must be infrastructure you own or hold written
permission to test.

## Rules for anything that touches a target

- **The scope file is the authority.** No code path may test a host that is not
  in `policy/scope.yaml`. Do not add a bypass, not even for development.
- **The planner is untrusted input.** Its output must validate against
  `policy/action_schema.json` before anything executes. Widening that schema
  widens the blast radius.
- **Respect the rate caps and time windows.** A check that ignores them is a
  denial of service with a friendly name.
- **Destructive checks require an operator in the loop.** Never make one the
  default.
- **Never commit a real target, credential, or engagement detail**, including in
  test fixtures and example configuration.
## Coding conventions

Match the surrounding code. Comment density, naming, and idiom should be
indistinguishable from what is already there. A change that reads as though it
were written by a different person is harder to review, whatever its merits.

## Never commit

- Live credentials, API keys, tokens, or connection strings with real passwords
- Customer data, real prompt text, or anything that identifies a person
- Generated artefacts, build output, or editor and OS scratch files
- Roadmap phases, customer names, pricing strategy, or other internal material.
  This repository is public.

If you believe a credential has been committed, email
**security@promptshields.com** immediately rather than opening a pull request
that removes it — a public commit that deletes a secret advertises the secret.

## Pull requests

- One logical change per pull request.
- Say what you changed and why. If you fixed a defect, say how you reproduced it.
- State what you verified, and how. "Tests pass" is only useful if you ran them.
- If a claim in the README stops being true because of your change, update the
  README in the same pull request.

By contributing you agree that your contributions are licensed under the same
terms as this repository, and you confirm you have the right to grant that
licence.

## Conduct

Participation is governed by [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md).
Vulnerabilities go to [SECURITY.md](SECURITY.md), never to a public issue.
