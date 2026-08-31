# UOB · IT Service Desk (training demo)

A single-page training/demo mockup of an internal IT Support Service Desk, themed
as "UOB Bank". It is a teaching artifact, not a real product — all names, phone
numbers, email domains (`@uobgroup.com`), SLAs, and FAQ content are fictitious.

**Training demo — not affiliated with or endorsed by United Overseas Bank Limited.**

## Live demo

https://twm1909.github.io/itsupport/

![Screenshot of the IT Service Desk demo — hero banner and quick-info cards](docs/screenshot.png)

## Running locally

The entire site is `index.html`. Open it directly in a browser — double-click it,
or `Start-Process index.html` (Windows) / `open index.html` (macOS). There is no
build step, package manager, or dependency, and it works from a `file://` URL.

## What's inside

- **Header** — sticky nav with mobile hamburger, hero banner, three quick-info cards.
- **Ticket form** — inline validation on blur and submit (Staff ID `######`,
  corporate email must end `@uobgroup.com`, priority, live character counters,
  attachment filename display, confirmation checkbox). A clean submit generates a
  mock reference `UOB-ITSD-YYYYMMDD-####` and shows the matching SLA.
- **FAQ** — eight one-at-a-time accordion items with live text search.
- **Footer** — quick links, contact details, and the disclaimer.

## Constraints (kept deliberately)

- **One self-contained file.** All CSS is in the single `<style>` block; all JS is
  in the single `<script>` block. No frameworks, no external URLs, no build tooling.
- **No persistence, no network.** Submitted tickets live only in the in-memory
  `submittedTickets` array — no `localStorage`, `sessionStorage`, or requests.
- **Never collects credentials.** No password / OTP / PIN / card-number fields; the
  description field shows a non-blocking warning if the user types something
  credential-like.

## Deployment

`.github/workflows/deploy.yml` publishes the repository root to GitHub Pages on
every push to `main` (source: GitHub Actions).
