# CertVault beta — TEST-RESULTS.md

App: `index.html` (same folder). Built 2026-10-19 from `~/workspace/certvault-demo/certvault-v0.html`.

## What was tested (no real browser available to this builder — logic tests only)

### 1. Syntax check
- Extracted the `<script>` block from `index.html` and ran `node --check` (node v24.20.0): **OK, no errors.**

### 2. Structural checks against the built HTML (10/10 pass)
- No "DEMO DATA" banner, no "prototype build" / "working title" framing
- Honest beta label present: "CertVault beta — your data stays in your browser (localStorage). Export your CSV anytime."
- FormSubmit endpoint exact: `https://formsubmit.co/1c091997c234a390cf773be5fd40f9b5`
- Hidden `_subject` = "CertVault deadline reminder signup", `_template` = "table"
- "Mandate in effect" copy present; no "cloud sync" claims; navy/slate design + viewport meta kept

### 3. Logic tests (`/tmp/cvtest/test.js`, 41 assertions, 41 pass)
Core is pure functions + a browser-guarded app; tests `require()` the extracted script with no DOM.
- **Countdown math:** daysBetween("2026-07-01","2026-07-08")=7; same-day=0; 1-day-after=-1.
  `mandateStatus("2026-06-01")` → 37 days left, not in effect.
  `mandateStatus("2026-07-08")`, `("2026-10-19")`, `("2030-01-01")` → in effect, daysLeft clamped to 0 (never negative).
- **Checklist completeness:** empty/null → 0%; 4-of-8 items → 50%; 7-of-8 → 88%; all 8 → 100%.
  Composite items require ALL subfields (certifier name-only → incomplete; whitespace-only → incomplete).
  certType accepts only "CPC"/"GCC".
- **CSV escaping:** commas/quotes/newlines quoted per RFC 4180; null→empty; CRLF line endings.
- **Export gating:** 3-SKU fixture (1×100%, 1×50%+cert, 1×100% no cert) → eFiling CSV has 19-column header + exactly 1 row (the 100% SKU), with checklist values merged (certifier name, lab ID, custodian contact, cert type). Gap list flags the 50% SKU as `EFILING_CHECKLIST_INCOMPLETE_50PCT` and the cert-less SKU as `NO_CERTIFICATE_ON_FILE`; ready SKU absent. Empty state exports header-only.

## NOT verified (do not claim)
- **Actual browser rendering / interaction** — no live-browser access from this builder. JS is syntax-clean and logic-tested; visual layout, mobile behavior, localStorage persistence across reloads, and the checklist editor UI flow need a real-browser pass.
- **FormSubmit POST success** — the form posts to the provided token with the exact hidden fields, but delivery/activation was not tested (would require a real submission = an outward send, which was out of scope).
- **"eFiling-ready" legal accuracy** — the 19 CSV columns mirror CBP's PGA message-set field groups; a customs broker should confirm field mapping before any live filing.
- Seed data is intentionally EMPTY (real product, first-run onboarding). The v0 fictional sample dataset was dropped entirely.

## Notes for the next pass
- localStorage key changed: `certvault_v0_demo` → `certvault_beta_v1` (no migration; beta starts clean).
- "Reset demo data" replaced with "Erase all data" (confirm-gated).
- Generator watermark changed from "SAMPLE — NOT A LEGAL DOCUMENT" to "DRAFT" with a verify-before-use disclaimer; generator now also prompts completing the SKU's eFiling checklist after saving to the vault.
