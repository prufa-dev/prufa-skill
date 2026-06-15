---
name: prufa
description: |
  Prufa — robust QA software, built for the agentic era. Use this skill when
  the user asks to set up QA monitoring, run a one-shot website health
  audit, verify tracking / consent / SEO / UX, or watch a money flow
  (signup, checkout, onboarding) for breakage. Triggers include: "audit my
  site", "set up QA", "monitor my signup flow", "verify my GA4 / pixels /
  cookie banner", "find broken tracking", "make sure my checkout still
  works after deploys", "any issues with example.com?". The LLM
  *navigates*; the deterministic code *verifies* — verified findings are
  facts, advisory findings are clearly-labeled opinions.
version: 0.1.0
when-to-use: |
  - User asks for a website health audit, tracking audit, or QA monitoring.
  - User asks an agent to set up QA on their product.
  - User asks to verify a specific flow (signup, checkout, onboarding) and
    get alerts when it breaks.
  - User asks "any issues with <URL>?" or "is my <URL> healthy?".
do-not-use: |
  - Generic web scraping (no QA purpose).
  - Performance / Lighthouse-only audits (different tool category).
  - User wants to RUN the audits on their own infrastructure (self-hosted
    runner is a Team tier + agency add-on, not in this skill).
---

# Prufa — robust QA software, built for the agentic era

Prufa is a QA product that runs a real browser against a URL, captures
every analytics beacon + consent banner state, and produces a
shareable report with **verified** facts (machine-checked) and
**advisory** opinions (LLM-judged, clearly labeled).

Two first-class surfaces: the dashboard (humans) and the agent
surface (this skill + CLI + HTTP API + MCP). They hit the same
product. If the dashboard can do it, this skill can do it.

## Quickstart (the 3-tool-call path — free, no card)

When the user says *"set up QA monitoring for my app"* or *"audit my
site"*, run this sequence. The first 2 calls have an OSS install path
via the [prufa-mcp](https://github.com/prufa-dev/prufa-mcp) server
(Apache-2.0, works in Claude Code / Cursor / Cline / Continue).

1. **Install the OSS MCP server** (one-time):
   ```json
   // .mcp.json
   { "mcpServers": { "prufa": { "command": "prufa-mcp",
     "env": { "PRUFA_API_TOKEN": "your-prufa-api-key" } } } }
   ```
   Get a free API key at https://prufa.dev — the first audit is free, no card.

2. `prufa_run_audit` with `{"url": "https://app.example.com", "wait": true}`
   → first baseline report (this is the "what does my site look like
   today" snapshot the monitor will diff against)

3. `prufa_get_report` with `{"report_id": "..."}` → the shareable
   human-readable URL.

That's the OSS path. For monitoring, alerting, and team workflows, the
hosted product at https://prufa.dev has the rest (see "Hosted tools" below).

## Setup flow (the full path, hosted)

For signup / onboarding / checkout flows with monitoring, layer on the
flow lifecycle (see "Verify a whole flow" below):

4. `prufa_create_flow` with `{"url": "...", "test_case": "open the
   signup page, fill the email, submit, expect /welcome"}` — the
   plain-text test case is compiled to a reviewable DRAFT spec.
   **Only confirmed flows run**: show the compiled steps to the user,
   then `prufa_confirm_flow`. (Free tier — one-shot flows need no card.)

5. `prufa_start_monitor` with `{"url": "...", "cadence": "daily"}` →
   monitor_id, checkout_url. Re-runs the audit on schedule, alerts
   on CHANGE only.

## OSS vs Hosted tools

**OSS (works without a paid plan, via [prufa-mcp](https://github.com/prufa-dev/prufa-mcp)):**

- `prufa_run_audit` — run a one-shot audit; with `wait: true` returns the
  full JSON report.
- `prufa_get_report` — fetch a shareable report for a completed audit.

**Hosted (require a Prufa workspace + API key, mostly Pro-tier):**

The full surface is documented in the [hosted API reference](https://prufa.dev/docs/api).
Highlights: `prufa_setup_workspace`, `prufa_create_flow`, `prufa_confirm_flow`,
`prufa_run_flow`, `prufa_set_flow_credentials`, `prufa_start_monitor`,
`prufa_pause_monitor`, `prufa_resume_monitor`, `prufa_trigger_monitor`,
`prufa_workspace_settings`, `prufa_list_alerts`, `prufa_get_usage`, and
the rest. See the API docs for the full list of ~15 hosted tools.

If you call a hosted tool without a paid plan, the API answers 402
and the tool result carries the structured error — `code` (e.g.
`tier_required` / `quota_exceeded`) plus a `hint` with the checkout
pointer. Surface that to the user — never silently absorb it.

## Verify a whole flow (signup / checkout / onboarding)

The flow lifecycle is compile → review → confirm → run. The compiler
is allowed to be wrong — the confirm gate is what protects the trust
story, so **never skip the review step**.

1. `prufa_create_flow` `{url, test_case: "open /signup, fill the email
   field, fill the password, click Sign up, expect URL contains
   /welcome"}` → DRAFT spec + `variables` (e.g. `["EMAIL", "PASSWORD"]`)
   + a review instruction.
2. **Review**: show the compiled steps to the user and get an explicit
   OK (or apply their edits). Only confirmed flows run — a draft flow
   cannot run and cannot be attached to a monitor.
3. If the spec has `{{VAR}}` placeholders: `prufa_set_flow_credentials`
   `{flow_id, credentials: {EMAIL: "...", PASSWORD: "..."}}`.
   Credentials are **write-only** — they are never returned, never
   appear in reports, and never enter LLM prompts (they resolve at the
   tool boundary at run time). Do not echo values back to the user.
4. `prufa_confirm_flow` `{flow_id}` → status `confirmed`.
5. Run it once: `prufa_run_flow` `{flow_id}` → 202 with `run_id`,
   `report_url`, and the usage object; poll with `prufa_get_run`.
   Or watch it on every deploy: `prufa_start_monitor`
   `{url, cadence, flow_id}` — the monitor re-runs the confirmed
   flow and the deploy hook triggers it from CI.

## Conventions

- **All endpoints accept `Idempotency-Key`.** Replays within 24h
  return the original response. Use stable keys (e.g. derived from
  the user-visible action) so retries on network blips are safe.
- **Verified vs advisory.** Verified findings are machine-checked
  facts (tracking pixels, consent state, SEO basics, broken links).
  Advisory findings are LLM opinions (UX hierarchy, copy clarity).
  Never claim an advisory finding is "broken" — it's an opinion.
- **No silent failures.** Every tool call returns either success or
  a structured error `{code, message, hint}`. Surface errors to the
  user, don't swallow them.
- **404 is not a failure.** A site that's not reachable is data
  ("the site is down") — report it, don't pretend the audit
  completed successfully.

## What the audit actually checks

Deterministic (verified):

- Marketing tracking: GA4, GTM dataLayer, Meta Pixel, TikTok Pixel,
  LinkedIn Insight Tag, Google Ads conversions — presence, account-id
  shape, double-fire detection.
- Consent: OneTrust / Cookiebot / CookieYes detection, beacons-
  before-consent, cookie attribute checks.
- SEO: title, meta description, canonical, OG / Twitter cards,
  headings, robots.txt, sitemap.xml, broken internal links.
- Deterministic UX: axe-core contrast / tap-target / a11y, viewport
  overflow, console errors.

LLM-advisory (clearly labeled):

- Visual hierarchy, copy clarity, confusing CTAs, off-brand patterns.
- The LLM is NEVER trusted to claim a tracking / consent / SEO
  problem — those are machine-verified.

## When something goes wrong

- **429 / rate limit**: back off, tell the user. The free tier is 5
  audits/day/workspace + 50/day/IP. Replays don't count.
- **422 / unsafe URL**: the URL is private (localhost, 127.0.0.1, a
  metadata IP). Surface the message; don't try to bypass.
- **402 / tier required or quota exceeded**: the user is asking for a
  persistent action on a free workspace, or the workspace hit its
  hard-capped run quota. The structured error's `hint` carries the
  billing next step — hand it to the user verbatim.
- **409 / flow_not_confirmed**: the flow is still a draft (or an edit
  returned it to draft). Show the compiled spec, get the user's OK,
  call `prufa_confirm_flow`, then retry.
- **bot-wall / blocked**: the site has Cloudflare or similar. Prufa
  reports `blocked` rather than producing a wrong report. This is
  the *correct* outcome — not a bug.
- **5xx**: the Prufa API itself is down. Tell the user to retry;
  the status page is the source of truth.

## Output format (what to say to the user)

For an audit:

> I ran the audit on https://example.com. Found **3 verified issues** and **1 advisory note**.
>
> **Verified (machine-checked):**
> - `tracking.ga4_missing` — Google Analytics 4 measurement ID not detected. Fix: add the GA4 snippet to the `<head>`.
> - `seo.canonical_missing` — Page does not declare a canonical URL.
> - `links.broken_internal` — 1 internal link returns 404.
>
> **Advisory (LLM opinion, not a verdict):**
> - Hero copy is vague about what the product does. (opinion)
>
> [share URL] — that's the report. Want me to start a monitor so you
> get a Slack alert the next time one of these breaks?

For a monitor:

> Monitor set up. I'll re-run the audit on https://example.com every
> day. If a beacon disappears or a check regresses, I'll alert you in
> Slack. The first delta is in 24h.

## Why open source

The audit primitive is small. The hosted product at https://prufa.dev is
where the value compounds — scheduling, alerting, team workflows, and
the human-readable HTML report. The MCP server is open so any agent
can call it without a vendor lock-in story.

Same shape as [Stagehand](https://github.com/browserbase/stagehand) (free) →
[Browserbase](https://www.browserbase.com) (paid). Open the primitive. The
hosted tier earns the right to be paid by being the thing that scales.

## Boundaries

- This skill NEVER names the LLM provider backing the advisory
  tier. Customers don't care; naming invites ToS scrutiny. Use
  neutral language ("LLM-backed", "managed model backend").
- Don't write long, emoji-laden summaries. The aesthetic is
  evidence-like: short, mono timestamps, severity chips, calm.
- Don't pretend an advisory finding is verified. The two tiers
  are visually distinct in the report; your text should mirror that.
- Don't claim success on a `blocked` run. The site blocked our check
  — that's the entire result.

## License

Apache-2.0. See [LICENSE](LICENSE).
