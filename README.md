# my-ledger

[![CI](https://github.com/jpuchta/my-ledger/actions/workflows/ci.yml/badge.svg?branch=main)](https://github.com/jpuchta/my-ledger/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Node](https://img.shields.io/badge/node-22.14.0-339933?logo=nodedotjs&logoColor=white)](.nvmrc)
[![TypeScript](https://img.shields.io/badge/TypeScript-5-3178C6?logo=typescript&logoColor=white)](https://www.typescriptlang.org/)
[![Astro](https://img.shields.io/badge/Astro-6-FF5D01?logo=astro&logoColor=white)](https://astro.build/)
[![Dependabot](https://img.shields.io/badge/Dependabot-enabled%20%28npm%20%2B%20actions%29-025E8C?logo=dependabot&logoColor=white)](.github/dependabot.yml)

A privacy-first personal-finance web app for a household bookkeeper. Imports arbitrary bank-supplied CSVs, lets the user define their own category taxonomy, keeps financial data under the user's control (no third-party aggregators), and produces a monthly CSV report as the artifact the household reviews.

The product is currently in pre-MVP scaffold state. See [context/foundation/prd.md](context/foundation/prd.md) for the full product requirements document.

## Quick links

- [DEVELOPMENT.md](DEVELOPMENT.md) — setup, tech stack, scripts, local Supabase, deployment, CI.
- [AGENTS.md](AGENTS.md) — guidance for AI agents working in this repo (commands, architecture, conventions).
- [context/foundation/prd.md](context/foundation/prd.md) — product requirements (vision, user stories, FRs, NFRs).
- [LICENSE](LICENSE) — MIT.

## Attribution

Bootstrapped from the [10x Astro Starter](https://github.com/przeprogramowani/10x-astro-starter) (Astro + Supabase + Cloudflare). See [context/foundation/tech-stack.md](context/foundation/tech-stack.md) and [context/changes/bootstrap-verification/verification.md](context/changes/bootstrap-verification/verification.md) for the bootstrap audit trail.
