# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-page training/demo mockup of an internal IT Support Service Desk, themed
as "UOB Bank". It is a teaching artifact, not a real product — all names, phone
numbers, email domains (`@uobgroup.com`), and SLAs are fictitious. The footer
carries a disclaimer that it is not affiliated with United Overseas Bank Limited;
keep that line intact.

## Running

There is no build step, package manager, test runner, or lint config. The entire
site is `index.html`. Open it directly in a browser (double-click, or
`Start-Process index.html` on Windows / `open index.html` on macOS). It must keep
working from a `file://` URL.

## Hard constraints (do not break these)

- **One self-contained file.** All CSS stays in the single `<style>` block in
  `<head>`; all JS stays in the single `<script>` block before `</body>`. No
  frameworks, no external CSS/JS/font/image URLs, no build tooling.
- **No persistence, no network.** Submitted tickets live only in the in-memory
  `submittedTickets` array inside the IIFE in `index.html`. Do not add
  `localStorage`, `sessionStorage`, `fetch`, or any request.
- **Never collect credentials.** No password / OTP / PIN / card-number fields. The
  description textarea runs a regex (`CRED_RE`) that shows a non-blocking warning
  if the user types something credential-like; keep that behaviour.

## Architecture

`index.html` is organised into commented sections that mirror the UI:

- **Section 1 — Header:** sticky `<nav>`, hamburger toggle (`#navToggle` toggles
  `.open` on `#navList` and flips `aria-expanded`), hero banner, three quick-info
  cards. Anchor links (`#home`, `#ticket`, `#faq`, `#contact`) drive smooth-scroll
  navigation; `scroll-padding-top` offsets the sticky bar.
- **Section 2 — Ticket form (`#ticketForm`):** the core logic. A `validators`
  object maps each field id to a function returning `""` (valid) or an error
  string. `FIELD_ORDER` fixes both validation order and first-invalid focus.
  `validateField()` / `setError()` drive inline `.error` spans and `aria-invalid`;
  they run on blur/change and again on submit. On a clean submit: `preventDefault`,
  build a reference `UOB-ITSD-YYYYMMDD-####`, push a ticket object onto
  `submittedTickets`, then swap the form for `#successPanel` (which shows ref,
  priority, and SLA from the `SLA` map). "Submit another ticket" resets the form
  and clears counters/errors/filename/warning.
- **Section 3 — FAQ:** eight accordion items. Clicking a `.faq-q` button closes all
  others then toggles its own `aria-expanded` + panel `hidden` (one open at a
  time). `#faqSearch` filters `.faq-item`s by `textContent` match and toggles
  `#faqEmpty`.
- **Footer:** three columns plus the disclaimer and copyright lines.

All JS is one IIFE in `index.html`; there are no modules or separate files.

## Conventions

- Vanilla ES5-ish JS (`var`, function expressions, `Array.prototype.slice.call`) —
  match the existing style rather than introducing modern syntax piecemeal.
- Design tokens are CSS custom properties on `:root` (`--blue #00539B`,
  `--accent #E8A33D`, `--text #2C3038`, etc.); use them instead of hard-coded
  colours. Layout is a 1120px `.container`; responsive rules collapse to one column
  in the `@media (max-width: 768px)` block.
- Accessibility is a requirement, not a nice-to-have: semantic landmarks, visible
  `:focus-visible` outlines, `aria-invalid` on bad fields, `role="alert"` /
  `role="status"` on live regions, labelled controls. Preserve these when editing.
