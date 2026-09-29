# Relocated CI workflows

This directory holds GitHub Actions workflows imported from
[`shubhamsaboo/awesome-llm-apps`](https://github.com/shubhamsaboo/awesome-llm-apps)
that could **not** be written to `.github/workflows/` during the import.

## Why

The import was performed by a GitHub App integration that does not hold the
`workflows` permission. GitHub rejects any push from such an app that creates or
updates a file under `.github/workflows/`:

```
remote: ! [remote rejected] (refusing to allow a GitHub App to create or update
workflow `.github/workflows/claude.yml` without `workflows` permission)
```

Rather than lose the content, the useful workflow was parked here.

## Files

| File | Upstream path | Status |
| --- | --- | --- |
| `skill-evals.yml` | `.github/workflows/skill-evals.yml` | **Kept** — worth enabling |
| — | `.github/workflows/claude.yml` | **Dropped** — upstream's `@claude` PR bot |

`claude.yml` was dropped outright because it drives `anthropics/claude-code-action`
using upstream's own `ANTHROPIC_API_KEY` repository secret. It does nothing in a
fork, so it was not worth relocating.

## Enabling `skill-evals.yml`

The body of `skill-evals.yml` is byte-identical to upstream apart from a leading
comment header, so no edits are required.

**Option A — GitHub web UI** (no extra permissions needed): create
`.github/workflows/skill-evals.yml` in the repo and paste in everything from the
`name: skill-evals` line downward.

**Option B — local push** with credentials that carry the `workflows` scope:

```bash
mkdir -p .github/workflows
tail -n +18 docs/ci/skill-evals.yml > .github/workflows/skill-evals.yml
git add .github/workflows/skill-evals.yml
git commit -m "ci: enable skill-evals workflow"
git push
```

**Option C — grant the permission.** Give the GitHub App the `workflows`
permission (or reconnect GitHub with a token that has the scope), then revert the
commit that removed the workflows.

## What the workflow does

It runs on pull requests and pushes to `main` that touch `agent_skills/**` or
`README.md`, and provides five checks:

- **evals** — runs every `agent_skills/evals/*/test_*.py` deterministic eval
- **lint** — validates each `agent_skills/*/SKILL.md` against the spec (strict)
- **security-scan** — scans all skills for supply-chain and injection patterns
- **trigger-routing** — confirms skill triggers clear near-miss cases
- **registry** — checks registry membership and README listings
