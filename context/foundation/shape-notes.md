---
project: "My Ledger"
context_type: greenfield
created: 2026-05-25
updated: 2026-05-25

product_type: web-app
target_scale:
  users: small
  qps: low
  data_volume: small
timeline_budget:
  mvp_weeks: 5
  hard_deadline: null
  after_hours_only: true

checkpoint:
  current_phase: 8
  phases_completed: [1, 2, 3, 4, 5, 6, 7]
  gray_areas_resolved:
    - topic: "primary persona scope"
      decision: "solo for MVP; couple/shared ledger recorded as explicit v2+ direction"
    - topic: "pain category"
      decision: "workflow friction + data trapped (per-bank CSV format divergence) + correctness anxiety (missing / duplicated / mistyped transactions)"
    - topic: "scenario #2 (couple sync)"
      decision: "out of MVP scope; solve solo-tracking first"
    - topic: "insight (why existing tools fail)"
      decision: "the gap is the combination: unreliable bank API coverage (esp. Polish banks), forced cloud sync / credential surrender, locked-in category taxonomies, and report formats that are dashboards rather than household artifacts"
    - topic: "access model"
      decision: "passwordless authentication (magic link / OAuth); real server-backed identity"
    - topic: "role model"
      decision: "owner + viewer roles defined in data model from MVP; only owner reachable in MVP scope (no invite flow); groundwork for v2 couple-sharing"
    - topic: "MVP flow scope (initial)"
      decision: "minimal CSV import (map columns per file; no saved schemas); manual categorization post-import; month-only reports; no Settings page; no confirmation email. (Note: duplicate detection and Internal-tagging were re-added in Phase 5 as the domain rule \u2014 see below.)"
    - topic: "MVP timeline"
      decision: "5 weeks after-hours; sustained-effort cost acknowledged"
    - topic: "FR Socrates revisions"
      decision: "FR-006 revised (inline category picker + modal for full-row edits); FR-007 revised (row-level delete with confirmation); FR-008/FR-009/FR-012/FR-013 dropped (no dedicated Categories/Accounts pages \u2014 management via dialogs reachable from the inline pickers); FR-010/FR-014 surface revised; FR-011 revised (drop predefined 'Other'); FR-015 revised (file picker only, no drag-and-drop)"
    - topic: "domain rule (business logic)"
      decision: "reconciliation rule at ledger ingress: exact (date+amount+account) match \u2192 candidate duplicate; opposite-sign cross-account within 5 calendar days \u2192 candidate transfer pair; asymmetric UX (auto-action with post-import review at CSV import; warn-then-confirm at manual add); always reversible"
    - topic: "product framing"
      decision: "product_type: web-app; target_scale.users: small; no hard deadline; after-hours-only"
  frs_drafted: 22
  quality_check_status: accepted
---

# Shape notes — My Ledger

Seed idea (from `context/foundation/project-idea-v0.1.md`, 2026-05-25):

> Monitorowanie i kategoryzowanie wydatków domowych w przypadku posiadania wielu kont bankowych
> oraz wielu kart płatniczych lub kredytowych jest uciążliwe, gdyż wymaga ręcznego przepisywania
> transakcji z wykazów, które każdy z banków dostarcza w innym formacie, a także ręcznego
> przypisywania kategorii do każdej transakcji.

This file is the input to `/10x-prd`. Sections below anticipate the 10 greenfield PRD sections.

## Vision & Problem Statement

A private person tracking household finances with multiple bank accounts and multiple payment/credit cards loses roughly two hours every first weekend of the month: they manually transcribe transactions from each bank's separate statement into one place, manually assign categories, and finish the work without confidence that they captured every transaction or avoided typos and duplicates. The monthly review is both expensive and uncertain.

Existing personal-finance tools fail this user along several axes at once: aggregators that pull from banks via API do not reliably cover the user's banks (e.g., Polish retail banks) and require surrendering credentials or syncing financial data to a third-party cloud; tools that work locally typically force their own category taxonomy rather than letting the user define their own; and their monthly outputs are dashboards rather than the report-artifact the household actually wants to read. The gap is the combination — there is no tool that ingests arbitrary bank-supplied CSVs, lets the user define categories on their own terms, keeps data under the user's control, and produces a monthly report as the artifact the household reviews.

## User & Persona

**The household bookkeeper.** A private person responsible for tracking and categorizing their household's spending. Holds multiple bank accounts (debit) and one or more payment/credit cards across at least two banks. Technologically comfortable enough to export CSV statements from each bank's online portal. Privacy-aware: prefers their financial data stay under their own control rather than synced to a third-party aggregator.

**The moment they reach for this product:** the first weekend of every month, sitting down to reconcile the previous month's spending and produce a single monthly report that summarizes what was spent and where.

For MVP, the bookkeeper is **one person — solo ledger ownership**. A future iteration extends the product to two partners sharing a household ledger; this is captured below under `## Forward: post-MVP direction`.

## Access Control

Each household bookkeeper has a real, server-backed identity. Authentication is **passwordless** — magic link by email or OAuth — with no user-managed passwords.

The data model defines two roles from day one:

- **Owner** — full read/write access to a ledger. Creates the ledger, owns its data, and is the only role that can mutate transactions, categories, CSV schemas, or any other ledger content.
- **Viewer** — read-only access to a ledger.

**MVP scope: only the Owner role is reachable.** There is no invitation flow, no second user on any ledger, and no UI to assign a Viewer. The Viewer role exists in the data model and authorization layer as groundwork for the v2+ couple-sharing direction, so v2 is a feature flip rather than a schema migration.

Unauthenticated requests to ledger-bearing routes are rejected and the user is routed to sign-in.

## Success Criteria

### Primary

- The household bookkeeper can complete a full month-end session end-to-end in a single sitting: sign in with Google, import one CSV from one bank account (mapping columns at import time), import a second CSV from a different bank account with a different column shape, manually assign categories to the imported transactions, and produce/export a monthly CSV report for one chosen calendar month — including months whose statements are split across files (e.g., two CSVs whose date ranges overlap with each other and do not align to month boundaries).

### Secondary

- (None explicitly chosen — the primary outcome is sufficient. Bonus indicators may be revisited post-MVP.)

### Guardrails

- **No transaction loss.** A transaction the user enters or imports never silently disappears. Any failure surface (validation error, import failure, network error) preserves the user's pending input or import payload until they explicitly discard or retry.
- **Privacy of financial data.** Transaction content (amount, date, description, comment, category, account) is never sent to third-party analytics, telemetry, advertising, or aggregator services. The data lives where the user's ledger lives, and nowhere else.
- **Category / account deletion does not silently mutate historical data.** Deletion is implemented as soft-delete (FR-010, FR-014): a deleted category or account is hidden from pickers but historical transactions retain their reference to it (read-only). Historical transactions never silently become uncategorized, change account assignment, or disappear because of a category- or account-management action.
- **Monthly report determinism (soft).** Running the export for the same month twice on unchanged data produces the same file.

## User Stories

### US-01: Household bookkeeper completes a full month-end session end-to-end

- **Given** a registered household bookkeeper who has not yet logged transactions for the just-ended calendar month
- **When** they sign in via Google, import a CSV statement from one bank into account A (mapping columns at import), import a second CSV statement from a different bank into account B (whose column order, header names, and date format differ from the first CSV, and whose date range overlaps with the first CSV but does not align to month boundaries), review the post-import panel listing what the application auto-skipped as duplicates and auto-tagged as transfer pairs and accept those decisions, assign categories to the remaining transactions (using existing categories and creating new ones on the fly as needed), and finally open Reports, pick the just-ended calendar month, and click Export
- **Then** they receive a downloaded CSV file containing exactly the transactions from that calendar month — across both accounts, excluding any categorized as "Internal" (including those the application auto-tagged as transfer pairs) — in the project's defined monthly-report CSV format

#### Acceptance Criteria
- Both CSVs are imported successfully even though their column orders, header names, and date formats differ.
- Imported rows that exactly match (date + amount + account) of an existing transaction are auto-skipped and shown in the post-import review panel with a per-row force-import action.
- Pairs of imported rows whose opposite-signed amounts match across two different accounts within 5 days (whether one is an existing transaction or both are new) are auto-tagged "Internal" and shown in the post-import review panel with a per-pair reverse action.
- Transactions outside the chosen calendar month are excluded from the report.
- Transactions categorized as "Internal" (including auto-tagged transfer pairs) are excluded from the report.
- Categories newly created during this session are immediately available for subsequent transactions in the same session.
- The exported file is a valid CSV in the project's documented monthly-report format.

## Functional Requirements

### Authentication & identity
- FR-001: The household bookkeeper can sign up for a new account by completing a Google OAuth round-trip. Priority: must-have
  > Socrates: Counter-argument considered: "Hard-locking to Google creates onboarding friction for non-Google users and is single-IdP vendor risk." Resolution: kept; Google OAuth is acceptable for the single-user MVP; broaden providers post-MVP if needed.
- FR-002: The household bookkeeper can sign in to an existing account by completing a Google OAuth round-trip. Priority: must-have
  > Socrates: Same as FR-001 (Google-only). Resolution: kept; broaden post-MVP if needed.
- FR-003: The household bookkeeper can sign out, after which any further request to a ledger-bearing route returns them to sign-in. Priority: must-have
  > Socrates: Counter-argument considered: "Explicit sign-out is a vestigial security ritual; session expiry handles it." Resolution: kept; explicit sign-out matters on shared devices and is a trust signal for a finance app.

### Ledger — transactions
- FR-004: The household bookkeeper can manually add a transaction with the fields: date (defaults to today, picked from a date picker), amount (numeric with two decimal places; signed — positive = expense, negative = reimbursement — with the form displaying which interpretation applies based on the sign), category (autocomplete from existing categories; typing a brand-new name and submitting creates the category and assigns it to the transaction), account (same behavior as category), description (free text, optional), comment (free text, optional). Priority: must-have
  > Socrates: Counter-argument considered: "On-the-fly category/account creation from the transaction form muddles transaction entry with taxonomy management; restricting on-the-fly creation to the dedicated pages keeps the form simpler." Resolution: kept; on-the-fly creation IS the value — forcing the user to context-switch to add a missing category mid-entry defeats the purpose.
- FR-005: The household bookkeeper can view all transactions in their ledger as a table. Priority: must-have
  > Socrates: Counter-argument considered: "All-transactions-ever is fine at 100 rows, painful at 5,000 rows; MVP should include pagination or filtering." Resolution: kept; MVP scale is small (one user, ~1 year of data max). Pagination/filtering work is deferred to v1.5.
- FR-006: The household bookkeeper can change a transaction's category directly from its ledger row via an inline category picker (no modal). For any other field edit, they click the row to open a modal containing the same fields as the Add form, pre-filled with the transaction's current values, and edit any field. Priority: must-have
  > Socrates: Counter-argument considered: "A modal is heavyweight for the most common post-import action (bulk-categorizing imported rows); a row-inline editor would be much faster." Resolution: revised — added inline category picker on each row (no modal needed just to change category); modal remains for full-row edits.
- FR-007: The household bookkeeper can delete a transaction directly from its ledger row (via an inline delete affordance), with an explicit confirmation step before the delete commits. Priority: must-have
  > Socrates: Counter-argument considered: "Delete buried in the edit modal is hard to find and easy to click by accident." Resolution: revised — moved delete out of the modal to a row-level affordance with an explicit confirmation step.
- FR-022: Transactions categorized as "Internal" are visually distinguished (e.g., greyed) in the ledger view. Priority: must-have
  > Socrates: Counter-argument considered: "Visual greying creates a 'second-class transaction' signal that breaks the mental model that Internal is just-another-category." Resolution: kept; the greying matches Internal's role as the only category with a system-wide rule (excluded from reports), and the visual treatment communicates that.

### Categories
- FR-010: The household bookkeeper can rename or soft-delete a category via a "Manage categories…" dialog accessible from the inline category picker (which appears in the ledger row picker and in the Add/Edit transaction form). The dialog lists all (non-soft-deleted) categories with rename and soft-delete actions. A soft-deleted category is hidden from the picker and from the dialog, but historical transactions retain their reference to it (read-only). It cannot be assigned to new transactions. Priority: must-have
  > Socrates: Counter-argument considered: "Soft-delete is a half-measure: users won't understand that the category is 'hidden' but still appears in old transactions." Resolution: kept; soft-delete is the right semantics for a finance ledger — history is sacred. Surface revised: no dedicated Categories page (FR-008/FR-009 dropped); rename and soft-delete live in a Manage-categories dialog reachable from the inline picker.
- FR-011: One category ("Internal") is present by default in every new ledger and cannot be renamed or soft-deleted. It is system-load-bearing: the rule that excludes Internal transactions from reports (see Reports section) depends on it. Priority: must-have
  > Socrates: Counter-argument considered: "Pre-seeding categories tempts users into the lazy-catch-all habit ('Other') and most personal-finance tools regret pre-seeding." Resolution: revised — dropped the "Other" predefined category; users build their own taxonomy from zero. "Internal" stays because the report-exclusion rule depends on it.

(FR-008 and FR-009 were dropped during the Socrates round: no dedicated Categories page; categories are added via on-the-fly creation in the transaction form and CSV import, and managed via the Manage-categories dialog described in FR-010.)

### Accounts
- FR-014: The household bookkeeper can rename or soft-delete an account via a "Manage accounts…" dialog accessible from the inline account picker (which appears in the Add/Edit transaction form and in the CSV-import account picker). The dialog lists all (non-soft-deleted) accounts with rename and soft-delete actions. A soft-deleted account is hidden from the picker and from the dialog, but historical transactions retain their reference to it (read-only). It cannot be assigned to new transactions and cannot be used as a CSV-import target. Priority: must-have
  > Socrates: Same shape as FR-010 (soft-delete vs block/migrate). Resolution: kept soft-delete; surface revised to the Manage-accounts dialog reachable from the inline picker. By parity with categories, no dedicated Accounts page (FR-012/FR-013 dropped).

(FR-012 and FR-013 were dropped during the Socrates round: no dedicated Accounts page; accounts are added via on-the-fly creation in the transaction form and CSV import, and managed via the Manage-accounts dialog described in FR-014.)

### CSV import
- FR-015: The household bookkeeper can upload a CSV file via a standard file picker. Priority: must-have
  > Socrates: Counter-argument considered: "Two upload patterns (drag-and-drop + picker) means twice the UX work and twice the testing." Resolution: revised — file picker only in MVP; drag-and-drop deferred to v1.5+.
- FR-016: After upload, the household bookkeeper sees a preview of the CSV: its detected column headers and a table of its rows. Priority: must-have
  > Socrates: Counter-argument considered: "A full-row preview is heavy to render; first-N-rows is enough to confirm column mapping." Resolution: kept; full preview gives the user confidence they're importing what they intended.
- FR-017: The household bookkeeper can assign each CSV column a role: date / amount / description / ignore. Priority: must-have
  > Socrates: Counter-argument considered: "Bank statements with separate debit/credit columns can't be imported under signed-amount-only." Resolution: kept; the user reports not having encountered debit/credit-split statements in Polish banks. Signed-amount only in MVP; debit/credit-split would be a v1.5 enhancement if it ever becomes necessary.
- FR-018: The household bookkeeper picks a single target account (existing or new; created via autocomplete-with-on-the-fly-creation as in FR-004) into which all rows from this CSV will be imported. Priority: must-have
  > Socrates: Counter-argument considered: "Card-network aggregate statements may contain rows from multiple sub-accounts." Resolution: kept; one-account-per-import matches the realistic input (per-bank CSV exports) for the MVP user.
- FR-019: On submit, all CSV rows are imported as transactions with the mapped fields, the chosen account, and no category (subject to FR-023 and FR-024 below, which may auto-skip or auto-tag specific rows). Priority: must-have
  > Socrates: Counter-argument considered: "A 'default category for this import' field would collapse 95% of post-import categorization into one decision." Resolution: kept; per-row categorization gives full control and trains the user's taxonomy organically. The inline category picker (FR-006) keeps post-import categorization fast.
- FR-023: At CSV import, the application flags any row whose (date + amount + account) exactly matches an existing transaction or another row in the same import batch as a duplicate and auto-skips it. The post-import review panel lists every skipped row alongside the matching reference transaction and provides a per-row "force-import this row" action that reinstates the skipped row as a normal transaction. Priority: must-have
  > Socrates: Counter-argument considered: "Exact (date + amount + account) match is brittle — banks sometimes post the same transaction with a 1-day date adjustment, and the user will see false negatives." Resolution: kept; exact match is a v1 floor; relaxation to a date-window-tolerant match is a v1.5 enhancement.
- FR-024: At CSV import, the application identifies candidate transfer pairs as any two transactions on two different (non-soft-deleted) accounts whose amounts are +X and −X for some X > 0 and whose dates are within five calendar days of each other; matches may pair a new import row with an existing ledger row or two rows within the same import batch. The application auto-tags both halves of every candidate pair with the category "Internal". The post-import review panel lists every auto-tagged pair with a per-pair "reverse this pair" action that untags both back to no category. Priority: must-have
  > Socrates: Counter-argument considered: "Opposite-sign + cross-account + 5-day window can produce false positives (e.g., a vendor refund landing on a different account around the same time as an unrelated debit). Auto-tagging may silently miscategorize." Resolution: kept; the post-import review panel surfaces every pair the rule found with a per-pair reverse action and full traceability, so misjudgments are recoverable. The cross-account + opposite-sign constraint filters most false positives in normal usage.

### Manual-entry reconciliation
- FR-025: At manual transaction add, if the submitted transaction's (date + amount + account) exactly matches an existing transaction, the form blocks submit and shows a warning naming the existing transaction with two actions: "save anyway" (which commits the new transaction unchanged) or "cancel" (which returns the user to the form for edits). Priority: must-have
  > Socrates: Counter-argument considered: "Manual entry of a real duplicate is rare; the warning will mostly fire on legitimate similar-amount transactions and annoy the user." Resolution: kept; the warning includes the matching transaction so the user can verify whether it's a real duplicate; the warning is dismissable in one click on rare collisions.
- FR-026: At manual transaction add, if the submitted transaction qualifies as one half of a candidate transfer pair under the rule in FR-024, the form warns the user before commit, naming the matching existing transaction and offering two actions: "save and tag both as Internal" (which commits the new transaction and updates the matching existing one's category to Internal) or "save as-is" (which commits the new transaction without altering the matching one). Priority: must-have
  > Socrates: Counter-argument considered: "Two writes from one form (the new transaction + a category update on an existing one) is a hidden side effect; users may not notice the other transaction was mutated." Resolution: kept; the warning explicitly names the matching transaction the user is about to mutate, the action label makes the side-effect explicit ("save and tag both as Internal"), and the change is reversible by editing either transaction's category afterwards.

### Reports
- FR-020: The household bookkeeper can pick a calendar month and see the filtered list of transactions for that month, with "Internal" category transactions excluded. Priority: must-have
  > Socrates: Counter-argument considered: "A 'Show Internal' toggle would let users verify their Internal tagging on the Reports view itself." Resolution: kept; Internal exclusion is THE rule of the Reports view. Verification of Internal tagging happens in the main Ledger view, not in Reports.
- FR-021: The household bookkeeper can export the report for the chosen month as a CSV file in the project's documented monthly-report format. Priority: must-have
  > Socrates: Counter-argument considered: "CSV-only excludes printable / emailable / non-spreadsheet uses; PDF or HTML would broaden usefulness." Resolution: kept; CSV is the seed's defined format. The report-as-artifact is meant to be opened in a spreadsheet. Other formats are v2.

## Business Logic

**Rule (one sentence):** When a new transaction enters the ledger — whether via CSV import or manual entry — the application reconciles it against the existing ledger and against other transactions in the same import batch, treating rows that exactly match (date + amount + account) of an existing transaction as candidate duplicates, and rows whose opposite-signed amount matches another transaction on a different account within a 5-calendar-day window as candidate transfer pairs.

The rule consumes the existing ledger (all non-soft-deleted transactions) and the new transaction(s) being submitted. For each new transaction the rule yields one of three outcomes: a normal accept; a duplicate flag (auto-skipped at CSV import, blocked-pending-confirmation at manual add); or a transfer-pair flag (both halves auto-tagged "Internal" at CSV import, prompt-with-default at manual add).

The user encounters the rule asymmetrically. During CSV import they see a post-import review panel listing what the application did — every skipped duplicate and every auto-tagged transfer pair — with per-row reverse actions (force-import the skipped row, untag the pair). During manual entry the form warns the user before save when the rule fires, naming the matching existing transaction and offering "save anyway / cancel" (for a duplicate) or "save and tag both as Internal / save as-is" (for a transfer-pair candidate).

Every decision the rule made is reversible: a force-imported skip becomes a normal transaction, an auto-Internal-tagged pair can be untagged or re-categorized via the inline category picker on either side.

## Non-Functional Requirements

- Transaction content (amount, date, description, comment, category, account) is never transmitted to any third party not directly serving the user's own ledger. No analytics, advertising, or aggregator services receive transaction data.
- Transaction data persisted at rest is protected against casual operator-side inspection: a database backup or storage snapshot that leaves the deployment environment is not readable as cleartext personally-identifiable transaction content.
- A transaction the user has deleted remains recoverable through a user-facing surface for at least a short window after the delete commits; the product does not hard-delete a transaction on the first user action.
- A successfully committed transaction (entered manually or imported via CSV) survives a server restart, a browser refresh, and a user-session expiry without loss.
- Amount values are displayed and parsed using two decimal places with the user's locale conventions (comma or period as the decimal separator). CSV import accepts dates expressed in the user's locale conventions, covering at minimum the common patterns DD.MM.YYYY, YYYY-MM-DD, and MM/DD/YYYY.

## Non-Goals

The MVP explicitly does NOT do any of the following. Each is recorded so it cannot creep back in without an explicit re-shape.

- **Importing non-CSV formats** (PDF, XLSX, OFX, QIF). CSV is the only supported import format. (Seed-explicit.)
- **Maintaining account balances or running totals.** The product tracks individual transactions; it does not compute, reconcile, or display per-account balances. (Seed-explicit.)
- **Migrating historical data when the category taxonomy changes.** Soft-delete and rename do not retroactively reshape past transactions; categories are decisions captured at the moment of categorization. (Seed-explicit.)
- **Automatic categorization** (ML, NLP, or rule-based on description). The user categorizes every transaction manually. Category suggestions, learning, and pre-population are out of MVP. (Seed-explicit.)
- **Native mobile application.** Web only for v1. (Seed-explicit.)
- **Bank API integrations, aggregator integrations, or screen-scraping.** CSV import is the only channel by which transactions enter the ledger. Aligns with the user's privacy stance (data stays under user control; no credentials surrendered to third parties).
- **Budgeting, variance analysis, forecasting, or financial planning.** The product describes past spending; it does not project, predict, prescribe, or compare to a budget.
- **Offline-first functionality.** The product requires a network connection. PWA / installable / works-without-internet are out of scope.
- **Multi-user collaboration on a single ledger.** Each ledger has exactly one Owner in MVP; the Viewer role exists in the data model only and is not reachable via UI. (Phase 2 decision.)
- **Any export channel beyond user-initiated CSV download.** No email-the-report, no public share-link, no integration with cloud-storage providers, no IFTTT-style webhooks.
- **Production SLA, uptime guarantee, or formal compliance certification.** No PCI, no SOC2, no formal GDPR certification beyond the baseline implied by the privacy NFR. Best-effort, self-hosted-friendly.

## Open Questions

(Working list — accumulates through phases. Final list lands in PRD.)

1. **What is the schema of the monthly-report CSV?** — Owner: user. Must be defined before MVP ships (FR-021 depends on it). Candidate columns: date, account, category, amount, description, comment — but the exact column order, header names, decimal/date formats, and whether to include a header row are TBD.
2. **What is the undo window for deleted transactions, and through what surface does the user recover?** — Owner: user. The NFR commits to "at least a short window" of recoverability and forbids hard-delete on first action, but does not pin the duration (a transient undo banner valid for ~30 seconds? a "recently deleted" view with a longer retention?). Resolve before or during MVP build.

## Timeline acknowledgment

Acknowledged on 2026-05-25: 5-week after-hours MVP. The user explicitly accepted that this requires sustained dedication over multiple weeks of evenings/weekends, with stretches where progress feels invisible. Acknowledgment recorded so the closing soft-gate (Phase 7) treats the cost as surfaced and accepted.

## Forward: post-MVP direction

(Informational — not part of the PRD schema. Surfaces commitments downstream skills may want.)

- **v2+ — couple-shared ledger.** Two partners with separate accounts share a single household ledger; reports remain compatible across partners. The MVP scope deliberately defers this. Scenario #2 from the seed (partners synthesizing across incompatible personal systems) is the v2 driver.
- **v1.5 — saved CSV schemas.** After the first successful import with a new column mapping, the app offers to save it as a named, reusable schema. Subsequent imports matching the schema's column shape apply it automatically. The seed's "Definiowanie schematów plików CSV" promise lands here, not in MVP.
- **v1.5 — drag-and-drop CSV upload.** MVP supports the standard file picker only (FR-015).
- **v1.5 — date-window-tolerant duplicate matching.** The MVP duplicate rule (FR-023) uses exact (date + amount + account) matching. A future relaxation could match within a small date window to catch bank-side date adjustments.
- **v2 — custom report date ranges.** Reports beyond a single calendar month (custom ranges, multi-month, year-to-date).
- **v1.5+ — pagination / filtering in the ledger view.** MVP renders all transactions as a single table (FR-005); becomes painful at scale.

