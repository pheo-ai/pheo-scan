# Pheo Scan

**Bring a skill. Get the verdict.** Before your agent reads a SKILL.md,
an AGENTS.md, a rule file, or an MCP config, see what it would actually
*do* — classified by consequence, deterministically, on your machine.

```bash
pip install pheo-oats
cd your-repo
oats scan
```

That reads the agent instructions here — skill files, rule files,
markdown — extracts every shell command they tell an agent to run, and
classifies each one through the same resolver that powers the
[pheo.ai](https://pheo.ai) index and the OATS runtime gate:

```
  34 entitlement types · 13 can never run unattended

  shell_remote_exec   curl -fsSL https://get.example.sh | bash
  secret_change       export AWS_SECRET_ACCESS_KEY=...
  deploy              gcloud run deploy api --project prod
  read_files          cat package.json
```

Nothing is installed, nothing is enforced, nothing leaves the machine.
A finding is not a vulnerability — the same command is routine at one
company and forbidden at the next. The scan tells you what the file
*asks for*, so the answer is yours rather than a default.

## In CI

Fail a pull request that adds agent instructions asking for something
irreversible:

```yaml
# .github/workflows/pheo-scan.yml
name: Pheo Scan
on: [pull_request]
jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pheo-ai/pheo-scan@main
```

The action runs `oats scan --strict`: exit non-zero when any command
falls in a never-graduating class — deleting repositories, touching
credentials, piping downloads into shells — so the diff gets a human
before it gets merged.

Machine-readable output for your own tooling:

```bash
oats scan --json | jq '.classes[] | select(.graduates == false)'
```

## What runs underneath

The resolver is deterministic — no model in the decision path, the
same answer every time, in microseconds. It is position-aware (it
tells *running* `rm -rf` apart from *mentioning* it), reads through
quoting tricks, and ships as compiled binaries inside the
[`pheo-oats`](https://pypi.org/project/pheo-oats/) wheels. The public
index it powers has classified 116,000+ skills, MCP servers and rule
files: [pheo.ai](https://pheo.ai).

## Sisters

- [open-agent-trust-system](https://github.com/pheo-ai/open-agent-trust-system) —
  the OATS spec: consequence classes, receipts, the booth protocol.
- [agent-passport](https://github.com/pheo-ai/agent-passport) — sign
  and verify agent HTTP traffic; scanning is what a file *asks*,
  the passport is who an agent *is*.

Scanning tells you what a file can do. Governing decides whether it
may. `oats quickstart` gives you both.
