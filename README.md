# script-injection-target

Intentionally vulnerable lab target for [TurlaFSB/cicd-compromise-lab](https://github.com/TurlaFSB/cicd-compromise-lab) scenario 02 — **script injection via issue title**.

**Do not use real secrets here. This repo is disposable. Every secret below is a dummy value.**

## The vulnerability

`.github/workflows/triage.yml` triggers on `issues: opened` and interpolates `${{ github.event.issue.title }}` directly into a `run:` shell step:

```yaml
run: |
  echo "Thanks for opening: ${{ github.event.issue.title }}"
```

GitHub Actions expands `${{ }}` expressions as a literal string substitution **before** the shell ever parses the line. An issue title is attacker-controlled text. If it contains shell metacharacters — backticks, `$()`, `;`, `&&` — those characters land in the generated script verbatim, not as a quoted string. The runner then executes whatever shell was injected, with this job's `GITHUB_TOKEN` and secrets in its environment.

This is the pattern described in [OWASP CICD-SEC-4](https://owasp.org/www-project-top-10-ci-cd-security-risks/CICD-SEC-04-Poisoned-Pipeline-Execution) / [CICD-SEC-1](https://owasp.org/www-project-top-10-ci-cd-security-risks/CICD-SEC-01-Insufficient-Flow-Control-Mechanisms), and has caused real incidents (notably a 2021 academic study found hundreds of public repos vulnerable to this exact pattern).

## Required repo secret

| Name | Value |
|---|---|
| `DUMMY_CLOUD_KEY` | any placeholder string, e.g. `dummy-cloud-key-not-real-12345` |

Never a real credential — it only proves the attack path can reach a secret.

## Status

Vulnerable baseline for scenario 02. See the main repo for the attack script, evidence, and hardened fix.
