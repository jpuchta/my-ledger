---
date: 2026-05-28
status: decided
owner: jpuchta
scope: visual-identity
implementation_status: implemented
---

# Visual identity — Badger bookkeeper on Trąbka Borsuka

## Decision

`my-ledger` adopts a playful, mathematics-flavoured visual identity built around three intentionally-chosen elements:

1. **Persona — a bookkeeping Badger.** The mascot is a badger sitting at a desk with a ledger, quill, and abacus or pocket calculator. The choice is phonetic-symbolic: *ledger* and *badger* rhyme; "badger" in Polish is *borsuk*, which connects the persona to the mathematical motif below. The badger reads as friendly, methodical, slightly grumpy — the right tonal mix for a tool you reach for on the first Sunday of every month with a stack of bank CSVs to reconcile.
2. **Motif — Trąbka Borsuka (Borsuk's trumpet).** A topological space named after the Polish mathematician Karol Borsuk: a horn/trumpet-shaped construction of infinitely many circles of decreasing radius, glued so that traversal goes around-and-around without ever reaching an edge. The motif is symbolic: monthly bookkeeping is iterative — you go round the same loop every month, the loop tightens (categories settle, imports get smoother), but the work itself does not end. The horn shape doubles as a visual metaphor for funnelling raw bank-CSV data into a single monthly report.
3. **Palette — sandy yellow with light-brown accents.** Warm, earthy, low-stress. The background is sand; the trumpet/horn-circle accents are light brown (think dry-grass, kraft-paper, oak); the badger's stripes pick up a darker accent for contrast. No blacks, no harsh blues — the current `bg-cosmic` purple/indigo starter aesthetic is explicitly retired.

## Why these choices

- **Personality lowers the barrier to a chore.** Monthly reconciliation is friction the user pays in time and attention. A persona that makes the screen feel like a companion rather than a spreadsheet is a small but real motivator.
- **The mathematical motif is private joke + functional metaphor.** The user is the only audience for the first version; an in-joke is fine and the symbolism (loops that never reach the edge) genuinely maps to the work.
- **Warm palette is data-hygiene.** Financial UIs typically lean on cold blues to signal "trust" / "bank". For a private, self-hosted ledger the priority is not signalling trust to a stranger but minimising eye-strain across long categorisation sessions. Warm + low-saturation reads as calm.

## Palette (target values)

Approximate oklch coordinates to drop into `src/styles/global.css` when implementation lands. The exact values will need eye-test tuning against the generated assets:

| Token (proposed)                   | oklch (approx)               | Plain description                |
| ---------------------------------- | ---------------------------- | -------------------------------- |
| `--background`                     | `oklch(0.96 0.04 85)`        | warm sand, very light            |
| `--background-deep` (banner bg)    | `oklch(0.90 0.07 80)`        | slightly deeper sand             |
| `--foreground`                     | `oklch(0.25 0.04 60)`        | dark roasted brown, near-black   |
| `--accent` (trumpet, links)        | `oklch(0.65 0.12 65)`        | light brown / kraft              |
| `--accent-strong` (badger stripe)  | `oklch(0.40 0.08 50)`        | darker oak                       |
| `--muted`                          | `oklch(0.88 0.03 80)`        | muted sand                       |
| `--destructive`                    | keep current red             | error states unchanged           |

Dark mode is **out of scope for v1.5** — the warm palette is the identity, and a "dark badger" variant can come once the light theme has had real wear.

## Assets

| Asset                | Current state                                    | Target                                                                                                                                          | Notes                                                                                                                                                |
| -------------------- | ------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| `public/favicon.png` | 32×32 starter favicon (generic)                  | 32×32 (and ideally 192×192 PWA-ready) badger-head silhouette in sandy/brown                                                                     | Small enough that the trumpet motif cannot survive at 32px — favicon focuses on the badger, banner carries the trumpet                               |
| `public/template.png` → rename `public/banner.png` | 1492×470 starter banner                          | 1492×470 (or whatever Astro layout wants) banner: bookkeeping badger at a desk, with a Trąbka-Borsuka horn coiling out of the ledger book        | The horn is the visual hook; the badger is the eye anchor                                                                                            |
| `src/components/Welcome.astro`                     | "cosmic" purple/blue starter landing page        | Sandy-warm landing using the new palette, the new banner, and a short product blurb sourced from `context/foundation/prd.md`                     | Probably easier to rewrite from scratch than to retheme the existing cosmic markup                                                                   |
| `src/styles/global.css`                            | neutral light/dark + `bg-cosmic` dark-blue gradient utility | Replace `--background` / `--foreground` / `--accent` family with the palette above; retire `bg-cosmic` utility or repurpose as `bg-sand`         | Tailwind v4 picks up the changes automatically because the layout uses `bg-background` / `text-foreground` tokens                                    |

## Implementation status

Implemented 2026-05-28:

1. Generated badger favicon (`public/favicon.png`, 32×32) and PWA icon (`public/icon-192.png`, 192×192); banner at `public/banner.png` (1492×470). Retired `public/template.png`.
2. Palette swapped in `src/styles/global.css`; `bg-cosmic` replaced with `bg-sand`.
3. `Welcome.astro`, `Topbar.astro`, and auth/dashboard pages rethemed; product copy in Polish from the PRD.
4. Git LFS enabled for raster images via `.gitattributes`.
5. oklch values may still need eye-test tuning on a real screen.

## Out of scope for this change

- Dark mode (deferred until the light theme has worn in).
- Animation / interactivity on the banner (no horn-rotation gimmicks in v1).
- Internationalisation of any persona-related copy (the few badger references that surface in UI copy will stay in Polish to match the existing landing-page banner copy in [src/lib/config-status.ts](../../src/lib/config-status.ts)).
- Adoption of a logotype / wordmark beyond the existing "My Ledger" plain-text title.

## References

- [context/foundation/prd.md](../../foundation/prd.md) — product positioning the persona must serve.
- [src/styles/global.css](../../../src/styles/global.css) — the file the palette change will land in.
- [src/components/Welcome.astro](../../../src/components/Welcome.astro) — the landing page the rebrand most visibly affects.
- Karol Borsuk — Polish topologist (1905–1982); *Trąbka Borsuka* is one of several spaces named after him in shape theory.
