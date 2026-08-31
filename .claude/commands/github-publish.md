---
description: Security-scan, capture a site screenshot into the README, push to GitHub, enable GitHub Pages via Actions, and fill in the repo About section
argument-hint: "[github repo url or owner/repo — optional, defaults to origin]"
allowed-tools: Bash(git status:*), Bash(git remote:*), Bash(git remote get-url:*), Bash(git remote set-url:*), Bash(git remote add:*), Bash(git ls-files:*), Bash(git add:*), Bash(git commit:*), Bash(git push:*), Bash(git log:*), Bash(git diff:*), Bash(git branch:*), Bash(git rev-parse:*), Bash(git cherry:*), Bash(gh auth status:*), Bash(gh repo view:*), Bash(gh repo edit:*), Bash(gh api:*), Bash(gh workflow run:*), Bash(gh workflow list:*), Bash(gh run list:*), Bash(gh run view:*), Bash(gh browse:*), Bash(mkdir:*), Bash(mv:*), Bash(find:*), Bash(ls:*), Bash(curl:*), Read, Write, Edit, Grep, Glob, mcp__playwright__browser_navigate, mcp__playwright__browser_resize, mcp__playwright__browser_wait_for, mcp__playwright__browser_take_screenshot, mcp__playwright__browser_close
---

You are publishing this repository to GitHub. Work through the steps **in order**.
The security scan in Step 2 is a hard gate: if it is not clean, stop and do nothing
that sends data off the machine.

Target repo argument: `$1` (a full `https://github.com/OWNER/REPO(.git)` URL or a
bare `OWNER/REPO`). If empty, use the current `origin` remote.

## Step 1 — Preconditions

- Run `gh auth status`. If not authenticated, stop and tell the user to run
  `gh auth login`.
- Run `git status` and `git branch --show-current`.
- Resolve the repo slug `OWNER/REPO`:
  - from `$1` if given (strip a leading `https://github.com/` and a trailing `.git`);
  - else from `git remote get-url origin`.
  - If neither exists, stop and ask the user for the repo.
- Compute the expected Pages URL: `https://OWNER.github.io/REPO/`.

## Step 2 — Security scan (HARD GATE — must pass before any push or API write)

Build the list of everything that would be published:
- `git ls-files` (tracked)
- `git status --porcelain` (staged + modified + untracked)
- `git ls-files --others --exclude-standard` (new files not ignored)
- unpushed commits: `git log --oneline @{u}..HEAD` (or all commits if no upstream)
  and their diffs.

Scan the **contents** of those files and the unpushed diffs for:

- **Secrets / keys:** private keys (`-----BEGIN * PRIVATE KEY-----`), `.pem` /
  `.key` / `.p12` files, AWS keys (`AKIA[0-9A-Z]{16}`), generic API tokens and
  bearer / OAuth client secrets, `Authorization:` headers with real values,
  connection strings containing a password, GitHub / Slack / Stripe / Google token
  prefixes.
- **Credentials:** passwords, OTPs, PINs, card numbers (13–19 digit PAN-like
  runs), CVV codes.
- **Confidential / PII:** `.env` or local-settings files, real personal emails and
  phone numbers, employee or customer records, internal-only hostnames / private
  IPs, database dumps or backups.
- **This project specifically** (it is a public training demo): the only email
  domain present should be the fictitious `@uobgroup.com`; phone numbers must stay
  as `1800-XXX-XXXX` placeholders; the footer disclaimer line ("not affiliated
  with or endorsed by United Overseas Bank Limited") must still be present; there
  must be no real credentials in the sample ticket data or FAQ text.

Also confirm `.gitignore` (create one if missing) excludes `.env*`, `*.key`,
`*.pem`, `.claude/settings.local.json`, `.playwright-mcp/`, and OS cruft.

**If anything is flagged:** STOP. Report every finding as `file:line` with a short
reason. Do not push, do not call any `gh api` write, do not "fix and continue"
silently — hand it back to the user.

**If clean:** print `Security scan passed — <N> files scanned, no secrets/PII/credentials found` and continue.

## Step 3 — Capture screenshot

Refresh `docs/screenshot.png` with a current capture of the site, using the
Playwright MCP browser.

- If the `mcp__playwright__browser_*` tools are unavailable (the `playwright` MCP
  server is not connected), print a one-line warning, keep any existing
  `docs/screenshot.png`, and continue to Step 4 — do **not** fail the publish.
- Otherwise:
  1. `browser_resize` to 1280 × 900.
  2. `browser_navigate` to the local file: `file://` + the absolute path to
     `index.html` in this repo (this captures the exact state being published and
     needs no network).
  3. `browser_wait_for` time 1 (let fonts and layout settle).
  4. `browser_take_screenshot` — `type: png`, `scale: css`, viewport only (not
     `fullPage`), `filename: "screenshot.png"`.
  5. `browser_close`.
  6. The file lands in the Playwright output dir or the repo root; `find` it, then
     `mkdir -p docs` and `mv` it to `docs/screenshot.png` (overwrite).
- Make sure `.gitignore` contains `.playwright-mcp/` so the MCP working directory
  is never staged.

## Step 4 — README

Read `README.md` if it exists, then create or update it. It should cover, concisely:
- what this is: a single self-contained `index.html` training/demo mockup of an
  internal IT Support Service Desk, themed "UOB Bank", with entirely fictitious
  names, numbers, email domains and SLAs;
- the not-affiliated-with-United-Overseas-Bank-Limited disclaimer;
- a **Live demo** link to the GitHub Pages URL from Step 1, immediately followed by
  the screenshot: `![<descriptive alt text>](docs/screenshot.png)`;
- how to run it: open `index.html` in a browser (works from `file://`), no build,
  no dependencies;
- the hard constraints from `CLAUDE.md` (one file, no network/persistence, never
  collects credentials).

Match the existing tone. Do not pad it with generic sections. If the screenshot
line is already present, leave it in place.

## Step 5 — Commit & push

- If `$1` was given and differs from the current `origin`: show the user the change
  and confirm before `git remote set-url origin <url>` (or `git remote add origin`
  if there is no remote).
- Stage the intended files only (`index.html`, `README.md`, `CLAUDE.md`,
  `docs/`, `.github/`, `.gitignore`, `.claude/commands/`, `.mcp.json`). Never stage
  scan-flagged files or `.playwright-mcp/`.
- `git commit` with a clear message describing what changed (skip if the tree is
  already clean).
- `git push -u origin <current-branch>`.
- Never `git push --force`. If the current branch is not `main`, ask before pushing.

## Step 6 — GitHub Pages via Actions

The workflow `.github/workflows/deploy.yml` already publishes Pages on every push
to `main`. Ensure Pages is enabled with the **GitHub Actions** source:

- `gh api -X POST repos/OWNER/REPO/pages -f build_type=workflow` — if it returns
  409 / "already exists", instead run
  `gh api -X PUT repos/OWNER/REPO/pages -f build_type=workflow`.
- Confirm the deploy run: `gh run list --workflow=deploy.yml --limit 3`. The push
  from Step 5 should have started one; if not, `gh workflow run deploy.yml`.
- Report the run URL and, once available, the real Pages URL from
  `gh api repos/OWNER/REPO/pages --jq .html_url`. Optionally verify the site and
  the screenshot asset return HTTP 200 with `curl`.

## Step 7 — About section

- `gh repo edit OWNER/REPO --description "Single-file IT Support Service Desk training/demo mockup (fictitious 'UOB Bank' theme)" --homepage "<Pages URL>"`
- Add topics: `gh repo edit OWNER/REPO --add-topic training-demo --add-topic html --add-topic github-pages --add-topic accessibility`
- Verify: `gh repo view OWNER/REPO --json description,homepageUrl,repositoryTopics`

## Step 8 — Report

Summarise for the user:
- security scan result and file count;
- whether the screenshot was refreshed (or skipped, and why);
- branch and commit SHA pushed, and the repo URL;
- Pages URL and the Actions run URL (note it may take a minute to go live);
- the About description, homepage, and topics that were set.
