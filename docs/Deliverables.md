# Carbon Atlas — Deliverables

**The single source of truth.** Roadmap, task list, AI prompts, file changes, conventions and
session history. Everything else in this folder is linked from here.

**Window:** 19 August → 28 September 2026 · **Team:** Dev, Dev, Dev · **Cadence:** weekly delivery

---

## Index

| Document | What it answers |
|---|---|
| **Deliverables.md** *(this file)* | What we build, by when, who has it, and the prompt to build it with |
| [IDENTITY-AND-SCOPE.md](./IDENTITY-AND-SCOPE.md) | A user in many tenants, companies and branches — and how "web but not mobile" is enforced |
| [ARCHITECTURE.md](./ARCHITECTURE.md) | The services, the data model, isolation, and what already exists |
| [INTEGRATION_STANDARD.md](./INTEGRATION_STANDARD.md) | **Binding.** How a screen talks to a microservice |
| [CONFIGURABILITY.md](./CONFIGURABILITY.md) | **Binding.** Whether a thing becomes a column, a configurable field, a rule, a catalogue row or code — and what each costs to support |
| [RULES-AND-WORKFLOWS.md](./RULES-AND-WORKFLOWS.md) | The cross-industry rule and workflow taxonomy, for all seven verticals |
| [EU-BIOMETHANE.md](./EU-BIOMETHANE.md) | The biomethane domain in depth: RED III, UDB, ETS, CSRD — with the calculations |
| [OFFSET.md](./OFFSET.md) | The offset planning engine, and why it differs per branch and location |
| [COST.md](./COST.md) | Unit economics, subscription design, and how a price is computed |
| [COMPETITORS.md](./COMPETITORS.md) | Twenty-one competitors, where they are strong, and the gaps worth taking |

> **Where this folder lives.** In `Carbon-atlas-webapp/` because the repository root is not version
> controlled, and a source of truth that is not versioned is not a source of truth. It covers all
> three repositories despite living in one. Moving it to a dedicated docs repository is one command —
> every internal link is relative.

---

## How to use this file

**Starting a session**

1. Read [Session log](#session-log) — the last entry says where work stopped.
2. Read [Ground truth](#ground-truth-what-already-exists). Most of the backend is built; the single
   most expensive mistake available here is rebuilding something that already works.
3. Find your task in the [Roadmap](#roadmap) by ID (`F1`, `A12`, `C8`…).
4. Copy the matching prompt from [AI prompts](#ai-prompts) into a fresh agent session.

**Finishing a task**

1. Tick the row in the roadmap (`☐` → `☑`).
2. Update or create the `GUIDE.md` in the feature folder you touched.
3. Add a [Session log](#session-log) entry: what landed, what did not, what the next person needs.

**Developers may edit this file directly.** Add rows, change estimates, leave notes in Comments. It
is the plan of record, not a generated artefact.

**Business team.** Read [Scope reality check](#scope-reality-check), the weekly deliveries, and then
[COST.md](./COST.md) and [EU-BIOMETHANE.md](./EU-BIOMETHANE.md).

---

## Scope reality check

29 working days × 3 developers × ~6 productive hours ≈ **522 hours**, minus the freeze day ≈ **504**.

Roughly double the original window, and it changes the answer: **most of the requirement now fits**,
including rules, workflow and a biomethane vertical slice for Ace. What still does not fit is in
[Deferred backlog](#deferred-backlog) with an estimate against each item, so deferring stays a
decision rather than a discovery.

Two structural risks, both cheaper to accept than to discover:

| | |
|---|---|
| **`UNIT` scope does not exist yet** | A sixth level below INDUSTRY, touching every resolver. Done in one change (`T1`) it is a day; discovered piecemeal it is a second resolution order that disagrees with the first, surfacing months later as "why is this field on the form". |
| **Payment gateway and invoice generation are not built** | The entities exist; the integration does not. Both are on the W4 critical path, and a slipped gateway means invoice screens ship read-only. |

---

## Ground truth: what already exists

**Do not rebuild these.** The backend for most of the requirement is done and tested. The bulk of
the remaining work is **web UI over existing APIs**.

| Capability | State | Where |
|---|---|---|
| Auth: register, login, forgot/reset/change, OTP, refresh, `/me` | ✅ built | `al.auth/Api/AuthEndpoints.cs` |
| Auth: challenge/nonce pre-login, single re-challenge | ✅ built | `al.auth/Services/LoginChallengeService.cs` |
| Four Ace users — RBAC assignments **and** login identities | ✅ built | `al.master/Data/AceTenantSeeder.cs`, `al.auth/Data/AceUserSeeder.cs` |
| Tenant list & context switch (`/auth/tenants`, `/auth/context/switch`) | ✅ built | `al.auth`, `useSwitchContext.hook.ts` |
| RBAC: roles, permissions, route rules, scope assignment | ✅ built | `al.master`, `al.net.authorization` |
| Membership grant model (application dimension, validity, grant type) | ✅ built | `al.net.entities/Identity/Grant.cs`, `al.master/Api/GrantEndpoints.cs` |
| Tenant / company / branch / industry catalogues | ✅ built | `al.net.entities/Master` |
| **UNIT** scope level | ❌ missing everywhere | task `T1` — blocks `T4`, `C4`, `D3` |
| Subscription plans, trial, provisioning snapshots | ✅ built | `al.subscription` |
| Quota: definition, scope resolution, enforcement | ✅ built | `al.net.data/Authorization/EfQuotaGuard.cs` |
| Wallet: scoped, nearest-first burn | ✅ built | `al.net.entities/Billing/Wallet.cs` |
| Promo codes, discounts, time windows | ✅ built | `al.net.entities/Billing/PromoCode.cs` |
| Invoice / PaymentProvider / PaymentTransaction **entities** | ✅ built | `al.net.entities/Billing` |
| Payment **gateway integration** | ❌ missing | task `C8` |
| Invoice **generation** | ❌ missing | task `C10` |
| Provider bindings — DB-driven, per scope, per channel | ✅ built | `al.net.data/Registry/EfProviderBindingResolver.cs` |
| UDF **metadata engine** — resolution, scope, i18n, tooltips, sections, conditions, static-field catalogue, validator | ✅ built | `al.net.udf`, `al.udf` |
| UDF **data plane** — `custom_fields` jsonb + GIN on company and branch, fail-closed save interceptor, validation over HTTP for services with no document store | ✅ built | `al.master`, `al.udf` |
| UDF **definition versioning** — every edit archives the outgoing revision; values keep the revision they were captured under and are still validated by it | ✅ built | `al.net.udf` |
| UDF **consumed by a screen** — no React screen renders a resolved form; `CompanyOnboardingPage` still hand-codes from `@/mocks/mockForms` | ❌ missing | task `T3` |
| UDF **form builder wired to UDF** — `/form-builder` is 25 files of localStorage with zero UDF references | ❌ missing | task `D1a` |
| UDF for **list/table views** (columns per scope) | ❌ missing | task `D2` |
| Rules engine | ✅ built | `al.net.rules`, `al.rules` |
| Workflow engine | ✅ built | `al.net.workflow`, `al.workflow` |
| Rules/workflow **authoring UI** | ❌ missing | `R1`–`R4`, `W1`–`W4` |
| Emission calculation engine | ◐ partial | `E1`–`E4` |
| Offset planning engine | ◐ partial | `O1`–`O3` |
| Bulk import/export + review grid | ✅ built | `al.ingestion`, `packages/react/src/forms/bulk` |
| Web: auth screens, DataGrid, DynamicForm, api-client, UDF adapters | ✅ built | `apps/web`, `packages/react` |
| Web: design tokens, app shell, error boundary, diagnostics | ❌ missing | `F1`, `F2`, `F3` |

---

## Backend gaps that block UI

Schedule these first. A UI task blocked on a missing API stalls a developer for a day.

| ID | Gap | Blocks | Week |
|---|---|---|---|
| `A6` | Membership grant model | `A12`, `A14`, `A15` | W1 |
| `A7` | Application gating at token issuance | mobile access control everywhere | W1 |
| `T1` | `UNIT` scope across entities, resolvers, migrations | `T4`, `C4`, `D3`, `R2` | W2 |
| `C3` | Pricing engine | `C5`, `C10` | W4 |
| `C8` | Payment gateway | `C11`, `C13` | W4 |
| `C10` | Invoice generation | `C13` | W4 |
| `D2` | UDF for list/table views | the config UI half of `D2` | W3 |
| `R0` | EU requirement extraction | `R3`, `B2`, `B6` | W5 |

---

## Scope model

```
PLATFORM  →  TENANT  →  COMPANY  →  BRANCH  →  INDUSTRY  →  UNIT
                                    ×
                       APPLICATION (web | mobile | api)
```

`APPLICATION` is **orthogonal**, not a level: the same branch shows five fields on web and three on
mobile. It breaks ties at the same organisational level; it does not sit inside the hierarchy.

**Specificity weights** — existing in `UdfScopes.Specificity`, extended by `T1`:

```
BRANCH 8 · COMPANY 4 · INDUSTRY 2 · APPLICATION 1                   ← today
UNIT 16 · BRANCH 8 · COMPANY 4 · INDUSTRY 2 · APPLICATION 1         ← after T1
```

One resolution order across the whole platform, deliberately. A second would eventually disagree
with the first.

**Permissions union; configuration resolves most-specific-wins; denials beat both.** The rule most
likely to be implemented backwards — see [IDENTITY-AND-SCOPE.md](./IDENTITY-AND-SCOPE.md).

**Downstream may not exceed upstream.** A tenant with a 1 TB allocation may divide it as it likes,
but the sum below can never exceed the ceiling above. That is `C4`, and it is the most-tested rule
in the commerce track.

---

## Conventions

### Documentation

Every feature folder carries a `GUIDE.md`, matching the `README.md` convention the SDK packages and
services already follow.

```
apps/web/src/features/<feature>/GUIDE.md
packages/react/src/<area>/README.md
```

A `GUIDE.md` answers, in order: **what this is for**, **the decisions that shape it and what the
alternatives cost**, **how to use it**, **the pitfalls**. Not an API dump — the types are the API
dump. `packages/react/src/forms/udf/README.md` is the reference for tone and depth.

**A task is not done until its `GUIDE.md` is current.** Documentation written a week later is
written from memory, and the reason a decision was made is the first thing memory loses.

### Folder shape

```
apps/web/src/features/<feature>/
├── GUIDE.md
├── api/            <feature>.api.ts, use<Feature>.ts       ← models + hooks
├── components/     presentational, no data fetching
├── pages/          route targets, wiring only
├── hooks/
├── utils/
├── types/
└── __tests__/
```

### Request / response models

One file per service, mirroring the wire, hand-maintained and commented.
`apps/web/src/api/udf.api.ts` is the reference. Hand-maintained rather than generated is a deliberate
cost: it is the only place the two sides meet, so it is worth being able to read.

- **Never send tenant, company, branch or application.** They come from the token. A client that can
  name its own tenant can read another customer's data — a defect this platform shipped with once.
- Query keys in `apps/web/src/api/queryKeys.ts`. **Every response-changing parameter goes in the
  key**, scope and culture included.
- Staleness from `STALE`; never a literal.

[INTEGRATION_STANDARD.md](./INTEGRATION_STANDARD.md) is binding, not advisory.

### Reusable components

New UI goes in `packages/react` when a second feature could plausibly want it, and in
`apps/web/src/components` when it is genuinely one screen's. Bias toward the package: promoting later
is a moved file; not promoting is a second implementation that drifts.

---

## Design system

**shadcn/ui + Tailwind**, already in `packages/react/src/forms/components/ui`. Extend it; do not
introduce a second component library.

| | |
|---|---|
| **Theme** | Light-first with a full dark palette. Tokens on bare `:root`, redefined under `@media (prefers-color-scheme: dark)` and `[data-theme="dark"]` so an explicit toggle wins both ways. |
| **Brand** | Existing `brand-*` scale — carbon greens and slate, with a warm accent on commerce surfaces so "you are about to spend money" reads differently from "you are configuring something". |
| **Density** | Compact by default. An operator console people live in all day; vertical space is the scarce resource. |
| **Motion** | Purposeful only — state transitions, optimistic feedback, progress. Respect `prefers-reduced-motion`. |
| **Illustration** | AI-generated SVG/GIF approved for empty states, onboarding and success moments. Inline or data URIs; no external hosts. |
| **Feel** | The "living" quality comes from **responsiveness and honesty**, not decoration: optimistic updates, skeletons matching the real layout, progress that names its phase, errors that say what to do next. A page that explains why a button is disabled feels better built than one that animates. |

---

## Weekly delivery, not sprints

Every Friday something works end to end and can be shown — not a demo of components, a path a person
can walk.

| Week | Days | Theme | **Friday delivery** |
|---|---|---|---|
| **W1** | 19–21 Aug *(3 days)* | Foundation | Themed shell, error pages with diagnostic reporting, scope switcher |
| **W2** | 24–28 Aug | Identity & access | Sign in as any of the four Ace users; navigation, data and available applications differ |
| **W3** | 31 Aug – 4 Sep | Tenancy & provisioning | Provision tenant → company → branch → industry → unit, end to end |
| **W4** | 7–11 Sep | Commerce | Buy a plan with add-ons and quota; live pricing, promo, wallet, invoice |
| **W5** | 14–18 Sep | Configuration & rules | Build a form from the builder; author a rule and dry-run it; calculate emissions |
| **W6** | 21–25 Sep | Workflow, billing, biomethane | Run a biomethane batch through approval; produce an invoice and an offset plan |
| **W7** | 28 Sep *(1 day)* | Freeze | Tagged, deployed, smoke-tested |

**Why weekly and not fortnightly.** Six of the seven weeks carry a hard external dependency —
regulatory rules, a payment gateway, a certifier's data shape. A two-week loop finds a wrong
assumption on day ten; a one-week loop finds it on day four, with five weeks left to absorb it.

---

## Roadmap

Legend: ☐ not started · ◐ in progress · ☑ done · **Cx** Low / Med / High

Task prefixes: `F` foundation · `A` auth & access · `T` tenancy & ingestion · `C` commerce ·
`D` configuration · `R` rules & compliance · `E` calculation · `W` workflow · `B` biomethane ·
`O` offset · `G` governance & release

### W1 — Foundation · 19–21 Aug

*Delivery: a themed shell with working error handling and a scope switcher.*

| ☐ | Date | Dev | Module | Task & subtasks | Cx | Hrs | Comments |
|---|---|---|---|---|---|---|---|
| ☑ | 19 Aug | Dev - RS | Foundation | `F1` **Design tokens & theme** — light/dark triple definition; brand scale; density; motion under `prefers-reduced-motion` | M | 4 | Everything else consumes this. Never define a colour only inside a media query. |
| ☐ | 19 Aug | Dev-AG | Foundation | `F3` **Error handling & diagnostics** — error boundary; 403/404/500; diagnostic report with **recursive redaction** | H | 4 | A diagnostic shipping a token or PII is a breach, not a feature. |
| ◐ | 19 Aug | Dev - VR | Identity | `A6` **Membership grant model (backend)** — one grant table; scope columns, application, validity, grant type, descendants | H | 4 | The spine of W2. See [IDENTITY-AND-SCOPE.md](./IDENTITY-AND-SCOPE.md). |
| ◐ | 20 Aug | Dev - VR | Foundation | `F2` **App shell** — sidebar, top bar, ⌘K palette, breadcrumbs; **nav hides on missing permission** | M | 4 | Mount `TenantSwitcher` here — built last session, not yet placed. |
| ☐ | 20 Aug | Dev | Identity | `A7` **Application gating** — refuse token issuance for an application with no grant; `app` claim; route check | H | 4 | The "web but not mobile" requirement, enforced at issuance rather than in the UI. |
| ◐ | 20 Aug | Dev - RS | Foundation | `F5` **Scope context** — PLATFORM→UNIT + application; switcher; **scope in every query key**; invalidate on switch | H | 4 | A cache surviving a scope switch shows another company's data. |
| ☐ | 21 Aug | Dev | Identity | `A8` **Effective-access resolver** — union for permissions, most-specific-wins for settings, deny beats both | H | 4 | The union/most-specific split is the subtle part. |
| ☐ | 21 Aug | Dev | Foundation | `F7` **Shared UI primitives** — page header, empty state, stat tile, scope badge, confirm dialog, drawer, illustration slot | M | 4 | Into `packages/react`, not `apps/web`. |
| ☐ | 21 Aug | Dev | Foundation | `F4` **Model & hook conventions** — extend the integration standard; scaffold script; scope-aware query-key helper | L | 3 | Land before others write API files. |

### W2 — Identity & access · 24–28 Aug

*Delivery: sign in as any of the four Ace users; navigation, data and applications differ correctly.*

| ☐ | Date | Dev | Module | Task & subtasks | Cx | Hrs | Comments |
|---|---|---|---|---|---|---|---|
| ☐ | 24 Aug | Dev | Auth | `A9` **Tenant & company chooser** — post-login selection when membership spans several; skip when there is one | M | 4 | `/auth/tenants` and `/auth/context/switch` exist. This is the screen. |
| ☐ | 24 Aug | Dev | RBAC | `A10` **Roles UI** — list, create, edit, clone, scope badge; delete refused while assigned, with the list shown | M | 4 | Follow the UDF definition-delete pattern. |
| ☐ | 24 Aug | Dev | RBAC | `A11` **Permission catalogue UI** — resource/action tree, route rules, search; platform rows read-only *with a reason* | M | 4 | |
| ☐ | 25 Aug | Dev | Identity | `A12` **Membership management UI** — grant access at any level, for named applications, with validity dates | H | 4 | The screen the whole identity model exists for. |
| ☐ | 25 Aug | Dev | RBAC | `A13` **Role→permission matrix** — bulk toggle, inherited vs explicit, **diff before save** | H | 4 | A matrix saved blind is how privilege creeps. |
| ☐ | 25 Aug | Dev | RBAC | `A14` **Effective-permission preview** — pick a user and a scope; see what they can do and *which grant said so* | H | 4 | Provenance is the feature. |
| ☐ | 26 Aug | Dev | Identity | `A15` **Parent/sub-tenant hierarchy** — parent link, `IncludesDescendants`, aggregated read for parent admins | H | 4 | Test all three directions. The sibling one ships broken. |
| ☐ | 26 Aug | Dev | Auth | `A16` **MFA enrolment** — TOTP enrol, verify, recovery codes; **mandatory for admin roles** | H | 4 | Backend has `MfaEnabled`. Policy is per role, per scope. |
| ☐ | 26 Aug | Dev | Identity | `A17` **Users management** — invite, deactivate, resend, view a user's grants across tenants | M | 4 | |
| ☐ | 27 Aug | Dev | Identity | `A18` **Support/impersonation access** — explicit, time-boxed, audited grant type; persistent banner while active | H | 4 | Support access that looks like normal access cannot be investigated afterwards. |
| ☐ | 27 Aug | Dev | Auth | `A19` **Session management UI** — active sessions, device, last seen, revoke; revoke-others after password change | M | 4 | |
| ☐ | 27 Aug | Dev | Tenancy | `T1` **UNIT scope (backend)** — entity, `IUdfScoped`, specificity 16, every resolver, migrations, tests | H | 4 | **Critical path.** Every resolver in one change. |
| ☐ | 28 Aug | Dev | Tenancy | `T1b` **UNIT wiring** — `al.master` endpoints, permission entries, `al.udf` catalogue | M | 3 | Finish before `C4` or `D3` start. |
| ☐ | 28 Aug | Dev | QA | `A20` **Access test matrix** — four Ace users × five scope levels × two applications, asserted end to end | H | 4 | The week's delivery. A suite, not a checklist. |
| ☐ | 28 Aug | Dev | Docs | `A21` **Identity GUIDE.md** + reconcile [IDENTITY-AND-SCOPE.md](./IDENTITY-AND-SCOPE.md) with what was built | L | 3 | |

### W3 — Tenancy, ingestion & form builder · 31 Aug – 4 Sep

*Delivery: provision a full organisation and upload validated activity data.*

| ☐ | Date | Dev | Module | Task & subtasks | Cx | Hrs | Comments |
|---|---|---|---|---|---|---|---|
| ☐ | 31 Aug | Dev | Tenancy | `T2` **Tenant provisioning wizard** — details, plan, quota, admin user, review; show `DesiredVersion`/`AppliedVersion` | H | 4 | Provisioning is async and versioned. Not a spinner. |
| ☐ | 31 Aug | Dev | Tenancy | `T3` **Company onboarding (UDF-driven)** — render from `GET /udf/forms/master/Company`; industry reveals fields; `split()` | M | 4 | Do **not** hand-code the static half. |
| ☐ | 31 Aug | Dev | UDF | `D1a` **Form builder — placement** — static catalogue + custom definitions, drag to order, colspan, per-application visibility | H | 4 | Mandatory columns cannot be hidden; explain rather than disable. |
| ☐ | 1 Sep | Dev | Tenancy | `T4` **Branch, industry & unit management** — CRUD; several industries per branch; unit tree | M | 4 | Depends on `T1`. |
| ☐ | 1 Sep | Dev | Tenancy | `T5` **Facility / site model** — sites under a branch, with geography, capacity, grid connection | M | 4 | Feeds the emission and offset engines. |
| ☐ | 1 Sep | Dev | UDF | `D1b` **Form builder — labels & i18n** — per-scope override, translation editor with missing-language indicator | H | 4 | Overrides overlay, never substitute. Use `HasTranslation`. |
| ☐ | 2 Sep | Dev | Tenancy | `T6` **Provisioning status console** — snapshot state, retry a failed step, name the step that failed | M | 4 | |
| ☐ | 2 Sep | Dev | Ingestion | `T7` **Data collection forms** — Scope 1/2/3 activity entry, UDF-driven, per site | H | 4 | Bulk upload already exists. |
| ☐ | 2 Sep | Dev | UDF | `D1c` **Form builder — sections & conditions** — section CRUD, step flag, condition builder, **live preview using the real renderer** | H | 4 | A preview that diverges from the renderer is worse than none. |
| ☐ | 3 Sep | Dev | Ingestion | `T8` **Five-layer validation pipeline** — format, schema, type, business rule, completeness | H | 4 | Layer 4 delegates to `al.rules`. Do not build a second evaluator. |
| ☐ | 3 Sep | Dev | Ingestion | `T9` **Data quality & confidence scoring** — per-record score, surfaced on every derived figure | H | 4 | An unscored number in an audited disclosure is a liability. |
| ☐ | 3 Sep | Dev | UDF | `D2` **List-view columns per scope** — `uvt-table` resolution + DataGrid consumption + config UI | H | 4 | "10 columns for one scope, 8 for another." |
| ☐ | 4 Sep | Dev | QA | `T10` **Provisioning end-to-end pass** on a reset database | H | 4 | The week's delivery. |
| ☐ | 4 Sep | Dev | Platform | `T11` **Platform master data console** — industries, applications, services, data types, seed status; gated on `IsPlatform` | M | 4 | Customer tenants must not see the screen at all. |
| ☐ | 4 Sep | Dev | Docs | `T12` **Tenancy + form-builder GUIDE.md** | L | 3 | |

### W4 — Commerce · 7–11 Sep

*Delivery: buy a plan with add-ons and quota, with live pricing, promo, wallet and an invoice.*

| ☐ | Date | Dev | Module | Task & subtasks | Cx | Hrs | Comments |
|---|---|---|---|---|---|---|---|
| ☐ | 7 Sep | Dev | Commerce | `C1` **Subscription plans admin** — FREE/TRIAL/PRO/ENTERPRISE, included quota, periodic wallet units | M | 4 | Label rollover "top up to" vs "add on top". False means top-up-to. |
| ☐ | 7 Sep | Dev | Commerce | `C2` **Feature management** — FREE/PAID/ADDON/BETA/NOT_ALLOWED, burn rate per feature | M | 4 | Validate sub-cent rates — one rounding to zero makes a paid feature free. |
| ☐ | 7 Sep | Dev | Commerce | `C3` **Pricing engine (backend)** — [COST.md](./COST.md) as a server-side calculator | H | 4 | The client never computes a total. |
| ☐ | 8 Sep | Dev | Commerce | `C4` **Quota configuration by scope** — allocation, overrides, **ceiling enforcement**, usage bars | H | 4 | Test 1 TB tenant / 25 GB role / 20 & 30 GB users. |
| ☐ | 8 Sep | Dev | Commerce | `C5` **Buy flow** — plan compare, quota sliders, add-on picker, live breakdown, promo, confirm | H | 4 | Base, add-ons, discount, tax as separate lines. |
| ☐ | 8 Sep | Dev | Commerce | `C6` **Wallet & credits UI** — balance per scope, top-up, burn history, low-balance alert | M | 4 | Wallets resolve nearest-first. Show **which** wallet paid. |
| ☐ | 9 Sep | Dev | Commerce | `C7` **Promo codes & discounts** — benefit types, feature targeting, windows, redemptions, refusals | M | 4 | Discounts round **down**; overage rounds **up**. |
| ☐ | 9 Sep | Dev | Commerce | `C8` **Payment gateway (backend)** — abstraction, charge, refund, **signature-verified idempotent webhooks** | H | 4 | A retried webhook must not double-charge. |
| ☐ | 9 Sep | Dev | Commerce | `C9` **Metering & usage capture** — per-feature events feeding quota, wallet and invoice from one source | H | 4 | Three consumers, one event stream. |
| ☐ | 10 Sep | Dev | Commerce | `C10` **Invoice generation (backend)** — line items, discounts, credits, totals, tax; **fixed documented order**; PDF via job | H | 4 | Discount-before-credit changes the total. Pin it. |
| ☐ | 10 Sep | Dev | Commerce | `C11` **Payment configuration UI** — gateway per scope, secret references only, test mode, connection check | M | 4 | Never render a stored secret. |
| ☐ | 10 Sep | Dev | Commerce | `C12` **Shared vs dedicated infrastructure tier** — infra class drives cost model and provisioning | H | 4 | See [COST.md](./COST.md). |
| ☐ | 11 Sep | Dev | Commerce | `C13` **Invoice UI** — list, detail, line items, download, payment status, retry | M | 4 | If `C8`/`C10` slip, ship read-only. |
| ☐ | 11 Sep | Dev | QA | `C14` **Commerce end-to-end pass** on a reset database | H | 4 | The week's delivery. |
| ☐ | 11 Sep | Dev | Docs | `C15` **Commerce GUIDE.md** + reconcile [COST.md](./COST.md) with what shipped | L | 3 | |

### W5 — Configuration, rules & calculation · 14–18 Sep

*Delivery: build a form, author a rule, dry-run it, and calculate emissions with an audit trail.*

| ☐ | Date | Dev | Module | Task & subtasks | Cx | Hrs | Comments |
|---|---|---|---|---|---|---|---|
| ☐ | 14 Sep | Dev | Config | `D3` **Provider bindings UI** — capability × channel × scope; per-branch selection; **disabled ≠ inherited** | H | 4 | Show the per-channel cost here — SMS is ~100× email. |
| ☐ | 14 Sep | Dev | Rules | `R1` **Condition group builder** — arbitrarily nested AND/OR, JSON preview | H | 4 | Build it reusable — `W1` needs it. |
| ☐ | 14 Sep | Dev | Compliance | `R0` **Extract EU requirements** → `docs/compliance/` | M | 4 | **Flag open questions rather than resolving them.** Compliance sign-off required. |
| ☐ | 15 Sep | Dev | Config | `D4` **Templates & assignments by scope** — import and notification templates, assignment matrix | M | 4 | |
| ☐ | 15 Sep | Dev | Rules | `R2` **Rule assignment by scope** — resource and event, priority, enable/disable, **dry-run** | H | 4 | Without dry-run people test in production. |
| ☐ | 15 Sep | Dev | Rules | `R3` **Rule catalogue fixtures** — encode [RULES-AND-WORKFLOWS.md](./RULES-AND-WORKFLOWS.md) as `R1` acceptance tests | M | 4 | `R-FEED-02` and `R-MB-01` are the stress cases. |
| ☐ | 16 Sep | Dev | Config | `D5` **Preferences & configuration by scope** — resolution preview naming **which level won** | M | 4 | Provenance is the whole value. |
| ☐ | 16 Sep | Dev | Calc | `E1` **Emission factor service** — third-party lookup with local cache fallback | H | 4 | Behind an interface. Cache is a 15× cost reduction. |
| ☐ | 16 Sep | Dev | Calc | `E2` **Scope 1 & 2 calculation** — location-based **and** market-based, stored separately | H | 4 | Never derive one from the other at render time. |
| ☐ | 17 Sep | Dev | Calc | `E3` **Scope 3 calculation** — the 15 categories, per-category method and confidence | H | 4 | |
| ☐ | 17 Sep | Dev | Calc | `E4` **Audit snapshot** — immutable inputs, factors, versions, method, actor | H | 4 | A figure that cannot be reproduced cannot be defended. |
| ☐ | 17 Sep | Dev | Rules | `R4` **Rule execution monitor** — what fired, on what, with what outcome; replay | M | 4 | |
| ☐ | 18 Sep | Dev | QA | `E5` **Calculation end-to-end pass** with a Spanish-locale run | H | 4 | The week's delivery. |
| ☐ | 18 Sep | Dev | Config | `D6` **Audit, telemetry & queue strategy per scope** — same pattern as `D3` | M | 4 | |
| ☐ | 18 Sep | Dev | Docs | `E6` **Rules + calculation GUIDE.md** | L | 3 | |

### W6 — Workflow, billing & biomethane · 21–25 Sep

*Delivery: a biomethane batch for Ace / islabhasolutions / Spain runs through approval and produces
an invoice and an offset plan.*

| ☐ | Date | Dev | Module | Task & subtasks | Cx | Hrs | Comments |
|---|---|---|---|---|---|---|---|
| ☐ | 21 Sep | Dev | Workflow | `W1` **Stage designer** — stages, transitions, approval config, assignee by role and scope, visual graph | H | 4 | **Reuse `R1`'s condition builder.** Two editors will drift. |
| ☐ | 21 Sep | Dev | Biomethane | `B1` **Batch ledger** — batch, feedstock, origin, intensity, custody, certificate; mass-balance period and site | H | 4 | A batch, not a meter reading, is the unit of account. |
| ☐ | 21 Sep | Dev | Offset | `O1` **Offset measure library** — per-industry measures with cost, abatement, complexity, payback | H | 4 | See [OFFSET.md](./OFFSET.md). |
| ☐ | 22 Sep | Dev | Workflow | `W2` **Time-triggered starts & async failure** — a workflow beginning on a date; a stage reversed later | H | 4 | `W-CERT` and `W-ALLOC`. Saga compensation. |
| ☐ | 22 Sep | Dev | Biomethane | `B2` **GHG intensity calculator** — the Annex VI chain, with per-term provenance | H | 4 | Every term shown and sourced. |
| ☐ | 22 Sep | Dev | Offset | `O2` **Offset plan generator** — `(abatement × ROI) / complexity`; quick wins, medium, long | H | 4 | Rule-based. AI ranking is deferred. |
| ☐ | 23 Sep | Dev | Workflow | `W3` **Runtime monitor** — instances, stage, history, manual advance/cancel **with a required reason** | M | 4 | The only audit trail an override leaves. |
| ☐ | 23 Sep | Dev | Biomethane | `B3` **Claim register** — one cross-register uniqueness point for UDB, GO, ETS and disclosure | H | 4 | Build it **vertical-agnostic**. `R-DC-01` is not disableable. |
| ☐ | 23 Sep | Dev | Offset | `O3` **Offset plan UI** — roadmap view, per-measure detail, branch and location overrides | M | 4 | Measures differ by branch: grid factors, permits, incentives. |
| ☐ | 24 Sep | Dev | Workflow | `W4` **Immutable terminal state** — a locked disclosure no administrator can reopen | H | 4 | `W-CSRD`. |
| ☐ | 24 Sep | Dev | Biomethane | `B4` **PoS & certificate management** — issue, attach, validity, expiry alerting, scheme per scope | H | 4 | A lapsed certificate blocks allocation. |
| ☐ | 24 Sep | Dev | Biomethane | `B5` **Ace vertical slice** — seed islabhasolutions / Spain / biomethane with real rules, workflows, offset plan | H | 4 | The week's delivery, and the demo. |
| ☐ | 25 Sep | Dev | Compliance | `B6` **ESRS E1 disclosure builder** — gross and market-based separate, methodology refs, locked output | H | 4 | Low-carbon fuel, **not** an offset. Never netted. |
| ☐ | 25 Sep | Dev | Biomethane | `B7` **UDB submission stub** — transaction shape, queue, retry, backlog alerting | M | 4 | Shape it so live integration is a provider swap. |
| ☐ | 25 Sep | Dev | QA | `B8` **Biomethane end-to-end pass** — batch → evidence → approval → allocation → claim → disclosure | H | 4 | |

### W7 — Freeze & release · 28 Sep

| ☐ | Date | Dev | Module | Task & subtasks | Cx | Hrs | Comments |
|---|---|---|---|---|---|---|---|
| ☐ | 28 Sep | All | QA | `G1` **Cross-review** — each dev reviews a track they did not build; P0 only | M | 2 | Look for a rule enforced client-side only, and a scope missing from a query key. |
| ☐ | 28 Sep | All | Docs | `G2` **Documentation sync** — every `GUIDE.md` current, this folder reconciled, session log closed | M | 2 | The `F4`/`F6` CI check must pass. |
| ☐ | 28 Sep | Dev | Release | `G3` **Code freeze** — tag, changelog, ordered migration list, **written rollback plan** | L | 1 | Write the rollback before deploying. |
| ☐ | 28 Sep | Dev | Release | `G4` **Deploy & smoke** — migrations in order, seeders, health checks, smoke suite | M | 2 | Migration ordering has bitten this project before. |
| ☐ | 28 Sep | Dev | Release | `G5` **Handover pack** — demo script, known limitations, deferred backlog with estimates | M | 2 | |

---

## Deferred backlog

Not in the window. Each estimated so scheduling stays a decision.

| ID | Item | Est. | Why deferred |
|---|---|---|---|
| `X1` | Live UDB integration | 20h | Needs credentials and the production schema; `B7` shapes the seam. |
| `X2` | ETS1/ETS2 attribution engine | 16h | Obligations begin 2027. `B3` holds the claim uniqueness it depends on. |
| `X3` | Power BI embedding & report designer | 16h | PDF reporting covers W6; embedded analytics is a separate surface. |
| `X4` | AI-ranked offset recommendations | 20h | `O2` is rule-based by design. AI ranking needs outcome data we will not have until pilots run. |
| `X5` | SSO / OIDC tenant configuration UI | 10h | `OidcTokenValidator` exists; not needed for the Ace cohort. |
| `X6` | Reseller billing & tenant revenue splits | 12h | Needs a commercial decision on the split model. |
| `X7` | Digital twin / IT-OT telemetry ingestion | 24h+ | A different ingestion class from batch upload. |
| `X8` | Verticals beyond biomethane (H₂/PtX, CCUS, heat, water, waste) | 20h each | Taxonomy exists, so each is configuration plus a measure library. |
| `X9` | Workflow saga compensation **authoring** UI | 8h | `W2` supports it; authoring via API until then. |
| `X10` | Full accessibility audit | 8h | Do it once the screens stop moving. |
| `X11` | Mobile application | — | The scope model supports it from `A7`; the client is out of window. |

---

## New files by area

What the roadmap creates, so a reviewer can see the shape before it exists.

### `al.net.packages` (SDK)

```
al.net.entities/Master/UnitCatalog.cs                       T1
al.net.entities/Identity/Grant.cs                           A6
al.net.payments/                                            C8   ← new package
  Abstractions/IPaymentProvider.cs
  Services/PaymentService.cs
  README.md
al.net.udf/Models/UdfTableView.cs                           D2
```

### `Carbon-Atlas-Services`

```
al.master/Api/GrantEndpoints.cs                             A6
al.master/Api/UnitEndpoints.cs                              T1b
al.master/Data/GrantSeeder.cs                               A6
al.auth/Services/ApplicationGate.cs                         A7
al.telemetry/Api/DiagnosticsEndpoints.cs                    F3
al.subscription/Services/PricingEngine.cs                   C3
al.subscription/Services/InvoiceGenerator.cs                C10
al.subscription/Jobs/InvoicePdfJob.cs                       C10
al.carbon/Services/EmissionFactorService.cs                 E1
al.carbon/Services/ScopeCalculator.cs                       E2 E3
al.carbon/Entities/CalculationSnapshot.cs                   E4
al.carbon/Entities/Batch.cs, Feedstock.cs, GhgIntensity.cs  B1
al.carbon/Services/GhgIntensityCalculator.cs                B2
al.carbon/Entities/Claim.cs                                 B3
al.carbon/Entities/Certificate.cs, ProofOfSustainability.cs B4
al.carbon/Services/UdbSubmissionService.cs                  B7
al.offset/Entities/MeasureDefinition.cs, LocationFactor.cs  O1
al.offset/Services/OffsetPlanGenerator.cs                   O2
docs/compliance/EU_BIOMETHANE.md                            R0
```

### `Carbon-atlas-webapp/packages/react`

```
src/theme/                          tokens.css, ThemeProvider.tsx, README.md    F1
src/shared/                         PageHeader, EmptyState, StatTile,           F7
                                    ScopeBadge, ConfirmDialog, Drawer,
                                    IllustrationSlot
src/rules/                          ConditionGroupBuilder.tsx, README.md        R1
```

### `Carbon-atlas-webapp/apps/web/src/features`

```
scope/          provider, useScope, ScopeSwitcher, GUIDE.md          F5
rbac/           roles, permissions, matrix, preview, GUIDE.md        A10 A11 A13 A14
identity/       membership, users, support access, GUIDE.md          A12 A17 A18
tenancy/        provisioning, onboarding, branch/unit, GUIDE.md      T2–T6
ingestion/      activity entry, validation surface, GUIDE.md         T7–T9
form-builder/   placement, labels, sections, GUIDE.md                D1a–D1c
configuration/  providers, templates, preferences, GUIDE.md          D3–D6
commerce/       plans, features, buy, quota, wallet, promo,          C1–C13
                payment, invoices, GUIDE.md
rules/          assignment, dry-run, monitor, GUIDE.md               R2 R4
workflow/       designer, monitor, GUIDE.md                          W1–W4
carbon/         calculation views, snapshots, GUIDE.md               E2–E5
biomethane/     batches, intensity, claims, certificates, GUIDE.md   B1–B8
offset/         plan, measures, roadmap, GUIDE.md                    O1–O3
platform/       master data console, GUIDE.md                        T11
```

### Webapp root

```
scripts/scaffold-feature.mjs        F4
scripts/check-guides.mjs            F4
docs/GUIDE_TEMPLATE.md              F4
```

---

## AI prompts

One per task. Paste into a fresh agent session at the repository root. Each assumes the agent reads
the referenced files first.

Every prompt inherits these **standing rules** — repeat them if the agent drifts:

> Follow `Carbon-atlas-webapp/Deliverables/INTEGRATION_STANDARD.md`. Never send
> tenant/company/branch/application from the client — they come from the token. Put every
> response-changing parameter in the query key. Reuse `packages/react` components; do not add a
> second component library. Write or update the feature's `GUIDE.md`. Add tests for the decisions,
> not the getters. Do not commit.

### F1 — Design tokens & theme

> Create the Carbon Atlas design token layer in `packages/react/src/theme/`. Define the complete
> light palette as CSS custom properties on bare `:root`; redefine only the changed tokens under
> `@media (prefers-color-scheme: dark)` guarded as `:root:not([data-theme="light"])`, and again under
> `:root[data-theme="dark"]` so an explicit toggle wins in both directions. Never give a colour its
> only definition inside a media query. Include: brand scale (carbon greens + slate), semantic
> surfaces, a distinct accent for commerce surfaces, spacing/density tuned for an all-day operator
> console, and motion tokens that collapse under `prefers-reduced-motion`. Export a `ThemeProvider`
> with a persisted light/dark/system toggle. Add `packages/react/src/theme/README.md` explaining the
> three-state theming rule and why the triple definition is required.

### F2 — App shell

> Build the authenticated app shell in `apps/web/src/components/layout/`. Sidebar with grouped
> navigation, collapsible and responsive; top bar with a scope switcher slot, user menu and command
> palette (⌘K); breadcrumbs derived from the route. **Navigation items must hide when the user lacks
> the permission**, not appear and 403 on click — read effective permissions from the auth context.
> Extend `Sidebar.tsx` and `AppHeader.tsx` rather than replacing them, and **mount the existing
> `TenantSwitcher`** (`apps/web/src/features/auth/components/TenantSwitcher.tsx`) in the top bar — it
> was built and never placed. Add `apps/web/src/components/layout/GUIDE.md`.

### F3 — Error handling & diagnostics

> Build error handling for `apps/web`. (1) A route-level error boundary catching render errors and
> showing a recoverable page rather than a blank screen. (2) Dedicated 403, 404 and 500 pages saying
> what happened and what to do next. (3) A **diagnostic report** action posting to
> `/api/v1/telemetry/diagnostics`: correlation id, route, user agent, app version, and a **redacted**
> snapshot of relevant state. Redaction is the critical part — strip tokens, passwords, emails and
> anything under a `secret`/`password`/`token` key, **recursively**, before the payload leaves the
> browser. Add a unit test proving a nested token is stripped. Create the receiving endpoint in
> `al.telemetry`. Document the redaction contract in the feature's `GUIDE.md`.

### F4 — Model & hook conventions

> Extend `Carbon-atlas-webapp/Deliverables/INTEGRATION_STANDARD.md` with the request/response model
> convention, using `apps/web/src/api/udf.api.ts` and `useUdfForm.ts` as worked examples. Add a
> scaffold script (`scripts/scaffold-feature.mjs`) generating a feature folder — `GUIDE.md`, `api/`,
> `components/`, `pages/`, `__tests__/` — with the api file, hook file and query-key block already
> shaped correctly. Add a `queryKeys.ts` helper folding the active scope into a key. Add
> `docs/GUIDE_TEMPLATE.md` capturing the house documentation style (what it is for, the decisions and
> what the alternatives cost, how to use it, the pitfalls — see
> `packages/react/src/forms/udf/README.md` for tone), plus a CI check `scripts/check-guides.mjs`
> wired into the lint script that fails when a folder under `apps/web/src/features/` has no
> `GUIDE.md`.

### F5 — Scope context

> Build the scope context in `apps/web/src/features/scope/`. A provider holding
> PLATFORM/TENANT/COMPANY/BRANCH/INDUSTRY/UNIT plus the application, a `useScope()` hook, a scope
> switcher, and persistence across reloads. **Changing scope must invalidate every scope-dependent
> query** — a cache surviving a scope switch shows one company's data under another's heading. Fold
> the scope into query keys via the `F4` helper. Test that switching scope drops the previous scope's
> cached data. Write `GUIDE.md` covering the hierarchy, why application is orthogonal rather than a
> level, and the invalidation rule.

### F7 — Shared UI primitives

> Add to `packages/react/src/shared/`: `PageHeader` (title, description, actions, breadcrumb slot),
> `EmptyState` (illustration slot, heading, body, primary action), `StatTile`, `ScopeBadge` (renders
> which scope level a value came from), `ConfirmDialog`, `Drawer`, and `IllustrationSlot` for inline
> SVG. All theme-token driven, all responsive, none fetching data. Add tests for `EmptyState`'s
> filtered-versus-genuinely-empty distinction — "no users yet" and "no users match *admin*" ask for
> different actions. Update `packages/react/src/shared/README.md`.

### A6 — Membership grant model (backend)

> Replace the implicit access model in `al.master` with a single grant table. Read
> `Carbon-atlas-webapp/Deliverables/IDENTITY-AND-SCOPE.md` first — it specifies the shape.
> `Grant(userId, tenantId, companyId?, branchId?, industryKey?, unitId?, applicationId?, roleId,
> includesDescendants, grantType, validFrom?, validUntil?, status)`. **Null means "everything below",
> not "nothing"** — that is what makes the common cases one row. Migrate `TenantAccess` to
> tenant-level grants and `TenantUserRole` to scoped ones, with `applicationId` null so existing
> behaviour is preserved exactly. Add endpoints for listing and managing grants. Test the four Ace
> users resolve to the same effective access before and after the migration.

### A7 — Application gating

> Enforce per-application access in `al.auth`. A user holding no grant for an application must
> **never receive a token for it** — refuse at `/auth/login` and `/auth/context/switch` rather than
> hiding features in the client. Add an `app` claim to the token, and check it in route
> authorization against the route's allowed applications. Read
> `Carbon-atlas-webapp/Deliverables/IDENTITY-AND-SCOPE.md` § *Application gating* for why this
> belongs on the grant and not on the role. The refusal must be **specific** — "your account is not
> enabled for the mobile app" — not a generic 403, because the user has done nothing wrong and needs
> to know who to ask. Test: web-only user is refused a mobile token; the same user succeeds on web.

### A8 — Effective-access resolver

> Build the effective-access resolver in `al.net.authorization`. Given a user and an active context,
> select the applicable grants (scope matches, application matches, dates valid, status active) and
> resolve: **permissions union, configuration most-specific-wins, explicit denials beat both**. That
> split is the subtle part — a company-level reader plus a branch-level admin must be an admin at
> that branch *and* still a reader elsewhere, which most-specific-wins would silently break. Read
> `Carbon-atlas-webapp/Deliverables/IDENTITY-AND-SCOPE.md`. Use the four Ace users as fixtures, and
> add a test asserting the union behaviour explicitly.

### A9 — Tenant & company chooser

> Build post-login context selection in `apps/web/src/features/auth/`. After sign-in, if membership
> spans more than one tenant, show a chooser; if exactly one, go straight through — a control with
> one option is not a choice. Use the existing `useAccessibleTenants` and `useSwitchContext` hooks.
> Extend to company selection where a tenant grant spans several. Test that the single-tenant path
> renders no chooser at all.

### A10 — Roles UI

> Build role management in `apps/web/src/features/rbac/`. List with scope badges, create, edit,
> clone. Deleting a role that is still assigned must be **refused with the assignment list shown**,
> so the admin can act — follow the UDF definition-delete pattern in `al.udf/Api/UdfEndpoints.cs`.
> Use `DataGrid` for the list and `DataGridFormDialog` for editing. Write
> `apps/web/src/features/rbac/GUIDE.md` covering scope-bound roles and why deletion is guarded.

### A11 — Permission catalogue UI

> Build the permission catalogue viewer in `apps/web/src/features/rbac/`. A resource → action tree
> from `al.master`'s permission catalogue, with search, plus the route-permission rules mapping
> endpoints to permissions. Platform-owned entries are **visible and not editable** — render them
> read-only **with an explanation of why**, rather than silently disabling the control. Read
> `al.master/Data/PermissionCatalogSeeder.cs` for the shape.

### A12 — Membership management UI

> Build grant management in `apps/web/src/features/identity/`. Grant a user access at any scope
> level, for named applications, with validity dates and a grant type. The screen the whole identity
> model exists for — read `Carbon-atlas-webapp/Deliverables/IDENTITY-AND-SCOPE.md` first. Use the
> scope picker from `F5`. Make **null-means-everything-below** legible in the UI: an empty company
> field must read as "all companies", not as an unfilled input. Show a user's existing grants across
> every tenant they belong to.

### A13 — Role→permission matrix

> Build the role-permission matrix in `apps/web/src/features/rbac/`. A grid of roles × permissions
> with bulk toggle by row, column and group, distinguishing **explicit** grants from ones
> **inherited** through role hierarchy. Before saving, show a **diff** — added and removed
> permissions, named. A permission matrix saved blind is how privilege quietly creeps. Virtualise
> the grid; the catalogue is large. Test that inherited entries cannot be toggled directly.

### A14 — Effective-permission preview

> Build the effective-permission preview in `apps/web/src/features/rbac/`. Given a user and a chosen
> scope, show what they can actually do **and which grant said so**. The provenance is the feature:
> "what can this person do" is otherwise unanswerable, and "why can they see this" becomes a
> developer question. Resolution is server-side via `A8` — the client must not re-implement
> most-specific-wins. Use the four Ace users from `al.master/Data/AceTenantSeeder.cs` as fixtures.

### A15 — Parent/sub-tenant hierarchy

> Add one-level tenant nesting in `al.master` and the corresponding UI. A `parentTenantId` on the
> tenant, plus `includesDescendants` on a grant — **both are required, and the flag is the important
> one**: a parent tenant existing does not mean every grant in it should reach downward. A parent
> admin reads aggregated sub-tenant data; a sub-tenant sees neither the parent nor its siblings.
> **Aggregation is a read, never a write** — acting *in* a sub-tenant means switching context, so the
> audit trail records who really did it. Test all three directions; the sibling one is the one that
> ships broken.

### A16 — MFA enrolment

> Build MFA in `apps/web/src/features/auth/` over `al.auth`'s existing `MfaEnabled`. TOTP enrolment
> with a QR code, verification, and single-use recovery codes shown once. **Mandatory for admin
> roles**, optional otherwise — the policy resolves per role and per scope, not as a global flag.
> Handle the lost-device path via recovery codes, and make clear at enrolment that the codes are the
> only way back. Test that an admin without MFA is prompted and cannot dismiss it.

### A17 — Users management

> Build user administration in `apps/web/src/features/identity/`. Invite (email, initial grant),
> deactivate, resend invite, and view a user's grants across every tenant they belong to. Use
> `DataGrid`. Deactivation must be reversible and must **not** delete grants — "this person has left"
> and "this person never had access" are different facts, and only the first stays answerable if the
> grants survive.

### A18 — Support/impersonation access

> Add a support access path in `al.master` and `apps/web/src/features/identity/`. A grant with
> `grantType = Support`, a **required** `validUntil`, and a reason recorded at creation. Every action
> taken under it is tagged in the audit log, and a persistent banner shows while it is active. Read
> `Carbon-atlas-webapp/Deliverables/IDENTITY-AND-SCOPE.md` § *Support access*. The alternative — a
> standing grant into every tenant — costs the ability to answer "who looked at this record", which
> is the question asked after an incident.

### A19 — Session management UI

> Build session management in `apps/web/src/features/auth/`. List active sessions with device, IP,
> location and last-seen; revoke one; revoke all others. Offer revoke-all-others after a password
> change — but **do not do it silently**, because it strands the user's other devices with no
> explanation. Read `al.auth`'s `ISessionStore` for the available fields.

### T1 / T1b — UNIT scope

> Add `UNIT` as the narrowest level of the scope hierarchy, below INDUSTRY. In `al.net.packages`: a
> `UnitCatalog` entity under `al.net.entities/Master`; `UnitId` on `IUdfScoped`, `UdfScope`,
> `ProviderBinding`, quota allocation, wallet scope, rule assignment and the grant from `A6`;
> specificity weight **16** in `UdfScopes.Specificity` and every sibling resolver. In
> `Carbon-Atlas-Services`: `al.master` CRUD endpoints, permission catalogue entries, `al.udf`
> catalogue declaration, migrations. **Every resolver that scopes must learn UNIT in this one
> change** — a resolver left behind creates a second resolution order that disagrees with the first,
> and the disagreement surfaces months later. Extend the scope tests to cover unit-beats-branch, and
> add a test asserting every `IUdfScoped` implementation exposes `UnitId`.

### T2 — Tenant provisioning wizard

> Build the tenant provisioning wizard in `apps/web/src/features/tenancy/`. Steps: details, plan
> selection, quota allocation, first admin user, review. Submit to `al.master` provisioning.
> Provisioning is **asynchronous and versioned** (`DesiredVersion` / `AppliedVersion` on the
> snapshot), so show which version is applied and what is still pending rather than an indefinite
> spinner. Handle partial failure by naming the step that failed and offering a retry. Write
> `apps/web/src/features/tenancy/GUIDE.md`.

### T3 — Company onboarding (UDF-driven)

> Build company onboarding in `apps/web/src/features/tenancy/`. Render the form entirely from
> `useUdfForm({ module: 'master', resource: 'Company' })` — **do not hand-code the static fields**;
> the static-field catalogue publishes them and hand-coding defeats the whole design. Selecting an
> industry reveals industry-specific fields via the server's conditions. On submit use `split()` to
> route values to the entity and the custom-field bag. Read `packages/react/src/forms/udf/README.md`
> first. Test with a biomethane selection revealing extra required fields and a manufacturing
> selection not requiring them.

### T4 — Branch, industry & unit management

> Build branch, industry and unit management in `apps/web/src/features/tenancy/`. Branch CRUD;
> assigning one or more industries to a branch (a branch legitimately runs several); a unit tree
> beneath branch + industry. Depends on `T1`. Use `DataGrid` for lists and the scope context for the
> active company. Update the feature `GUIDE.md`.

### T5 — Facility / site model

> Add sites beneath a branch in `al.master` and `apps/web/src/features/tenancy/`. A site carries
> geography (for grid emission factors and local incentives), capacity, and grid connection details.
> This is the entity the emission engine calculates against and the offset engine assesses measures
> for — see `Deliverables/OFFSET.md` on why a measure's value is a property of *(measure × site)*.
> Include a `LocationFactor` record per site: grid factor, energy price, incentive scheme, permit
> regime.

### T6 — Provisioning status console

> Build the provisioning console in `apps/web/src/features/tenancy/`. Snapshot state per tenant,
> which version is desired and which applied, per-step status, and retry for a failed step. **Name
> the step that failed** — "provisioning failed" is unactionable, "storage container creation failed:
> quota exceeded" is a ticket somebody can close.

### T7 — Data collection forms

> Build Scope 1/2/3 activity data entry in `apps/web/src/features/ingestion/`, per site and period.
> UDF-driven so a tenant can add activity fields without a release. Bulk upload already exists
> (`al.ingestion` + `packages/react/src/forms/bulk`) — this is the manual path beside it, and both
> must produce the same validated shape. Read `Deliverables/ARCHITECTURE.md` § 4.2.

### T8 — Five-layer validation pipeline

> Implement the five-layer validation pipeline in `al.ingestion`: (1) format — file type, encoding,
> size; (2) schema — required columns; (3) type — numerics parse, dates valid; (4) business rules;
> (5) completeness — critical fields populated. Each layer reports its own errors so a user is told
> which one rejected them. **Layer 4 delegates to `al.rules`** — building a second rule evaluator
> inside ingestion is how an import accepts a row the API would reject, and the disagreement is
> discovered after the data is in.

### T9 — Data quality & confidence scoring

> Add per-record confidence scoring in `al.ingestion` and `al.carbon`. Score from data source
> (measured / calculated / estimated), method, and completeness. **Surface the score on every derived
> figure**, not just at import — an unscored number in an audited disclosure is a liability, and the
> auditor's first question is how confident you are in the input.

### T11 — Platform master data console

> Build the platform administration console in `apps/web/src/features/platform/`. Manage industries,
> applications, services, UDF data types and view types, and show seeder status per service. **Gate
> the whole feature on the platform tenant** (`TenantCatalog.IsPlatform`) — these are platform-owned
> records and a customer tenant must not see the screen at all, not merely be refused on save.

### C1 — Subscription plans admin

> Build subscription plan administration in `apps/web/src/features/commerce/`. CRUD over
> `SubscriptionPlan` covering FREE / TRIAL / PRO / ENTERPRISE, included quotas, and periodic wallet
> units. **Label `PeriodicWalletUnitsRollOver` in the UI as "top up to" versus "add on top"** — false
> means top-up-to, and a plan configured on the wrong reading gives away or withholds credits every
> period. Read `al.net.entities/Subscription/SubscriptionPlan.cs` and `Deliverables/COST.md` for the
> plan matrix. Write `apps/web/src/features/commerce/GUIDE.md`.

### C2 — Feature management

> Build feature management in `apps/web/src/features/commerce/`. CRUD over platform features with
> classification (FREE / PAID / ADDON / BETA / NOT_ALLOWED) and a burn rate per feature — units or
> currency. **Validate against sub-cent rates**: a rate rounding to zero silently makes a paid
> feature free, which is a revenue loss nobody notices. Show the effective cost of a feature per plan.

### C3 — Pricing engine (backend)

> Implement the pricing engine in `al.subscription` from `Deliverables/COST.md`. Compute:
> `platform fee + Σ add-ons + Σ over-allowance consumption × unit price × volume multiplier −
> discount − wallet credits + tax`. **Discounts round down; overage rounds up** — opposite directions,
> both deliberate. **Fix and document the order of operations** (discount before or after credit
> changes the total) and pin it with a test against the worked example in COST.md. Expose a quote
> endpoint the buy flow calls, so **the client never computes a total** — a client figure that
> disagrees with the invoice is a chargeback.

### C4 — Quota configuration by scope

> Build quota configuration in `apps/web/src/features/commerce/`. Allocate quota per scope level and
> per role/user override, with usage bars. The rule to enforce and make visible: **the sum of
> allocations below can never exceed the ceiling above**. Show remaining headroom at each level and
> warn before an over-allocation is saved. Test fixture: a tenant with a 1 TB subscription
> allocation, a role with 25 GB, and users overridden to 20 GB and 30 GB — the user overrides are
> legal; the sum against the tenant ceiling is what must be checked. Read
> `al.net.data/Authorization/EfQuotaGuard.cs`.

### C5 — Buy flow

> Build the subscription purchase flow in `apps/web/src/features/commerce/`. Plan comparison, quota
> sliders bounded by plan limits, add-on picker, promo code entry, and a **live price breakdown from
> `C3`** — base, add-ons, discount and tax as separate lines. Handle a rejected promo code
> (`PromoRefusal`) by explaining which rule refused it. Read
> `al.net.entities/Billing/PromoCode.cs`.

### C6 — Wallet & credits UI

> Build wallet management in `apps/web/src/features/commerce/`. Balances at tenant, company, branch
> and user scope; top-up with a **$50 minimum** (payment fees make smaller top-ups unprofitable —
> see COST.md); burn history with the feature that burned it; low-balance alerting; per-feature
> enablement. Wallets resolve **nearest-first** — show *which* wallet paid for a burn, because "why
> did my branch balance drop" is otherwise unanswerable. Read `al.net.entities/Billing/Wallet.cs`
> and `WalletRef.cs`.

### C7 — Promo codes & discounts

> Build promo code management in `apps/web/src/features/commerce/`. CRUD with benefit types (percent
> off, amount off, bonus wallet units, rate discount), feature targeting, and time windows. Show
> redemptions and refusal reasons. **Discounts round down while overage rounds up** — surface the
> computed value so an author can see it. Read `al.net.entities/Billing/PromoCode.cs`.

### C8 — Payment gateway (backend)

> Build payment gateway integration as a new `al.net.payments` package, wired into `al.subscription`.
> Provider abstraction over the existing `PaymentProvider` entity, resolved per scope through
> `IProviderBindingResolver` like every other provider. Implement charge, refund and webhook receipt.
> Non-negotiable: **verify the webhook signature before parsing the body**, and make charge and
> webhook handling **idempotent by key** — a retried webhook must not double-charge, and gateways
> retry aggressively. Store no raw card data. Add tests for replayed webhook, out-of-order webhook,
> and signature mismatch. Write `al.net.payments/README.md`.

### C9 — Metering & usage capture

> Build usage metering in `al.subscription`. Emit a per-feature usage event on every billable
> operation, and have quota enforcement, wallet burn and invoice line items **all read from that one
> stream**. Three consumers, one source — two sources of usage will disagree, and the disagreement
> surfaces as an invoice a customer can prove is wrong. Include the scope on every event so usage
> rolls up correctly.

### C10 — Invoice generation (backend)

> Build invoice generation in `al.subscription`. Compose line items from the subscription period,
> per-feature usage (`C9`), quota add-ons and one-off purchases; apply promotional discounts and
> wallet credits; compute totals and tax using `C3`'s documented order of operations. Generate the
> PDF via a background job, **not on the request**. Read `al.net.entities/Billing/Invoice.cs`. Pin
> the order of operations with a test using the worked example in `Deliverables/COST.md`.

### C11 — Payment configuration UI

> Build payment configuration in `apps/web/src/features/commerce/`. Choose a gateway per scope,
> supply credentials, toggle test mode, and run a connection check. **Never render a stored secret** —
> show the `@secret:` reference and offer replace-only. Follow the provider-binding UI pattern from
> `D3`. Depends on `C8`.

### C12 — Shared vs dedicated infrastructure

> Surface the tenant infrastructure class in `apps/web/src/features/commerce/` and honour it in
> provisioning. `TenantCatalog` already carries `InfraType`, `ConnectionMode` and `DatabaseConfig`,
> so this is a provisioning decision rather than an architectural one. Read `Deliverables/COST.md` §
> *shared vs dedicated* — a dedicated tenant costs 30–50× the incremental of a shared one, so it must
> be priced as an add-on and **never bundled**, or shared tenants subsidise it.

### C13 — Invoice UI

> Build the invoice screens in `apps/web/src/features/commerce/`. List with status filter, detail
> with line items, download, payment status, and retry for a failed payment. Depends on `C8` and
> `C10`; if either slips, ship the list and detail read-only rather than blocking. Reuse the download
> pattern from `apps/web/src/api/useExport.ts` — fetch the link on click, never cache it.

### D1a — Form builder: field placement

> Build the form builder's placement surface in `apps/web/src/features/form-builder/`. Pick a module
> and resource, then show the available fields: the **static-field catalogue**
> (`GET /api/v1/udf/static-fields/{module}/{resource}`) alongside the tenant's custom definitions.
> Drag to order, set column span, set per-application visibility. **Mandatory columns cannot be
> hidden or made optional** — the resolver refuses, so the UI must explain rather than silently
> disable. System-managed keys are shown as not-placeable. Read `al.net.packages/al.net.udf/README.md`
> first. Write `apps/web/src/features/form-builder/GUIDE.md`.

### D1b — Form builder: labels & i18n

> Add label and translation editing to the form builder. Per-scope `labelOverride`, help text,
> tooltip and placeholder, each with a translation editor. Two rules to get right: overrides are
> **overlaid, not substituted** — supplying only English must keep the tenant's Spanish; and the
> missing-translation indicator must use `HasTranslation`, because `Resolve` always succeeds and can
> never tell you something is untranslated. Show a live preview in the selected language. Worked
> example: one branch shows "First Name", another shows "First", and both render "Nombre" in Spanish.

### D1c — Form builder: sections & conditions

> Add sections and conditions to the form builder. Section CRUD with translated title, order,
> collapse and wizard-step flags; a condition builder producing `UdfCondition` (field, operator,
> values) attachable to a field or a whole section. Render a **live preview using the real
> `DynamicForm`**, not a mock — a preview that diverges from the renderer is worse than none.
> Conditions say when a field is *shown* and the client inverts them for `hidden`; use the existing
> `toHiddenRule`, do not write a second inversion. Worked example: selecting biomethane reveals four
> extra fields, and a section-level condition gates all four at once.

### D2 — List-view columns per scope

> Extend UDF to table views. Backend (`al.net.udf` + `al.udf`): let a `UdfViewConfiguration` target
> the `uvt-table` view type so a column set resolves per scope, and add
> `GET /api/v1/udf/views/{module}/{resource}/table` returning resolved columns in order. Frontend:
> have `DataGrid` accept a resolved column set, and build the configuration UI. Requirement to
> satisfy: **one scope sees 10 columns, another sees 8**, from one configuration. Update
> `packages/react/src/datagrid/README.md` — it currently states the grid does not consume UDF.

### D3 — Provider bindings UI

> Build provider configuration in `apps/web/src/features/configuration/`. A matrix of capability
> (notification, storage, cache, telemetry, queue, audit) × channel × scope, letting an administrator
> bind a provider at any level. Worked requirement: for one tenant's biomethane line, branch A uses
> SMS, B WhatsApp, C Teams, D Email and E all of them. Make the distinction between **disabled** and
> **inherited** visible — a disabled binding *suppresses* the capability and does not fall through to
> the parent, which is the behaviour people misread. **Show the per-message cost at the point of
> choosing** — SMS is ~100× email (see `Deliverables/COST.md`), and a branch routing everything to
> SMS turns a profitable account into a loss-making one. Read
> `al.net.data/Registry/EfProviderBindingResolver.cs`. Write
> `apps/web/src/features/configuration/GUIDE.md`.

### D4 — Templates & assignments by scope

> Build template management in `apps/web/src/features/configuration/`. Import templates (columns,
> commit policy, policy-override flag) and notification templates, each assignable per scope, with a
> matrix showing which template wins where. Import templates already resolve by scope in
> `al.ingestion` — this is the authoring UI over it.

### D5 — Preferences & configuration by scope

> Build the preferences console in `apps/web/src/features/configuration/`. Grouped preferences
> settable at any scope level, with a **resolution preview naming which level supplied the effective
> value**. That provenance is the entire value of the screen: a resolved value with no explanation of
> where it came from is unsupportable, and "why is this setting on" becomes a developer question.

### D6 — Audit, telemetry & queue strategy per scope

> Extend the `D3` provider-binding UI to the remaining capabilities: audit sink, telemetry exporter
> and queue provider, each configurable per scope. Same matrix, same disabled-versus-inherited
> distinction. Note that no `IQueueReceiver` implementation exists in the SDK yet — configure the
> binding, and flag the missing implementation rather than stubbing one.

### R0 — Extract EU biomethane requirements

> Produce `docs/compliance/EU_BIOMETHANE.md` from primary sources. Start from
> `Carbon-atlas-webapp/Deliverables/EU-BIOMETHANE.md`, which lists what is needed and its open
> questions, and from the sources at the foot of that document. Extract, **with a citation for
> each**: GHG-saving thresholds by installation commissioning date and end use (as transposed in
> Spain); Annex IX Part A and Part B categories and their double-counting treatment, with the Spanish
> Part B cap; the UDB transaction schema and submission window; the ETS2 timeline and obligated
> parties; the ESRS E1 datapoints biomethane affects. **State plainly where sources disagree or leave
> something open rather than resolving it** — a flagged question is useful, a guess presented as fact
> is not. Mark the document as requiring compliance-lead sign-off before any rule is configured from
> it.

### R1 — Condition group builder

> Build a reusable nested condition builder in `packages/react/src/rules/`. Arbitrarily nested AND/OR
> groups, each leaf being field + operator + value(s), with add/remove at any depth and a JSON
> preview. Target expression: `field1 = "abc" AND (field2 = "def" OR field3 = "ghi")` — **nesting is
> the requirement**, not an enhancement. The comparison operand must be able to be a **scope-resolved
> value**, not only a literal — see `Deliverables/RULES-AND-WORKFLOWS.md` shape 1, the one people
> hard-code. Fields come from the UDF resolved form for the target resource, so a custom field is
> usable in a rule with no code change. **This component is reused by `W1` for workflow transition
> guards — build it to be reused.** Write `packages/react/src/rules/README.md`.

### R2 — Rule assignment by scope

> Build rule management in `apps/web/src/features/rules/`. Attach rules to a resource and event,
> scoped, with priority ordering, enable/disable, and severity (block / flag / warn). Include a
> **dry-run**: paste or pick a sample payload and show which rules match and what they would do.
> Dry-run is what makes rule authoring safe — without it people test in production. Rules marked
> **not disableable** must be enforced in the resolver, not by greying out a checkbox. Rule changes
> are versioned and audited. Read `al.net.rules` and `al.rules`. Write
> `apps/web/src/features/rules/GUIDE.md`.

### R3 — Rule catalogue as acceptance fixtures

> Take the rule catalogue in `Carbon-atlas-webapp/Deliverables/EU-BIOMETHANE.md` and encode all
> fourteen rules as fixtures for the `R1` condition builder. Each fixture is the rule's condition tree
> as JSON plus payloads that should and should not match. Four are the point of the exercise:
> `R-FEED-02` needs a **nested** boolean over a scope-resolved cap; `R-MB-01` needs an **aggregate
> over a window**; `R-DC-01` must be **non-disableable** — prove the UI cannot switch it off; and
> `R-SLIP-01`/`R-DIG-01` prove the severity split (flag and warn, not block). If the builder cannot
> express any of these, report it as a design defect **now** rather than discovering it during the
> industry build.

### R4 — Rule execution monitor

> Build the rule monitor in `apps/web/src/features/rules/`. What fired, on what record, with what
> outcome and at what severity; filter by rule, scope and period; replay a single evaluation against
> the current rule set. Use `DataGrid`. The replay is what turns "the system rejected my batch" into
> an answerable question.

### E1 — Emission factor service

> Build the emission factor service in `al.carbon`. Third-party lookup (Climatiq / Carbon Interface)
> **behind an interface**, with a local document-store cache as fallback — offline capability and a
> 15× cost reduction on the most-repeated operation in the product (see `Deliverables/COST.md`).
> Lookup keys: geography, sector, activity type, period. Cache entries carry the factor's **version
> and source**, because `E4` snapshots them and a factor revised later must not silently change a
> historical calculation.

### E2 — Scope 1 & 2 calculation

> Implement Scope 1 and Scope 2 calculation in `al.carbon`. Scope 1: fuel combustion, process and
> fugitive emissions. Scope 2: purchased electricity, **location-based and market-based computed and
> stored as two separate values** — never one derived from the other at render time, because both are
> reported figures and a derived one cannot be audited. Apply industry-specific methodology per
> vertical. Read `Deliverables/ARCHITECTURE.md` § 4.4.

### E3 — Scope 3 calculation

> Implement Scope 3 calculation in `al.carbon` across the 15 GHG Protocol categories. Each category
> records its own **method** (spend-based, average-data, supplier-specific) and its own confidence
> score, because they will differ widely within one report and an auditor asks per category. Support
> partial coverage — a customer reporting 6 of 15 categories must be able to say so explicitly rather
> than appearing to report zero for the rest.

### E4 — Audit snapshot

> Add immutable calculation snapshots in `al.carbon`. Every calculation records its activity inputs,
> the emission factors used **with their versions and source**, the methodology, the actor and the
> timestamp. A figure that cannot be reproduced cannot be defended to an auditor, and factors get
> revised. The snapshot is append-only: a recalculation creates a new one rather than mutating the
> old, so "what did we report last year and why" stays answerable.

### W1 — Workflow stage designer

> Build the workflow designer in `apps/web/src/features/workflow/`. Define stages, transitions,
> approval configuration (who approves, by role and scope, and how many), and a visual graph.
> **Reuse the `R1` condition builder for transition guards** — do not write a second one; two
> condition editors will drift and then the same expression means different things in a rule and in a
> workflow. Support a stage owned by an **external party** (a certifier outside the tenant). Read
> `al.net.workflow` and `al.workflow`. Write `apps/web/src/features/workflow/GUIDE.md`.

### W2 — Time-triggered starts & asynchronous failure

> Extend the workflow engine and designer for two patterns it currently cannot express. (1)
> **Time-triggered start** — a workflow that begins because a date arrived, with no user action;
> `W-CERT` starts when a certificate approaches expiry. (2) **Asynchronous stage failure with
> compensation** — a stage completes, then a later external confirmation reverses it; `W-ALLOC` marks
> a volume allocated and the UDB rejects it afterwards, so the workflow must compensate rather than
> simply fail. Read `Carbon-atlas-webapp/Deliverables/RULES-AND-WORKFLOWS.md` for both patterns.

### W3 — Workflow runtime monitor

> Build the workflow monitor in `apps/web/src/features/workflow/`. Running instances with current
> stage, full history, and manual advance or cancel. **Require a reason on any manual
> intervention** — it is the only audit trail an override leaves, and "who pushed this through and
> why" is the first question asked afterwards. Use `DataGrid` for the list.

### W4 — Immutable terminal state

> Add a locked terminal state to the workflow engine. `W-CSRD` ends in **Locked**, and locked means
> locked — no administrator, no support grant and no API call reopens it. A filed regulatory
> disclosure that can be edited afterwards is not a filed disclosure. Reopening requires a new
> workflow instance that supersedes the old one, leaving both in the record. Test that every
> privileged path is refused.

### O1 — Offset measure library

> Build the measure library in `al.offset`. `MeasureDefinition` (vertical, mechanism, description,
> default abatement basis) at platform scope, plus `SiteMeasureAssessment` (abatement, capex, opex,
> complexity, feasibility, incentives) **per site** — a measure's value is a property of *(measure ×
> site)*, not of the measure, and getting that backwards produces a plan confidently wrong everywhere
> except where it was calibrated. Seed the biomethane, heat, CCUS, water and waste libraries from
> `Carbon-atlas-webapp/Deliverables/OFFSET.md`.

### O2 — Offset plan generator

> Build the plan generator in `al.offset`. Rank measures by `(annual abatement × ROI) / complexity`
> and group into quick wins (0–6 months), medium term (6–18) and long term (18–36). **Rule-based, not
> AI** — AI ranking needs outcome data that will not exist until pilots have run. Support a
> **negative abatement**: electrification on a carbon-intensive grid can raise emissions, and the
> engine must return that rather than clamping to zero. Order the plan reduce → remove → credit, and
> keep credits out of the gross figure.

### O3 — Offset plan UI

> Build the offset plan screens in `apps/web/src/features/offset/`. Roadmap view by horizon,
> per-measure detail with the assessment inputs shown, and branch/location overrides. A plan is
> presented to a board and funded against, so **version and date it** and keep what was recommended
> and on what basis — the same reasoning as the calculation audit snapshot. Read
> `Carbon-atlas-webapp/Deliverables/OFFSET.md`.

### B1 — Batch ledger

> Build the batch ledger in `al.carbon`. `Batch` (volume, period, site, status), `Feedstock`
> (category, Annex IX class, origin, quantity), `ChainOfCustody` (mass-balance movements per period
> and site). **A batch, not a meter reading, is the unit of account** — every claim traces to a
> physical volume with a feedstock, an origin, an intensity and a certificate. Read
> `Carbon-atlas-webapp/Deliverables/EU-BIOMETHANE.md` § *Data model summary*.

### B2 — GHG intensity calculator

> Implement the RED III Annex VI intensity chain in `al.carbon`:
> `E = eec + el + ep + etd + eu − esca − eccs − eccr`, and the saving against a fossil comparator.
> **Store each term separately with its own provenance** — value, unit, source (measured / default /
> disaggregated default), evidence reference and actor. An auditor does not accept an aggregate; they
> ask which digester, what methane slip was assumed and where each figure came from. Support a
> negative `eec` from manure credits, which can produce savings above 100%. Read
> `Carbon-atlas-webapp/Deliverables/EU-BIOMETHANE.md` § *The GHG intensity calculation*.

### B3 — Claim register

> Build the claim register in `al.carbon`. One `Claim` record per (volume, register, counterparty),
> where register is UDB, Guarantee of Origin, ETS attribution or corporate disclosure. **A volume may
> be claimed once**, and this is the single point every downstream use consults — exactly as
> `EfQuotaGuard` is the single place quota is enforced. Build it **vertical-agnostic**: a captured
> tonne on a biogas upgrader is both a biomethane term and a CCUS asset, so the register cannot live
> inside one vertical. `R-DC-01` reads from it and must not be disableable.

### B4 — PoS & certificate management

> Build certificate and Proof of Sustainability management in `al.carbon` and
> `apps/web/src/features/biomethane/`. Scheme (ISCC EU / REDcert EU) configured per scope; certificate
> number and validity window; PoS issued against a batch and linked to a certificate; expiry alerting
> that starts the `W-CERT` renewal workflow. **A lapsed certificate blocks allocation** (`R-SUS-03`).

### B5 — Ace vertical slice

> Seed the Ace biomethane slice as an on-demand seeder: tenant `ace`, company `islabhasolutions`,
> branch `spain`, industry `biomethane`, units `digester-1`, `upgrading-1`, `injection-point-1`.
> Include a feedstock mix (manure + food processing residues), **two batches — one comfortably above
> threshold and one marginal** so `R-SUS-01` is visible doing its job rather than passing silently, a
> certificate approaching expiry so `W-CERT` fires, one allocation the UDB stub rejects so
> `W-ALLOC`'s compensation path runs, and an offset plan from the measure library. This is the demo:
> make it real, not decorative.

### B6 — ESRS E1 disclosure builder

> Build the disclosure builder in `al.carbon` and `apps/web/src/features/biomethane/`. **Biomethane is
> a low-carbon fuel reducing gross Scope 1 — never an offset netted against gross.** Store gross and
> market-based as two values, attach the methodology reference and its auditor approval to any
> market-based figure (`R-GHG-01`), and end in a **locked** state (`W4`). Read
> `Carbon-atlas-webapp/Deliverables/EU-BIOMETHANE.md` § *Why this is not a carbon-offset product* —
> the vocabulary matters, because a screen saying "offset" teaches a customer to describe their own
> disclosure incorrectly.

### B7 — UDB submission stub

> Build UDB submission in `al.carbon` behind a provider interface, with a stub implementation. Model
> the transaction shape from raw-material origin to final supplier allocation, a submission queue,
> retry with correction (`W-UDB`), and backlog alerting when the mandated window is at risk
> (`R-UDB-01`). Live integration is deferred (`X1`) — **shape the seam now so it becomes a provider
> swap**, not a rewrite. Reconcile submissions against the `B3` claim register.

### A20 / T10 / C14 / E5 / B8 — End-to-end passes

> Run the week's journey on a **reset database** and fix what breaks. Record every defect in the
> session log; fix P0 only and push P1 to the deferred backlog with an estimate.
>
> - **`A20` Access** — the four Ace users × five scope levels × two applications. Assert each sees
>   exactly what their grant allows and no more, and that a web-only user is refused a mobile token.
>   Build it as a **suite, not a checklist** — it is the regression net for every later access change.
> - **`T10` Provisioning** — tenant → company → branch → industry → unit → site → first admin →
>   upload activity data → validate.
> - **`C14` Commerce** — provision → subscribe → allocate quota → burn a feature → apply a promo →
>   generate an invoice → pay.
> - **`E5` Calculation** — upload → validate → calculate Scope 1/2/3 → snapshot, **including a
>   Spanish-locale run**, which exercises resolution paths a default-locale run never touches.
> - **`B8` Biomethane** — batch → evidence → approval → allocation → claim → disclosure, including
>   the marginal batch that `R-SUS-01` should reject and the allocation the UDB stub rejects.

### A21 / T12 / C15 / E6 — Documentation passes

> Write or update the feature `GUIDE.md` files for the week's work, and reconcile the affected
> document in `Carbon-atlas-webapp/Deliverables/` with what was actually built.
>
> A `GUIDE.md` answers, in order: **what this is for**, **the decisions that shape it and what the
> alternatives cost**, **how to use it**, **the pitfalls**. Not an API dump — the types are the API
> dump. Use `packages/react/src/forms/udf/README.md` as the reference for tone and depth.
>
> - **`A21`** — `features/identity/GUIDE.md`, `features/rbac/GUIDE.md`; reconcile
>   [IDENTITY-AND-SCOPE.md](./IDENTITY-AND-SCOPE.md), especially the *What this replaces* section.
> - **`T12`** — `features/tenancy/GUIDE.md`, `features/form-builder/GUIDE.md`,
>   `features/ingestion/GUIDE.md`.
> - **`C15`** — `features/commerce/GUIDE.md`; reconcile [COST.md](./COST.md) against shipped pricing,
>   and replace any placeholder figure that measurement has now given a real value.
> - **`E6`** — `features/rules/GUIDE.md`, `features/carbon/GUIDE.md`; reconcile
>   [RULES-AND-WORKFLOWS.md](./RULES-AND-WORKFLOWS.md) and fold `R0`'s findings into
>   [EU-BIOMETHANE.md](./EU-BIOMETHANE.md), replacing the ⚠️ working values with sourced ones.

### G1 — Cross-review

> Each developer reviews a track they did not build. Look for the two things self-review misses: a
> rule enforced on the client but not on the server, and a scope or culture parameter missing from a
> query key. Fix P0 defects only; everything else goes to the deferred backlog with an estimate.

### G2 — Documentation sync

> Bring every `GUIDE.md` up to date with what was actually built, update any SDK or service `README`
> touched during the window, reconcile this `Deliverables/` folder — particularly `COST.md` against
> shipped pricing and `EU-BIOMETHANE.md` against `R0`'s findings — and close the session log. The
> `F4` CI check must pass.

### G3 — Code freeze

> Tag the release, write the changelog, produce the **ordered** migration list across all three
> repositories, and write the **rollback plan**. Write the rollback before deploying — nobody writes
> a good one during an incident.

### G4 — Deploy & smoke

> Run migrations in dependency order, run seeders, verify health checks, then run the smoke suite.
> Migration ordering has bitten this project before — follow the local verification notes and do a
> dry run against a reset database first.

### G5 — Handover pack

> Produce the handover: a demo script walking the biomethane slice end to end, known limitations
> stated plainly, the deferred backlog with estimates, and the open decisions still outstanding.

---

## Definition of done

A task is done when **all** of these hold:

1. It builds, type-checks and lints clean.
2. Tests cover the **decisions** — the rules with consequences — not the getters.
3. The feature's `GUIDE.md` exists and reflects what was built.
4. Any SDK or service README touched by the change is updated.
5. The roadmap row is ticked and the session log has an entry.
6. **Nothing is committed.** Changes are left in the working tree for review.

---

## Session log

| Date | Who | What landed | Notes for the next session |
|---|---|---|---|
| 2026-08-15 | Claude | UDF form-metadata system: per-scope label overrides, `LocalizedText` i18n, tooltips, conditions, sections, static-field catalogue. Bulk import/export pages + `BulkUploadGrid`. DataGrid & bulk READMEs. | SDK `.64` published and consumed. |
| 2026-08-15 | Claude | Auth: `POST /auth/challenge` (single-use, 2 min, identifier-bound, rate-limited, no enumeration); four Ace login identities; single re-challenge on `challenge_invalid`; `ApiError.code` + `error`-body normalisation; `useChangePassword`; `features/auth/GUIDE.md`. | Services 346 tests, web 85, react 152 — all green. |
| 2026-08-16 | Claude | Login challenge pre-fetch on identifier blur; resend cooldown; `/auth/tenants` + `/auth/context/switch`; `useSwitchContext` (clears the cache, does not invalidate); `TenantSwitcher`. `Deliverables/` folder created with 8 documents. | **`TenantSwitcher` is not mounted** — `F2` places it in `AppHeader`. |
| 2026-08-16 | Claude | Merged the old `DELIVERABLES.md` into this file — prompts, ground truth, conventions, file changes — and deleted it. Moved `INTEGRATION_STANDARD.md` into this folder. | This file is now the only plan. |
| 2026-08-16 | Claude | **Onboarding closed out.** `POST /master/users/onboard` orchestrates identity → applications → invitation → role across the two services; `features/identity/` gives it a UI with a `GUIDE.md`. Added the missing `user-invitation` **and** `user-verification` templates — both were dispatched by name with nothing behind them. Emails now carry a link from `Auth:WebAppUrl` rather than a raw token. `AssignUserRoleAsync` gained `scopePath`. | **Two bugs found while testing it live:** the repeat call wrote a **duplicate role row** (`AssignUserRoleAsync` added unconditionally — now idempotent, verified 3 calls → 1 row), and it stored `/` where seeded rows store NULL, giving one column two spellings of "no path". SDK 407 / services 359 / web 85 / react 152, both .NET repos warning-clean on `--no-incremental`. |
| 2026-08-16 | Claude | **SDK `.67` verified — all four Ace users now resolve at their own level.** The `ScopePath` fix reached the services, so Amar (industry-level) works. Extended `LandingScope` to derive **scopePath** too, and `TokenResponse` now returns `companyId` / `branchId` / `scopePath`, so a client sends `x-branch-id` and `x-scope-path` from the login response rather than hard-coding its own scope. | Verified live on `.67`: `/auth/me` OK for tenant-, company-, branch- **and** industry-level users, with nothing hard-coded client-side. Services build warning-clean, 359 tests pass. |
| 2026-08-16 | Claude | **Warning regression fixed.** My three new test files introduced **57 xUnit1051 warnings** (22 SDK, 35 services) into repositories that had been warning-clean — every awaited call needed `TestContext.Current.CancellationToken`. Both repos are clean again on a `--no-incremental` build. | Worth knowing: an *incremental* build reports `0 Warning(s)` because nothing recompiles. **Verify with `dotnet build --no-incremental`** before claiming a repo is clean — I reported clean twice on incremental output before this surfaced. |
| 2026-08-16 | Claude | **End-to-end run of the fresh-environment path. Four real bugs found and fixed.** (1) `al.ingestion` missing from the reset script. (2) `al.ingestion` had **never deployed** — index filters written snake_case against PascalCase columns; migration regenerated. (3) `set-tenant-ace.ps1` missed `ace-biomethane` — the seeder key is per *seeder*, not per service, so Amar's industry never existed. (4) Four new endpoints had **no route-permission rules**, so a valid token got 403. Plus: `al.ingestion` secrets held stale `Password=postgres`. | Verified live: all four users sign in on web; Amar **refused on mobile** (`application_not_enabled`) and refused when the client names no application; nonce replay rejected; wrong password 401 (distinct from 403). |
| 2026-08-16 | Claude | **Landing scope implemented** (`ILandingScopeResolver`) — login and context-switch now derive company/branch from the user's assignments and stamp company into the token. Without it, three of four users signed in and were refused **every** endpoint, because a company-scoped assignment does not apply to a request naming no company. **SDK fix:** `ApiContextMiddleware` never populated `ScopePath`, so *any* industry- or unit-level assignment resolved to nothing. | `/auth/me` and `/auth/tenants` now OK for tenant-, company- and branch-level users. **Amar (industry-level) still 403 until the SDK is republished** — the `ScopePath` fix is in `al.net.packages` and builds, SDK 407 tests green, but services consume the published package. **Branch travels as `x-branch-id`**: `IssuePair` has no branch parameter. *(Later that day: reviewed and kept as headers — see the decision in [IDENTITY-AND-SCOPE.md](./IDENTITY-AND-SCOPE.md#why-branch-and-scope-path-travel-as-headers).)* |
| 2026-08-16 | Claude | **Local environment setup split in two.** `reset-dev-databases.ps1` now seeds the platform baseline only and creates no tenant; **`al.ingestion` added — it was missing, so a "full reset" left `ace_ingestion` behind**. New `set-tenant-ace.ps1` runs `seed-tenant ace` across **five** services (the old inline version ran two, so identities, rules and workflows were never seeded). `DEV_GUIDE.md` §10 and both READMEs updated. | Migrations verified fresh: one `InitialCreate` per context, **no model drift** on any of the six checked. Both scripts parse clean and are **pure ASCII** — a non-ASCII `.ps1` without a BOM is read as ANSI by Windows PowerShell and smart punctuation decodes into characters the parser treats as quotes. |
| 2026-08-16 | Claude | **`A7` application gating done.** `ApplicationAccessService` + refusal at `/auth/login` **and** `/auth/context/switch` with `application_not_enabled`; `VITE_APPLICATION_ID` sent by the client; Ace web/mobile applications seeded; **Amar seeded web-only** so the control is demonstrated by a real account. `TenantSwitcher` mounted in `AppHeader` and rebuilt as a **two-level cascade** (tenant → company) with last-scope persistence. New `GET /api/v1/master/me/scopes`. New `POST /auth/users/onboard` — identity + tenant access + applications + invitation, idempotent on email. | Services **359** tests (+13), web 85, all builds clean. **Not done:** `A6` grant-table consolidation (`TenantAccess` + `TenantUserRole` still separate — application now lives on `TenantAccess`, which was the blocking gap); the onboarding **UI** and al.master's orchestration of role assignment around `/users/onboard`; invitation email template `user-invitation`. |
| 2026-08-18 | Claude | `F1` design tokens & theme done — full token layer, `ThemeProvider`, README. Real bug found: Tailwind tree-shook an unused token; fixed with `@theme static`. | Verified live by toggling theme and reading computed styles. |
| 2026-08-19 | Claude | F1 re-verified against actual code during a broader dark-mode audit: token layer complete; density/motion/commerce tokens defined but unconsumed anywhere — F2/F7's job. | 255 tests green, no new errors. |
| 2026-08-19 | Claude | Wired F1's motion tokens where they exactly matched existing hardcoded values (`duration-200`/`300`, 38 of 49 occurrences, 20 files) so `prefers-reduced-motion` now has a real effect. Left the rest (no clean match) and Framer Motion (needs new JS) untouched. | Verified via computed style, not just the diff. 255 tests green. |
| 2026-08-19 | Claude | Wired F1's density tokens where they exactly matched (shared `Button` default + shared text input, both 36px = `--control-height-md`). Everything else measured close-but-not-exact — left alone, documented in root `CLAUDE.md` for F2/F7. | Verified via pixel measurement before/after — unchanged. 255 tests green. |

---

## Pick up here

The authentication track is **complete and verified live**. Nothing is outstanding from it — start on
the W1 roadmap above.

**Decided, do not "fix":** `branchId` and `scopePath` travel as `x-branch-id` / `x-scope-path`
headers rather than token claims. They drive both query filtering and write stamping, but they are
also what permission resolution compares against — so naming a scope you hold no assignment for
resolves **zero permissions** rather than granting anything. A header selects among the assignments
somebody already holds; it cannot fabricate one. Full reasoning, and the two caveats it does not
solve, in [IDENTITY-AND-SCOPE.md](./IDENTITY-AND-SCOPE.md#why-branch-and-scope-path-travel-as-headers).

**Done and verified on `.67`:**

- `POST /api/v1/master/users/onboard` — one call across two services: identity, application access
  and invitation from al.auth, then the role assignment written by al.master. Verified live end to
  end, including idempotency (three identical calls → **one** role row).
- `user-invitation` and `user-verification` templates now exist. Both were being dispatched by name
  with nothing behind them, failing silently into a log line. Emails now carry a **link** built from
  `Auth:WebAppUrl`, not a bare token.
- Onboarding UI: `features/identity/` with `OnboardUserForm`, `useOnboardUser` and a `GUIDE.md`.
- `AssignUserRoleAsync` gained `scopePath` (industry/unit assignment) and is now **idempotent** —
  it previously wrote a duplicate row on every repeat.

**Environment, if starting cold:**

```powershell
.\scripts\set-dev-secrets.ps1                 # once per machine
.\scripts\reset-dev-databases.ps1 -Deploy -Force
.\scripts\set-tenant-ace.ps1 -Force           # optional demo tenant
```

Then start `al.udf` once. Sign in as any of the four `@ace.local` users; the password is
`Secrets:ace-demo-password` on **al.auth**, or generated and logged on first seed. The seeder hashes
**on create only**, so changing that secret afterwards does nothing until `ace_auth` is recreated.

---

## Open decisions

Things the team cannot resolve alone. Each blocks something dated.

| # | Decision | Needed by | Owner |
|---|---|---|---|
| 1 | **Margin target** — 25%, 35% or 45%. Drives every published price in [COST.md](./COST.md). | 7 Sep (`C3`) | Business |
| 2 | **Payment gateway** — Stripe, Adyen or a regional provider. Shapes `C8`. | 7 Sep | Business |
| 3 | **Certification scheme** — ISCC EU, REDcert EU or other, for the Ace slice. | 21 Sep (`B4`) | Compliance |
| 4 | **Data residency** — EU-only for all tenants, or per tenant. ±30% on infrastructure cost. | 10 Sep (`C12`) | Business |
| 5 | **Shared vs dedicated threshold** — which plan tier earns dedicated infrastructure. | 10 Sep (`C12`) | Business |
| 6 | **Compliance sign-off on thresholds** — nothing in [EU-BIOMETHANE.md](./EU-BIOMETHANE.md) becomes a configured rule without it. | 14 Sep (`R0`) | Compliance |
| 7 | **FREE tier at all?** — costs ~$8–25/month per tenant with no revenue. A funnel, not charity: cap it hard or drop it. | 7 Sep (`C1`) | Business |
