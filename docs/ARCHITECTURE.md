# Architecture

The services, the data model, the isolation boundaries — and an honest account of what already
exists versus what is still to build.

← [Deliverables.md](./Deliverables.md) · see also [IDENTITY-AND-SCOPE.md](./IDENTITY-AND-SCOPE.md)

---

## Shape

Three repositories, twelve services, one SDK.

```
al.net.packages        23 SDK packages, published as 0.1.0-integration.N, consumed by every service
Carbon-Atlas-Services  12 microservices, ports 7001–7012
Carbon-atlas-webapp    pnpm monorepo — apps/web + packages/{react,api-client}
```

**The SDK is the reason twelve services do not diverge.** Scope resolution, multi-tenant filters,
caching, quota enforcement, provider bindings and UDF all live in `al.net.packages`, so a rule about
how tenancy works is implemented once. The recurring failure mode in a system this shape is a second
implementation of a shared rule inside one service — which is why several documents here repeat that
"a second implementation eventually disagrees with the first".

---

## Services

| Service | Port | Owns | State |
|---|---|---|---|
| `al.auth` | 7001 | Identity, sessions, tokens, MFA, challenge/nonce | ✅ built |
| `al.master` | 7002 | Tenants, companies, branches, industries, RBAC, provider bindings, permissions | ✅ built |
| `al.subscription` | 7003 | Plans, subscriptions, quota, wallet, promo, invoices | ✅ built · payment gateway & invoice generation ❌ |
| `al.carbon` | 7004 | Emission calculations | ◐ partial — engine is `E1`–`E4` |
| `al.offset` | 7005 | Offset planning | ◐ partial — engine is `O1`–`O3` |
| `al.ingestion` | 7006 | Bulk import, export, staging, validation | ✅ built |
| `al.storage` | 7007 | Blob abstraction, presigned URLs | ✅ built |
| `al.notification` | 7008 | Multi-channel dispatch via provider bindings | ✅ built |
| `al.rules` | 7009 | Rule evaluation | ✅ engine · authoring UI is `R1`–`R4` |
| `al.workflow` | 7010 | Stage execution, approvals, saga | ✅ engine · designer is `W1`–`W4` |
| `al.telemetry` | 7011 | Traces, metrics, diagnostics intake | ✅ built |
| `al.udf` | 7012 | Form metadata, sections, conditions, i18n, static-field catalogue | ✅ built |

**Read this table before starting any task.** The most expensive mistake available in this codebase
is rebuilding something that already works — the rules and workflow engines in particular exist and
are tested; what is missing is the authoring surface.

---

## Data model & storage strategy

Two stores, chosen per data shape rather than per service.

### Relational (PostgreSQL)

**For:** anything needing ACID guarantees, joins, referential integrity or an audit trail.

Tenants and their hierarchy · companies, branches, industries, units · grants and roles ·
subscriptions, quota, wallets, invoices · emission calculations and audit snapshots · offset plans ·
import jobs and staged rows · compliance records.

Isolation is `MultiTenantDbContext` global query filters keyed on the active context, with
`ScopeFilterOverride.SuppressTenantIsolation(reason)` as the deliberate, reasoned escape hatch — a
named method rather than a flag, so bypassing isolation appears in code review as a sentence
explaining itself.

### Document (Cosmos DB)

**For:** flexible schemas, high-read reference data, per-tenant configuration.

UDF definitions, view configurations, sections, static-field catalogue · emission factor database ·
activity data with variable schemas · calculation metadata.

Isolation is the **partition key**, and in a document store the partition key *is* the isolation
boundary. That is why no API accepts a partition or tenant from a caller — a caller-supplied
partition is a caller-supplied tenant. This API shipped once with that defect and had it removed.

### The rule for choosing

> If losing it or getting it slightly wrong would be a financial, legal or audit problem — relational.
> If its shape varies per tenant and it is read far more than written — document.

An emission *factor* is document (varies, read constantly). An emission *calculation* is relational
(audited, must reconcile). Both are used in the same request.

---

## Core components

### 4.1 Onboarding

Multi-step wizard: tenant → company → branch → industry → unit, plus users and their grants.

**UDF-driven, not hand-coded** — *target state for `T3`, not what exists today.* The form will come
from `GET /udf/forms/master/Company`, so the resource's own columns and the tenant's custom fields
render together, in the order somebody arranged them, in the requested language. Selecting an industry
reveals industry-specific fields through server-side conditions.

The backend half is done and verified end to end: `company_catalog` and `branch_catalog` carry a
`custom_fields` jsonb column with a GIN index, onboarding validates submitted values against the
definitions in force before writing, and a save interceptor refuses anything unvalidated.

What is still missing is the **screen**. `CompanyOnboardingPage.tsx` hand-codes the form from
`@/mocks/mockForms` and a local Zod schema, and no React screen consumes a resolved form. That is the
shortcut this section warns about, already taken — it stays flagged until `T3` closes it.

Hand-coding the static half defeats the static-field catalogue entirely — and it is the tempting
shortcut, because the first version is quicker. See
[CONFIGURABILITY.md](./CONFIGURABILITY.md) for which fields belong in the catalogue in the first place.

### 4.2 Data collection & upload

Manual entry for Scope 1/2/3 activity data (`T7`); CSV and Excel upload with presigned direct-to-blob
transfer; staged rows reviewed and corrected before commit. All built — see `al.ingestion` and the
bulk grid.

Cloud events trigger asynchronous parsing; nothing reaches a real record until commit.

### 4.3 Transformation & validation (`T8`)

Five layers, each with its own error surface so a user is told which one rejected them:

```
1  Format        file type, encoding, size
2  Schema        required columns present
3  Type          numerics parse, dates valid
4  Business      values within configured ranges     ← delegates to al.rules
5  Completeness  critical fields populated
```

**Layer 4 delegates to the rules engine.** Building a second rule evaluator inside ingestion is how
an import accepts a row the API would reject — the two disagree, and the disagreement is discovered
after the data is in.

### 4.4 Emission calculation engine — the critical component

Hybrid by design:

| | |
|---|---|
| **Factors** | Third-party API (Climatiq / Carbon Interface) for verified global factors, **behind an interface** |
| **Cache** | Local document-store cache — offline capability and a 15× cost reduction ([COST.md](./COST.md)) |
| **Logic** | Custom .NET, with industry-specific methodology per vertical |
| **Templates** | Pre-configured per sector |

```
Activity data → factor lookup (geography, sector, period)
              → Scope 1 (fuel, process, fugitive)
              → Scope 2 (location-based AND market-based, stored separately)
              → Scope 3 (15 GHG Protocol categories)
              → industry-specific adjustment
              → confidence score
              → immutable audit snapshot
```

Two non-negotiables:

**Location-based and market-based are two stored figures** (`E2`), never one derived from the other.
Both are reported; deriving one at render time makes it unauditable.

**Every calculation snapshots its inputs** (`E4`) — activity values, factor identities and versions,
methodology, actor, timestamp. A figure that cannot be reproduced cannot be defended, and factors
get revised.

### 4.5 Offset planning engine

Specified in **[OFFSET.md](./OFFSET.md)**. Rule-based prioritisation by
`(abatement × ROI) / complexity`, grouped into quick wins, medium and long term, with per-site
assessment because a measure's value is a property of *(measure × location)*.

---

## Multi-tenancy

Fully specified in **[IDENTITY-AND-SCOPE.md](./IDENTITY-AND-SCOPE.md)**. In summary:

```
PLATFORM → TENANT → COMPANY → BRANCH → INDUSTRY → UNIT
                              ×
                   APPLICATION (web | mobile | api)
```

Application is **orthogonal**, not a level — it breaks ties at the same organisational level rather
than sitting inside the hierarchy.

Tenants nest one level: parent, sub-tenant, standalone. A parent admin reads aggregated sub-tenant
data; a sub-tenant sees neither the parent nor its siblings. **All three directions need testing, and
the sibling direction is the one that ships broken.**

---

## Security

| | |
|---|---|
| **Authentication** | ECDSA-signed JWT, refresh rotation, challenge/nonce before password login, OIDC ready |
| **MFA** | Mandatory for admin roles (`A16`), optional otherwise |
| **Authorization** | Database-driven route permissions — policy changes without a deployment |
| **Encryption at rest** | Database TDE, document-store encryption, blob encryption; `[Encrypted]` on PII columns |
| **In transit** | TLS 1.2+ everywhere |
| **Secrets** | `@secret:` references resolved at startup. No committed credentials; never rendered back to a UI |
| **Isolation** | Global query filters (relational) + partition key (document) |
| **Audit** | Immutable log, 7-year retention, correlation id on every request |
| **GDPR** | EU residency option, right-to-erasure workflow, PII redaction in diagnostics (`F3`) |

**The diagnostic-report feature is a security surface.** It ships browser state to the server, so
redaction is recursive and tested — a diagnostic carrying a token or an email is a breach, not a
feature.

---

## Frontend

```
apps/web              React 19, Vite, TanStack Query, react-hook-form, Zod
packages/react        shared components — forms, datagrid, bulk grid, UDF adapters
packages/api-client   HTTP client, service resolution, reactive 401 refresh
```

Three rules that keep it coherent:

1. **The server resolves; the client renders.** No precedence logic in the browser.
2. **Every response-changing parameter is in the query key** — scope and culture included.
3. **Never send tenant, company, branch or application.** They come from the token.

---

## DevOps

| | |
|---|---|
| **Branching** | `main` → `stage` → `integration`, SDK published per branch as `0.1.0-{channel}.N` |
| **Quality gates** | Static analysis, >80% coverage target, dependency and secret scanning |
| **Tests** | Unit, integration, API contract |
| **Deploy** | DEV auto · STAGING manual approval · PRODUCTION blue-green |
| **Migrations** | EF Core, ordered, with rollback |
| **Observability** | Distributed tracing over W3C `traceparent` — one trace spans every service a request touches |

**Targets:** API < 500 ms p95 · page load < 2 s · uptime 99.5% (higher only for ENTERPRISE — see
[COST.md](./COST.md) on SLA economics).

---

## Risks

| Risk | Impact | Mitigation |
|---|---|---|
| Calculation complexity | **High** — timeline | Domain-expert review weekly; start with Scope 1/2; third-party fallback |
| Poor customer data quality | **High** — wrong outputs | Five-layer validation, confidence scoring, pilot assistance |
| `UNIT` scope introduced piecemeal | **High** — two resolution orders | `T1` does every resolver in one change |
| Payment gateway slips | **Medium** — W6 delivery | Invoice screens ship read-only rather than blocking |
| Scope creep | **Medium** | Locked list, deferred backlog with estimates, weekly delivery surfaces drift on day four |
| Third-party factor API dependency | **Medium** — lock-in | Behind an interface, local cache, contractual SLA |
| Regulatory misinterpretation | **High** — customer liability | `R0` + compliance sign-off; nothing configured from an unverified threshold |
| Cross-vertical double counting | **High** — audit failure | One vertical-agnostic claim register from the start |

---

## Delivery phases

Mapped to the weekly plan in [Deliverables.md](./Deliverables.md#roadmap).

| Phase | Weeks | Outcome |
|---|---|---|
| Foundation & identity | W1–W2 | Sign in; access differs correctly by scope and application |
| Tenancy & ingestion | W3 | Provision an organisation; upload and validate data |
| Commerce | W4 | Subscribe, meter, invoice |
| Configuration & calculation | W5 | Author rules; calculate emissions with audit trail |
| Workflow & vertical | W6 | Approvals, billing, biomethane slice for Ace |
| Release | W7 | Frozen, deployed, documented |
