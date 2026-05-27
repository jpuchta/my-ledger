---
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
---

## Why this stack

A solo bookkeeper building a privacy-first personal finance web app in 5 after-hours weeks needs a battle-tested, agent-friendly starter that bundles auth, a relational database, and edge deployment without forcing third-party credential sharing. The 10x Astro Starter is the recommended default for `(web-app, js)` and clears all four agent-friendly quality gates: explicitly typed (TypeScript + Zod), convention-based (Astro file-based routing), extremely popular in training data (React + Astro), and well-documented. Supabase covers PostgreSQL (complex transaction reconciliation logic and relational ledger data fit naturally in a relational schema), Google OAuth (FR-001–003), and row-level security — the exact auth shape the PRD requires. Cloudflare Pages provides the edge deploy with no cold-start penalty for an interactive ledger UI. GitHub Actions with auto-deploy-on-merge suits a solo after-hours shipping cadence. Auth is the only technology-forcing feature detected in PRD FRs; payments, realtime, AI, and background jobs are all out of scope.
