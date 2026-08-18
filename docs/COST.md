# Cost, Pricing & Subscription Design

What it costs to run a tenant, and how that becomes a price a customer pays.

← [Deliverables.md](./Deliverables.md) · see also [ARCHITECTURE.md](./ARCHITECTURE.md)

---

> ## All figures are placeholders with a documented basis
>
> Every number below is an **estimate to be replaced with measured data**, and each says what it is
> derived from so it can be checked rather than trusted. They exist so the pricing *engine* (`C3`)
> can be built against realistic shapes.
>
> **Cloud list prices change and vary by region.** Anything published to a customer must be
> recalculated against current rates for the actual deployment region — see
> [Data residency](#data-residency-the-compliance-tax).

---

## The three-layer model

Carbon Atlas has a cost structure that a single per-seat price cannot cover, because three
different things drive spend.

```
Layer 1   Fixed platform      shared infrastructure, amortised over all tenants
Layer 2   Tenant allocation   what one tenant occupies whether or not they use it
Layer 3   Consumption         what they actually do — storage, notifications, AI, reports
```

A per-seat price only works when cost tracks headcount. Here it does not: a tenant with five users
and a million telemetry rows costs more than one with five hundred users and monthly spreadsheet
uploads. So the model is **platform fee + included allowances + metered consumption**, with the
existing quota and wallet machinery doing the metering.

**Why this shape and not pure usage-based:** pure usage means a customer cannot budget, and a
finance director who cannot forecast a line item does not sign. The platform fee buys predictability;
the allowance covers normal use; metering catches the outliers who genuinely cost more.

---

## Layer 1 — Fixed platform costs (monthly)

The burn before a single tenant does anything.

| Component | Purpose | Est. base / month |
|---|---|---|
| PostgreSQL (managed, HA) | Relational: tenants, grants, calculations, invoices, audit | ~$400 |
| Cosmos DB / document store | UDF configuration, emission factors, activity documents | ~$250 |
| Application hosting (12 services) | Containers, autoscaling, load balancing | ~$450 |
| Redis / cache | Sessions, coherent cache, rate limiting, nonces | ~$120 |
| Blob storage (hot tier, platform) | Templates, exports, system artefacts | ~$50 |
| Observability | Traces, metrics, logs — 90-day retention | ~$180 |
| Key management / secrets | | ~$30 |
| CI/CD & registries | | ~$70 |
| **Total shared baseline** | **Minimum monthly burn** | **~$1,550** |

**What this means commercially.** At a $199/month platform fee, **8 tenants** cover the baseline
before any consumption revenue. At $499, three do. That ratio is the single most useful number in
this document for setting the entry price.

Observability is the line people cut first and regret: 90-day trace retention is what makes a
customer incident answerable. Cutting it saves ~$180 and costs a week of engineering the first time
something goes wrong in production.

---

## Layer 2 — Tenant allocation: shared vs dedicated

Two infrastructure classes, and the boundary is a commercial decision (open decision #5).

| | **Shared** | **Dedicated** |
|---|---|---|
| Database | Shared cluster, row-level isolation by tenant | Own database or cluster |
| Document store | Shared account, partition per tenant | Own account |
| Compute | Shared service pool | Dedicated pool or namespace |
| Blob | Shared account, container per tenant | Own account, own keys |
| **Incremental cost / month** | **~$8–25** | **~$450–900** |
| Noisy-neighbour risk | Present; mitigated by quotas and rate limits | None |
| Data residency control | Region-level | Full |
| Suits | SMB, mid-market, trial | Enterprise, regulated, on-premise-adjacent |

**The platform already supports this.** `TenantCatalog` carries `InfraType` (Shared / Dedicated),
`ConnectionMode` and an optional `DatabaseConfig` — so this is a provisioning decision, not an
architectural one. `C12` surfaces it.

**Price dedicated at cost + margin, never bundled.** A dedicated tenant is 30–50× the incremental
cost of a shared one. Folding that into a plan price means shared tenants subsidise dedicated ones,
and the enterprise deal that looked profitable is the one destroying the margin.

---

## Layer 3 — Consumption factors

The metered half. Each maps to a burn against quota or wallet.

### Storage & retention (per GB / month)

| Retention | Base (S3/Blob standard) | +10% op. | 25% margin | 35% margin | 45% margin |
|---|---|---|---|---|---|
| Hot, 12 months | $0.023 | $0.025 | $0.034 | $0.039 | $0.046 |
| Cool, 12–36 months | $0.010 | $0.011 | $0.015 | $0.017 | $0.020 |
| Archive, 7 years (audit) | $0.004 | $0.004 | $0.006 | $0.007 | $0.008 |

**Audit records must be kept for seven years** and are the volume that quietly dominates. Tiering
them to archive is the difference between $0.046 and $0.008 per GB-month — a 5.75× saving on the
largest and least-read dataset. Build lifecycle rules at provisioning, not later.

### Notifications (per message)

Third-party pass-through, and the place a "free" feature becomes expensive.

| Channel | Base cost | +10% | 25% | 35% | 45% |
|---|---|---|---|---|---|
| Email (transactional) | $0.0004 | $0.0004 | $0.0006 | $0.0007 | $0.0008 |
| SMS (EU average ⚠️ varies 5× by country) | $0.0450 | $0.0495 | $0.0660 | $0.0762 | $0.0900 |
| WhatsApp Business (utility) | $0.0350 | $0.0385 | $0.0513 | $0.0592 | $0.0700 |
| Teams / webhook | $0.0000 | $0.0000 | — | — | — |
| Push (mobile) | $0.0000 | $0.0000 | — | — | — |

> **SMS is ~100× the cost of email and varies enormously by destination country.** A tenant that
> routes every workflow notification to SMS at branch level can turn a profitable account into a
> loss-making one without anyone noticing until the monthly bill. Three defences, all needed:
> **(1)** SMS and WhatsApp burn wallet credits, email does not; **(2)** the provider-binding UI
> shows the per-channel cost at the point of choosing; **(3)** a per-scope monthly cap with an alert.
>
> This is exactly what the DB-driven provider bindings were built for — branch A on SMS, branch B on
> WhatsApp, branch C on Teams. The cost consequence has to be visible where the choice is made.

### Compute & AI (per operation)

| Operation | Base | +10% | 25% | 35% | 45% |
|---|---|---|---|---|---|
| Emission calculation (cached factors) | $0.0002 | $0.0002 | $0.0003 | $0.0003 | $0.0004 |
| Emission calculation (third-party factor lookup) | $0.0030 | $0.0033 | $0.0044 | $0.0051 | $0.0060 |
| Bulk import, per 1,000 rows | $0.0150 | $0.0165 | $0.0220 | $0.0254 | $0.0300 |
| Export / report generation (standard) | $0.0100 | $0.0110 | $0.0147 | $0.0169 | $0.0200 |
| PDF disclosure pack | $0.0500 | $0.0550 | $0.0733 | $0.0846 | $0.1000 |
| Rule evaluation, per 1,000 | $0.0010 | $0.0011 | $0.0015 | $0.0017 | $0.0020 |
| Workflow instance | $0.0020 | $0.0022 | $0.0029 | $0.0034 | $0.0040 |
| AI offset-plan generation (LLM) | $0.1500 | $0.1650 | $0.2200 | $0.2538 | $0.3000 |
| AI anomaly detection, per 1,000 records | $0.0800 | $0.0880 | $0.1173 | $0.1354 | $0.1600 |

**Cache the emission factors.** $0.0030 → $0.0002 is a 15× difference on the most-repeated operation
in the product. `E1` specifies the local cache with third-party fallback; the cost table is why it is
not optional.

### Third-party pass-through

| Service | Model | Note |
|---|---|---|
| Emission factor API (Climatiq / Carbon Interface) | Per lookup or tier | Cache aggressively |
| UDB integration | Likely free (regulatory) ⚠️ | Confirm in `R0` |
| Payment gateway | ~2.9% + $0.30 | **Never absorb on small transactions** — see below |
| Certification scheme portal | Per certificate | Varies by scheme |
| Identity verification (KYC, enterprise) | ~$0.50–2.00 per check | Add-on only |

> **The payment-fee trap.** At 2.9% + $0.30, a $10 transaction costs $0.59 — **5.9%**. Wallet
> top-ups must have a minimum (~$50) or the smallest top-ups are the least profitable. This is the
> same reason the attached proctoring analysis warns against fractional margins in B2C.

---

## Volume scaling

Cloud costs fall with volume; prices should follow, on a published ladder rather than ad hoc
discounting.

| Tenant scale | Cost multiplier | Pricing posture |
|---|---|---|
| < 50 users / < 100 GB | 1.0× | Full retail — 45% margin |
| 50–500 users / 100 GB–1 TB | 0.9× | 35% margin tier |
| 500+ users / 1 TB+ | 0.75× | 25% margin tier (enterprise) |

Publishing the ladder means a salesperson does not invent a discount, and a customer who grows knows
what happens next.

---

## Subscription plans

Mapped onto the machinery that already exists — `SubscriptionPlan`, `TenantQuota`, `Wallet`,
`PromoCode`.

| | **FREE** | **TRIAL** | **PRO** | **ENTERPRISE** |
|---|---|---|---|---|
| Platform fee / month | $0 | $0 (30 days) | **$499** | From **$2,500** |
| Users | 3 | 10 | 50 | Unlimited |
| Companies / branches | 1 / 1 | 1 / 3 | 5 / 25 | Unlimited |
| Storage included | 1 GB | 10 GB | 250 GB | 2 TB |
| Calculations / month | 100 | 1,000 | 50,000 | Unlimited |
| Wallet credits / month | 0 | 500 (no rollover) | 2,000 (no rollover) | Negotiated |
| Infrastructure | Shared | Shared | Shared | **Dedicated available** |
| Rules & workflows | — | Read-only | Full | Full + custom shapes |
| Verticals | 1 | 1 | 3 | All |
| Support | Community | Email | Business hours | SLA + named contact |
| Data residency | Default region | Default | Choice of region | **Any, incl. dedicated** |

**Wallet credits do not roll over** on FREE, TRIAL and PRO — `PeriodicWalletUnitsRollOver = false`
means *top up to*, not *add on top*. Label it that way in `C1` or it will be configured backwards
and either give away credits or withhold them every period.

### Add-ons

| Add-on | Price | Basis |
|---|---|---|
| Extra storage | $0.05 / GB / month | ~2× cost at 45% margin plus lifecycle overhead |
| Extra users | $15 / user / month | Support-driven, not infrastructure-driven |
| Additional vertical | $250 / month | Measure library, rules, workflows, factors |
| Dedicated infrastructure | From $1,200 / month | Cost + margin; **never bundled** |
| Advanced analytics / BI embed | $200 / month | |
| Human-verified audit support | $500 / month | People, not compute |
| Custom rule shape development | Quote | Platform change — see [RULES-AND-WORKFLOWS.md](./RULES-AND-WORKFLOWS.md) |
| Extended audit retention (10 yr) | $0.01 / GB / month | Archive tier |

### Credit packs (wallet top-up)

For metered consumption beyond the allowance. 1 credit = $0.01 of metered spend at list.

| Pack | Credits | Price | Effective |
|---|---|---|---|
| Starter | 5,000 | $60 | $0.012 / credit |
| Growth | 25,000 | $275 | $0.011 / credit |
| Scale | 100,000 | $1,000 | $0.010 / credit |

Minimum top-up **$50** — the payment-fee trap above.

---

## How a price is actually computed

`C3` implements this server-side. **The client never computes a total** — a client figure that
disagrees with the invoice is a chargeback and a support case.

```
Monthly charge
  = platform fee (plan)
  + Σ add-ons
  + Σ over-allowance consumption × unit price × volume multiplier
  − promotional discount        (rounds DOWN)
  − wallet credits applied
  + tax
```

Two rounding rules pull in opposite directions, and both are deliberate:

| | |
|---|---|
| **Discounts round down** | In the customer's favour. A rounding dispute over a discount is not worth the support cost. |
| **Overage rounds up** | In the platform's favour, and it must — a sub-cent rate that rounds to zero makes a paid feature free. `C2` validates against exactly this. |

**Fix and document the order of operations.** Discount-before-credit and credit-before-discount give
different totals. `C10` pins the chosen order with a test against a worked example, because the day
it changes silently is the day a month of invoices is wrong.

---

## Data residency — the compliance tax

EU-only or GovCloud regions run **20–30% higher** than the cheapest region for compute and storage.

**Do not absorb it.** A "Data Localisation Premium" of +30% on the platform fee and consumption
rates maintains the margin. Customers who mandate residency understand that it costs more; customers
who do not should not pay for it.

Relevant here because the Ace tenant is EU-based and biomethane certification data is subject to EU
residency expectations — so the *default* deployment for this vertical is the more expensive one.
Open decision #4.

---

## Worked example

**Mid-size biomethane operator.** One company, three branches, 12 users, 40 GB, 5,000 calculations,
2,000 emails, 300 SMS, 20 disclosure packs, one extra vertical. PRO, shared, EU region.

| Line | Basis | Amount |
|---|---|---|
| PRO platform fee | | $499.00 |
| Additional vertical | 1 × $250 | $250.00 |
| Storage | 40 GB, within 250 GB allowance | $0.00 |
| Calculations | 5,000, within 50,000 allowance | $0.00 |
| Email | 2,000 × $0.0008 | $1.60 |
| SMS | 300 × $0.0900 | $27.00 |
| Disclosure packs | 20 × $0.1000 | $2.00 |
| *Subtotal* | | *$779.60* |
| Wallet credits applied | 2,000 included = $20.00 | −$20.00 |
| EU residency premium | +30% on platform + vertical | $224.70 |
| **Monthly total** | *(before tax)* | **$984.30** |

Estimated cost to serve: shared allocation ~$20, storage ~$1, notifications ~$16, compute ~$5,
plus baseline share ~$65 ≈ **$107**. Gross margin ≈ **89%**.

That is healthy and it is *not* the number to plan on. It excludes support, sales, onboarding and
customer success — the costs that actually scale with customer count. The margin exists to cover
them; see below.

---

## What the margins have to cover

| Overhead | Allocation | Model |
|---|---|---|
| Engineering & maintenance | Across all tenants | Covered by the 25–45% margin |
| Sales & marketing | Across all tenants | Covered by margin |
| Support (tiered) | Per plan | FREE community · PRO business hours · ENTERPRISE SLA |
| Onboarding & data migration | Per customer | **One-off fee, $2,500–15,000. Do not subsidise from subscription** |
| Compliance & domain expertise | Across all tenants | The differentiator; fund it explicitly |
| Dedicated customer team | Per customer | **$10,000–25,000 / month retainer** |

**Never offer a 99.9%+ SLA below ENTERPRISE.** An SLA credit on a $499 account can exceed the
account's annual margin, and multi-region failover to support it costs more than the tier earns.

---

## On-premise / self-hosted

Governments and large industrials mandate it. The economics invert: the customer absorbs
infrastructure, so the price is licence-based.

| | |
|---|---|
| Implementation | $25,000–75,000 — deployment, integration, training |
| Annual licence | $100,000–500,000+ by user volume and verticals |
| Support & maintenance | 20% of licence per year |
| **Caveat** | No managed emission-factor API. Ship a versioned factor database and an update mechanism — and price the updates |

---

## Open decisions

Blocking, and dated in [Deliverables.md](./Deliverables.md#open-decisions).

1. **Margin target** — 25 / 35 / 45%. Every published price above assumes 45% unless stated. *(7 Sep)*
2. **Payment gateway** — shapes `C8` and the fee model. *(7 Sep)*
3. **Shared-vs-dedicated threshold** — which tier earns dedicated. *(10 Sep)*
4. **Data residency default** — EU-only for all, or per tenant. ±30%. *(10 Sep)*
5. **FREE tier at all?** — it costs ~$8–25/month per tenant with no revenue. Justifiable as a funnel,
   not as charity. Cap it hard or drop it.
