# Trady Perch

Monorepo for the Trady Perch flagship marketing website and its shared design-system packages.

## Structure

```
apps/
  marketing-site/     Next.js (App Router) marketing site — the only app built so far
packages/
  tokens/              Design tokens (Design System Bible Ch.2-11, 52) -> CSS custom properties + TS
  motion/              Motion timing/easing tokens (Ch.15) + animation governance (Ch.40)
  ui/                  Shared React component library (framework-agnostic w.r.t. Next.js)
  config/              Shared tsconfig, eslint, prettier, performance budget
docs/
  adr/                 Architecture Decision Records
  _synthesis/          Extracted, actionable summaries of the ~150-document constitution
  design-system-bible/, product-implementation-constitution/, ...   Source-of-truth documents
```

## Why this structure

Every architectural choice here traces to a specific chapter of the constitutional documents in `docs/`, per the Translation Doctrine (Product Implementation Constitution Ch.3). Where the constitution left a decision open (it deliberately names no frontend framework, for example), the decision and its reasoning are recorded in `docs/adr/`. Start there — and in `docs/_synthesis/` — before assuming a value or convention; nothing in this codebase should be invented from habit or a framework's defaults when a governing document exists.

## Getting started

```bash
npm install
npm run dev     # builds tokens, then starts the marketing site at localhost:3000
```

```bash
npm run build       # production build (tokens + Next.js)
npm run lint         # ESLint across all workspaces
npm run typecheck    # TypeScript across all workspaces
npm run test         # Vitest across all workspaces
```

## Status

Milestones 1–10 complete: setup; global layout/navigation/footer; homepage; every remaining page; motion & interactivity; responsive implementation; accessibility; performance; SEO (structured data, `llms.txt`, sitemap/robots); final QA.

Two things this section previously overstated, corrected here rather than quietly edited:

- **Accessibility conformance is WCAG 2.1 AA, not 2.2 AA.** Design System Bible Ch.53 maps the system to WCAG 2.1 specifically (it predates 2.2), and `npm run test:a11y` gates on the `wcag2a`/`wcag2aa`/`wcag21a`/`wcag21aa` tag set to match. The Milestone 10 review additionally checked the 2.2-only criteria by hand and fixed what it found (2.4.11 Focus Not Obscured; 2.5.8 Target Size). 2.2 is not yet a *claimed, mapped* conformance target — see the Chapter 66 register.
- **The performance budget exception is sitewide, not "one".** Every route exceeds Ch.36's 2000ms LCP budget under Lighthouse's throttled lab methodology, so `npm run test:performance` exits non-zero by design. Unthrottled real-browser LCP is 204–272ms. The full record, including root cause, is [[ADR-0008]] plus its Chapter 66 entry — read those before re-diagnosing.

**CI runs on every push and pull request to `main`** via `.github/workflows/ci.yml` — static checks (lint, typecheck, production-dependency vulnerability scan), then unit/integration tests and the full-route axe-core accessibility audit in parallel, in Ch.49 §2's gate order. `test:performance` is deliberately excluded because it fails by design (see the LCP note below). **These checks report but do not yet block**: making them enforcing requires a branch-protection rule on `main`, which is a repository-settings action — see the Chapter 66 register entry for the exact steps.

**Contact form delivery is live as of 2026-07-26, and delivers to `hello@tradyperch.com` as of 2026-08-01.** Validated submissions are delivered via Resend to the address in `MARKETING_SITE_CONTACT_INBOX_EMAIL`. Copy `apps/marketing-site/.env.example` to `.env.local` and fill it in; with no values set the form still validates and reports success, logging server-side that nothing was delivered, so the app builds and tests without a secret.

`MARKETING_SITE_CONTACT_INBOX_EMAIL` and `MARKETING_SITE_RESEND_FROM_EMAIL` are a matched pair — Resend rejects any `from` on an unverified domain, and its shared `onboarding@resend.dev` fallback can only deliver to the Resend account owner's own address. `tradyperch.com` was verified on 2026-07-31, which is what allows the inbox to be `hello@` rather than that owner address; the sender is now `hello@tradyperch.com` on the verified domain. **Clearing the `from` variable would silently drop delivery back to the shared test sender, which cannot reach `hello@` — every submission would fail with a 403 that the route surfaces as a 502, losing the message.** Sending and receiving are independent: Resend verification only grants the right to send, while `hello@` receives because the domain's MX records point at Hostinger mail.

Note that a truthful `{"ok":true}` from `/api/contact` does **not** by itself prove delivery — the handler returns exactly that, by design, when the API key or inbox address is unset. Confirm real delivery against the Resend dashboard or `GET https://api.resend.com/emails`, not the endpoint's response.

**Manual accessibility testing (Ch.19 Layer 3) — partially complete as of 2026-07-26.** A human pass with NVDA on Windows covered the screen-reader walkthrough and keyboard-only task completion and found no defects; that entry is resolved in the Chapter 66 register. Touch/gesture verification on real touch hardware — Ch.19 §2's third component, which a desktop screen reader cannot exercise — remains open there as its own entry. Per Ch.19 §3, the passing result binds to this release cycle only and the flows re-enter the cadence next cycle.

See `docs/adr/` for architectural decisions and Chapter 66 for every currently-open, disclosed gap.

Additional verification scripts beyond `npm run test`, each self-contained (boots its own server):

```bash
npm run test:a11y --workspace=@trady-perch/marketing-site         # real-Chromium axe-core, every route
npm run test:keyboard --workspace=@trady-perch/marketing-site     # keyboard-only navigation
npm run test:performance --workspace=@trady-perch/marketing-site  # Lighthouse + CDP byte measurement
npm run test:schema --workspace=@trady-perch/marketing-site       # structured-data (JSON-LD) validation
```
