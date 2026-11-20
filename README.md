# struXctrl (formerly SiteSync Pro)

**A site-management platform for pre-engineered building (PEB) and structural steel erection.**
Built solo. Live on site and in daily operational use. Product site: **https://struxctrl.in**

> This repository is a public showcase. **It contains no source code** — struXctrl is a
> commercial product and the codebase is private. What follows is an architecture and
> capability write-up for people evaluating the work.

---

## What it replaces

Most PEB and structural steel erection projects are coordinated with spreadsheets and
WhatsApp. Progress lives in one place, material in another, crane hours in a third, and
nobody can reconcile them until month end — by which point the cost overrun already happened.

struXctrl is the single system of record for one question:

> *What was erected today, by whom, with which crane, against what plan, using what material,
> at what cost per metric ton?*

The bar it was built to: **a project manager standing in a yard should be able to answer that
in under ten seconds, from a phone, without deciding which screen to open.**

---

## Scale

| | |
|---|---|
| Screens | 69 |
| Source files | 489 |
| Test files | 198 |
| Commits | 982 |
| Time from first line to production | **under 3 months** |
| Time live on site in daily operational use | **5 months** |
| Contributors | 1 |

---

## Who uses it

Five role types, each with a genuinely different context — not one interface behind five
permission flags.

**Site Engineer** — mobile-first, in the field. Phone, outdoors, direct sun glare, sometimes
thin gloves, hands dusty or oily, patchy network and sometimes fully offline for hours. Logs
progress, materials, machinery hours, hindrances, HSE checks, NCRs. Frequently interrupted and
needs to resume mid-task. Large tap targets and offline queueing are not nice-to-haves here.

**Project Manager / Senior Site Engineer** — same field context, elevated capability. Approves
progress submissions, assigns engineers, builds the daily plan, works an approval queue.

**Admin** — desktop, command centre. Project schedule, drawings, procurement, plate
requirements, machinery rates, BOM generation, RA bills, NCR audits. Information-dense screens
are correct here. Long sessions, multi-tab workflows.

**Executive** — read-only, KPI-focused. Cost per tonne erected, schedule variance, hindrance
trends. Skim-reads on a phone between meetings, deep-dives at a desk.

**Production Floor** — factory side, physically separate from the erection site. Shipper
lists, dispatch, QC review.

---

## What makes it deep

The parts that separate this from a generic project tracker:

**Offline-first, idempotent sync.** The field app queues writes to IndexedDB when offline and
replays on reconnect. Replays are idempotent — a queued manpower append re-merges cleanly and
"already done" is treated as success — so a flaky site connection never duplicates or corrupts
data.

**True cost per tonne.** Crane and equipment hours, fuel (with litres-per-hour anomaly
flagging), and labour are attributed to the steel actually erected. That produces a live
rupees-per-tonne figure with a daily cost breakdown, rather than a month-end estimate.

**Material waste optimisation.** A plate-nesting optimiser selects mill sheets from a real
catalogue and traps scrap. A section optimiser chooses 6 m against 12 m mill stock for
RHS/SHS/CHS/ISMB and similar. Both run automatically when procurement is approved and produce
tracked, receipt-managed line items. Off-cuts above a threshold are logged as reusable
remnants.

**Planned vs actual in pieces *and* tonnage.** End-of-day comparison of the published plan
against PM-approved erection at part-mark granularity, with off-plan work split out and
completion percentage shown. A daily steering tool, not a monthly autopsy.

**OCR in the field.** Shipper lists and hour-meter readings are photographed and read by OCR,
turning a paper list into structured data with no manual re-entry step.

**One shared crew per day.** Manpower is modelled as a single shared record per project per
day that engineers append to — matching how a real crew is logged once and shared, rather than
duplicated per person. Modelling this wrong is the most common way site software produces
numbers nobody trusts.

**Optimistic write UX throughout.** Every write commits instantly in the UI and reconciles in
the background, with per-row rather than whole-page disabling. Applied across dozens of write
surfaces so the app stays usable on a slow connection.

---

## Architecture

**Frontend** — React 19 + Vite. Mobile-first PWA, Tailwind, list virtualisation for long
tables, charts, on-device PDF generation for reports.

**Backend** — Supabase: Postgres, authentication, file storage, server-side RPC.

**Edge** — server endpoints (auth, OCR, optimiser review, weather) run as Cloudflare Pages
Functions. The dev server reuses the same handlers, so there is a single source of truth for
server logic rather than a drifting local mock.

**Offline** — service worker plus an IndexedDB queue make the app installable and
field-resilient.

**Auxiliary** — Gemini for OCR, OpenWeather for weather-driven hindrance logging, Firebase for
push notifications, Sentry for production client-side error capture.

**Structure** — many small focused modules over few large ones. A service layer of one file per
domain mediates all database access; pure independently testable helpers handle the derivations
(planned-vs-actual, costing, erection rollups). That is what makes 198 test files tractable for
one person.

---

## Engineering decisions worth defending

**Structured input over AI extraction, where accuracy is non-negotiable.** Drawing extraction
by model was evaluated and deliberately not used for billing- and fabrication-critical records.
A plausible-looking wrong part mark is more expensive than a slower correct one. OCR is used
where a human verifies the result immediately (shipper lists, hour meters); it is not used
where an error would propagate silently into a bill.

**The LLM advises, it does not decide.** The steel plate optimiser is deterministic: it scores
candidate layouts on waste and scrap-trap. A model is then given the top candidates and asked
to confirm or re-rank based on project context that geometry alone cannot capture. If the model
is unavailable, the deterministic result stands. Knowing where a model does *not* belong
matters more than knowing how to call one.

**Provenance over polish.** Every number is shown with how it was derived. Tonnage attributed
to a specific crane. Fuel cost computed from a stated formula. An audit log with a hash chain.
The product earns trust by showing its work.

**Builder language, not SaaS language.** "Log Completed", not "Submit progress milestone".
"Crane Used", not "Equipment allocation". The interface uses vocabulary that was already in the
site office. Adoption is a language problem before it is a feature problem.

---

## Built with

Developed end to end using **Claude Code** and **Google Antigravity** as the primary
development environments — agentic, not autocomplete. That is how one person took 69 screens
with a 198-file test suite to production in under three months.

---

## About

Built by **Vishnu Vardhan Maganti** — 14+ years delivering heavy civil, infrastructure and
structural steel projects, most recently as General Manager, now building the software that
runs them.

The reason struXctrl got adopted is not the stack. It is that requirements were gathered
standing on site, and the workflows were rebuilt around how crews already worked rather than
how the software wanted them to.

📧 magantivishnu1@gmail.com · 💼 [LinkedIn](https://www.linkedin.com/in/vishnu-maganti-440b04119/)

---

<sub>No source code, screenshots, client data, or personal information is published in this
repository. Product screenshots are available on request in a redacted form.</sub>
