---
bootstrapped_at: 2026-05-27T17:45:00Z
starter_id: 10x-astro-starter
starter_name: "10x Astro Starter (Astro + Supabase + Cloudflare)"
project_name: my-ledger
language_family: js
package_manager: npm
cwd_strategy: git-clone
bootstrapper_confidence: first-class
phase_3_status: ok
audit_command: "npm audit --json"
---

## Hand-off

```yaml
starter_id: 10x-astro-starter
package_manager: npm
project_name: my-ledger
hints:
  language_family: js
  team_size: solo
  deployment_target: cloudflare-pages
  ci_provider: github-actions
  ci_default_flow: auto-deploy-on-merge
  bootstrapper_confidence: first-class
  path_taken: standard
  quality_override: false
  self_check_answers: null
  has_auth: true
  has_payments: false
  has_realtime: false
  has_ai: false
  has_background_jobs: false
```

### Why this stack

A solo bookkeeper building a privacy-first personal finance web app in 5 after-hours weeks needs a battle-tested, agent-friendly starter that bundles auth, a relational database, and edge deployment without forcing third-party credential sharing. The 10x Astro Starter is the recommended default for `(web-app, js)` and clears all four agent-friendly quality gates: explicitly typed (TypeScript + Zod), convention-based (Astro file-based routing), extremely popular in training data (React + Astro), and well-documented. Supabase covers PostgreSQL (complex transaction reconciliation logic and relational ledger data fit naturally in a relational schema), Google OAuth (FR-001–003), and row-level security — the exact auth shape the PRD requires. Cloudflare Pages provides the edge deploy with no cold-start penalty for an interactive ledger UI. GitHub Actions with auto-deploy-on-merge suits a solo after-hours shipping cadence. Auth is the only technology-forcing feature detected in PRD FRs; payments, realtime, AI, and background jobs are all out of scope.

## Pre-scaffold verification

| Signal      | Value                                               | Severity | Notes                                                   |
| ----------- | --------------------------------------------------- | -------- | ------------------------------------------------------- |
| npm package | not run                                             | n/a      | cmd_template starts with `git clone`; npm check skipped |
| GitHub repo | przeprogramowani/10x-astro-starter last pushed 2026-05-17 | fresh    | from card.docs_url                                      |

## Scaffold log

**Resolved invocation**: `git clone https://github.com/przeprogramowani/10x-astro-starter .bootstrap-scaffold && cd .bootstrap-scaffold && npm install`
**Strategy**: git-clone
**Exit code**: 0
**Files moved**: 20 items (files + directories)
**Conflicts (.scaffold siblings)**: `README.md.scaffold`, `.vscode/settings.json.scaffold`
**.gitignore handling**: moved silently (no prior .gitignore in cwd)
**.bootstrap-scaffold cleanup**: deleted

## Post-scaffold audit

**Tool**: `npm audit --json`
**Initial summary (pre-fix)**: 0 CRITICAL, 1 HIGH, 9 MODERATE, 0 LOW
**Post-fix summary**: 0 CRITICAL, 0 HIGH, 5 MODERATE, 0 LOW
**Direct vs transitive (post-fix)**: 1/0 direct of total 5 moderate (direct: `@astrojs/check`)

### Remediation taken (post-bootstrap cleanup)

Ran `npm audit fix` (non-breaking only): 4 packages added, 1 removed, 13 changed. The HIGH `devalue` advisory and 4 of the 9 prior MODERATEs (`@cloudflare/vite-plugin`, `miniflare`, `wrangler`, `ws`) are cleared.

`npm audit fix --force` was **not** taken — it would downgrade `@astrojs/check` from `^0.9.8` to `0.9.2` (semver-major regression of the toolchain). The remaining `@astrojs/check` chain is accepted as-is; it is dev-only (lint/type-checking) and the upstream advisory affects only the YAML language server's behaviour on deeply-nested YAML inputs, which the project does not feed it.

#### CRITICAL findings

None.

#### HIGH findings

None (the pre-fix `devalue` finding was cleared by `npm audit fix`).

#### MODERATE findings (all in the `@astrojs/check` toolchain, dev-only, accepted as-is)

- **@astrojs/check** — moderate, isDirect: true; via `@astrojs/language-server` → `volar-service-yaml` → `yaml-language-server` → `yaml`. Fix would downgrade to `@astrojs/check@0.9.2` (semver-major break); not taken.
- **@astrojs/language-server** — moderate, isDirect: false; via `volar-service-yaml`.
- **volar-service-yaml** — moderate, isDirect: false; via `yaml-language-server`.
- **yaml** — moderate, isDirect: false; GHSA-48c2-rrv3-qjmp "Stack Overflow via deeply nested YAML collections", CVSS 4.3, range `yaml >= 2.0.0 < 2.8.3`.
- **yaml-language-server** — moderate, isDirect: false; via `yaml`.

#### LOW / INFO findings

None.

## Hints recorded but not acted on

| Hint                    | Value               |
| ----------------------- | ------------------- |
| bootstrapper_confidence | first-class         |
| quality_override        | false               |
| path_taken              | standard            |
| self_check_answers      | null                |
| team_size               | solo                |
| deployment_target       | cloudflare-pages    |
| ci_provider             | github-actions      |
| ci_default_flow         | auto-deploy-on-merge |
| has_auth                | true                |
| has_payments            | false               |
| has_realtime            | false               |
| has_ai                  | false               |
| has_background_jobs     | false               |

These hints were read into the run's working memory and copied here for the audit trail. No automated action was taken on any of them in v1. `deployment_target`, `ci_provider`, `ci_default_flow`, and all `has_*` flags are deferred to the future M1L4 skill ("Memory Architecture") which will generate `AGENTS.md`, `CLAUDE.md`, and CI workflow files.

## Next steps

Next: a future skill will set up agent context (CLAUDE.md, AGENTS.md). For now, your project is scaffolded and verified — happy hacking.

Useful manual steps in the meantime:
- `git init` (if you have not already) to start your own repo history.
- Review any `.scaffold` siblings the conflict policy created and decide which version of each file to keep: `README.md.scaffold` and `.vscode/settings.json.scaffold`.
- Address audit findings per your project's risk tolerance — the full breakdown is in this log.
