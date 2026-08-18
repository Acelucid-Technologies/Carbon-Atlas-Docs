# EU Biomethane — Domain, Calculations, Rules & Workflows

The domain knowledge behind the biomethane vertical, written so that a business analyst can design
features from it and a developer can implement rules against it.

← [Deliverables.md](./Deliverables.md) · see also [RULES-AND-WORKFLOWS.md](./RULES-AND-WORKFLOWS.md) · [OFFSET.md](./OFFSET.md)

---

> ## Read this before configuring anything
>
> **Nothing in this document is legal advice, and no number here becomes a configured rule without
> compliance-lead sign-off.** Values marked ⚠️ are the working set to be confirmed by task `R0`
> against primary sources. They are recorded here so the system can be *designed* against realistic
> shapes, not so they can be shipped unchecked.
>
> A wrong threshold configured as a rule is worse than no rule at all: it passes review because it
> looks authoritative, and it produces certificates that fail an audit months later.

---

## Why this is not a carbon-offset product

The single most consequential thing to get right, and it is a modelling decision, not a labelling
one.

Under **CSRD / ESRS E1**, biomethane is a **low-carbon fuel** that reduces **gross Scope 1**
emissions. It is **not** a carbon offset netted against a gross figure. Removals and purchased
credits are disclosed *separately* and never subtracted from gross emissions.

The consequence for the data model:

> **Gross and market-based figures are two stored values, not one derived from the other.**

A market-based figure computed at render time from a gross figure cannot be audited — the inputs,
the factor versions and the methodology at the moment of calculation are gone. This is an audited
regulatory disclosure, and "we can recompute it" is not the same as "here is what we reported and
why". Task `E2` stores both; task `E4` snapshots the inputs.

The vocabulary matters too. A screen that says "offset" where the standard says "low-carbon fuel"
teaches a customer to describe their own disclosure incorrectly.

---

## The frameworks, and what each demands

| Framework | Governs | What the system must do |
|---|---|---|
| **RED II / RED III** (2018/2001, 2023/2413) | Sustainability & GHG-saving criteria for renewable fuels | Track feedstock category and origin; compute GHG intensity along the chain; enforce the threshold for the installation's commissioning date and end use; hold the Proof of Sustainability evidencing it |
| **Union Database (UDB)** | Mandatory EU platform, centralised | Log **every transaction from raw-material origin to final supplier allocation**. Anti-double-counting. This is an *integration*, not a report — batches must reconcile with UDB records or the claim is unsupported |
| **Mass balance** | How sustainable and non-sustainable volumes may mix | A batch ledger balancing inputs and outputs over a defined period and site. No physical segregation assumed |
| **ETS1** | Power, industry, aviation | Sustainability-compliant biomass is zero-rated for CO₂; produce the evidence that qualifies it, per installation |
| **ETS2** | Buildings, road transport, small combustion (obligations from 2027 ⚠️) | Attribute volumes to the obligated party; the attribution reduces the obligation and must not also be claimed elsewhere |
| **CSRD / ESRS E1** | Corporate sustainability reporting | Low-carbon fuel affecting gross Scope 1. Never netted. See above |
| **GHG Protocol** (revision in progress ⚠️) | Corporate accounting | Support market-based **and** location-based side by side, each with its methodology and auditor approval recorded |
| **Guarantees of Origin (GO)** | Renewable attribute certificates for gas | GO and PoS are different instruments over the same molecule. Claiming both is double counting — detect and refuse |
| **Voluntary schemes** (ISCC EU, REDcert EU, 2BSvs…) | Certification against RED criteria | Scheme is per-scope configuration; certificate validity windows drive expiry alerts and block claims once lapsed |

---

## The GHG intensity calculation

This is the arithmetic the whole certification rests on. RED II Annex V (biofuels) and **Annex VI
(biomethane and other biomass fuels)** define it as a chain of terms:

```
E = eec + el + ep + etd + eu − esca − eccs − eccr
```

| Term | Meaning | Typical driver for biomethane |
|---|---|---|
| `eec` | Extraction or cultivation of raw materials | Feedstock type. Wastes and residues are **zero-rated** here — the reason Annex IX feedstocks dominate |
| `el` | Annualised land-use change emissions | Zero for wastes and residues |
| `ep` | Processing | Digester parasitic load, upgrading energy (membrane vs PSA vs amine), **methane slip** |
| `etd` | Transport and distribution | Feedstock haulage, digestate return, grid injection or virtual pipeline trucking |
| `eu` | Emissions from the fuel in use | Zero for biogenic CO₂ |
| `esca` | Savings from improved agricultural management | Rare; requires evidence |
| `eccs` | Savings from carbon capture and geological storage | Where the upgrading CO₂ is captured — the CCUS overlap |
| `eccr` | Savings from carbon capture and replacement | Where captured CO₂ substitutes fossil CO₂ |

**Saving relative to a fossil comparator:**

```
Saving % = (E_fossil − E_biomethane) / E_fossil × 100
```

⚠️ Working comparator values: **94 gCO₂eq/MJ** transport · **80** heating/cooling · **183**
electricity.

### Why every term is stored separately

An auditor does not accept "our intensity is 18 g/MJ". They ask which digester, which upgrading
technology, what methane slip was assumed, what the haulage distance was, and where each figure came
from. `B2` therefore stores **each term with its own provenance** — value, unit, source (measured /
default / disaggregated default), the evidence reference, and who entered it.

The two terms that decide most projects:

| | |
|---|---|
| **Methane slip in `ep`** | Upgrading technology and its maintenance state. A slip figure taken from a datasheet rather than measured is the most common weakness in a dossier — and the difference between passing and failing a threshold. |
| **Manure credit in `eec`** | Avoided methane from manure management can produce a *negative* term, and hence a saving above 100%. Legitimate, valuable, and heavily scrutinised — the evidence requirement is proportionally higher. |

### Thresholds ⚠️

Applicable threshold depends on **when the installation started operating** and **what the fuel is
used for**. Working set, to be confirmed by `R0`:

| End use | Installation start | Minimum saving |
|---|---|---|
| Transport fuel | from 1 Jan 2021 | 65% |
| Electricity / heating / cooling | 2021–2025 | 70% |
| Electricity / heating / cooling | from 2026 | 80% |

**The threshold is scope-resolved, not a constant.** Two branches of the same company with digesters
commissioned in different years face different thresholds, and a plant crossing an installation-age
boundary faces a different one next year. Hard-coding it produces a system that certifies correctly
this year and incorrectly the next — see rule `R-SUS-01`.

---

## Feedstock and Annex IX

| Category | Treatment | Examples |
|---|---|---|
| **Annex IX Part A** | Advanced; eligible for **double counting** toward transport targets | Manure and sewage sludge, biowaste, straw, husks, cobs |
| **Annex IX Part B** | Double counting, but **capped** at member-state level ⚠️ | Used cooking oil, animal fats cat. 1 & 2 |
| **Food/feed crops** | Capped; land-use criteria apply | Maize silage, energy crops |
| **Other wastes/residues** | Zero `eec`; not necessarily Annex IX | Food processing residues |

Two rules fall directly out of this:

- **`R-FEED-01`** — Part A ⇒ double-counting eligible; Part B ⇒ eligible but capped.
- **`R-FEED-02`** — food/feed crop share above the national cap ⇒ refuse. This is the rule that
  needs *nested* conditions over a *scope-resolved* cap, and it is the acceptance test for the
  condition builder (`R3`).

---

## Mass balance

Sustainability characteristics travel with volumes through a shared physical system. Within a
defined **period** and **site**, the sum of sustainable inputs must equal the sum of sustainable
outputs, within tolerance.

```
Σ(sustainable in)  ≈  Σ(sustainable out)  +  closing balance − opening balance
```

Design consequences:

- **The balance is a scheduled aggregate, not a per-transaction check.** `R-MB-01` runs over a
  window, which is why the rules engine must support period aggregation or delegate to a job. This
  is the second acceptance test for `R1`/`R2`.
- **Period and site are configuration, per scope.** Different schemes and member states permit
  different balancing periods.
- **A failing balance blocks allocation, it does not reverse history.** Reversing an allocation
  already submitted to the UDB is a correction workflow (`W-UDB`), not a delete.

---

## Double counting — the central integrity rule

A volume may be claimed **once**. The ways to claim it are:

```
UDB transaction  ·  Guarantee of Origin  ·  ETS attribution  ·  Corporate disclosure (ESRS)
```

Four registers, four teams, four screens — and no natural place for the check. So the platform needs
**one claim register that every downstream use consults**, exactly as `EfQuotaGuard` is the single
place quota is enforced. Anything else produces two subsystems that each believe they own the
volume, and the disagreement is discovered by an auditor.

`R-DC-01` is therefore the one rule that **must not be configurable off**. Task `B3` builds the
register; task `R3` asserts the UI cannot disable the rule.

> **GO and PoS are different instruments over the same molecule.** Selling the Guarantee of Origin
> and separately claiming the sustainability attribute is the classic double count, and it is easy
> to do accidentally because two different departments hold the two instruments.

---

## Rule catalogue

These are the acceptance fixtures for the condition builder (`R1`) and rule assignment (`R2`). Use
them rather than invented examples — an engine proven against these is proven.

| ID | Rule | Trigger | Shape it exercises |
|---|---|---|---|
| `R-SUS-01` | GHG saving below the threshold for the installation's start date and end use ⇒ reject | Batch evidence complete | Numeric comparison against a **scope-resolved** threshold |
| `R-SUS-02` | No valid PoS reference ⇒ block allocation | Allocation requested | Presence + validity window |
| `R-SUS-03` | Certifying scheme's certificate lapsed ⇒ block | Allocation requested; scheduled | Date window against configuration |
| `R-FEED-01` | Annex IX Part A ⇒ double-count eligible; Part B ⇒ eligible, capped | Batch registered | Category lookup driving a branch |
| `R-FEED-02` | Food/feed crop share above national cap ⇒ refuse | Batch registered | **Nested** boolean over a scope-resolved cap |
| `R-MB-01` | Mass balance outside tolerance for the period and site ⇒ block further allocation | Scheduled, per period | **Aggregate over a window** |
| `R-DC-01` | Volume already claimed in UDB, as a GO, against ETS, or in a disclosure ⇒ refuse | Any claim | Cross-register uniqueness. **Not disableable** |
| `R-ETS-01` | Zero-rate for ETS1 only when sustainability evidence is complete for the installation | Reporting period close | Composite AND over presence checks |
| `R-ETS-02` | Attribute volume to the ETS2 obligated party; refuse if attributed elsewhere | Allocation | Uniqueness + counterparty resolution |
| `R-CSRD-01` | Classify as low-carbon fuel affecting gross Scope 1 — never as an offset netted against gross | Disclosure build | Classification enforced at write time |
| `R-GHG-01` | Market-based figure without an auditor-approved methodology reference ⇒ block | Disclosure build | Presence gated on figure type |
| `R-UDB-01` | Transaction not submitted within the mandated window ⇒ escalate | Scheduled | Time-based escalation |
| `R-SLIP-01` | Methane slip above the configured ceiling for the upgrading technology ⇒ flag for review | Batch evidence | Threshold per equipment class |
| `R-DIG-01` | Digestate outlet unverified ⇒ warn (does not block) | Batch registered | **Warning, not blocker** — proves the severity split |

`R-SLIP-01` and `R-DIG-01` earn their place by being the two that are *not* hard blocks: one flags,
one warns. A catalogue where everything blocks trains people to ignore everything.

---

## Workflow catalogue

Acceptance fixtures for the stage designer (`W1`–`W4`).

| ID | Workflow | Stages | Capability it proves |
|---|---|---|---|
| `W-BATCH` | Batch sustainability approval | Registered → Evidence attached → Technical review → Certifier verification → Approved / Rejected | A stage owned by an **external party** |
| `W-ALLOC` | Volume allocation to an off-taker | Requested → Double-count check → Mass-balance check → Allocated → UDB submitted → Confirmed | **Asynchronous failure**: a stage completes, then a later external confirmation reverses it |
| `W-UDB` | UDB submission with correction | Queued → Submitted → Accepted / Rejected → Corrected → Resubmitted | Retry with correction, and a compensating path |
| `W-ETS` | ETS attribution and reporting | Draft → Evidence complete → Internal approval → Verifier sign-off → Submitted | **Conditional auto-approval** where evidence is complete, manual otherwise |
| `W-CSRD` | Disclosure preparation | Data collected → Gross computed → Market-based computed → Methodology attached → Auditor review → **Locked** | An **immutable terminal state** |
| `W-CERT` | Certificate renewal | Expiring → Renewal requested → Evidence submitted → Renewed / Lapsed | A **time-triggered start** — no user action |
| `W-DIGEST` | Digestate offtake | Produced → Analysed → Contracted → Dispatched → Confirmed | Ordinary multi-party chain; the volume baseline |

Three capabilities a naive stage designer will not have, and which `W1`–`W4` must therefore cover:

1. **Time-triggered start** (`W-CERT`) — begins because a date arrived.
2. **Asynchronous stage failure with compensation** (`W-ALLOC`) — the saga case.
3. **Immutable terminal state** (`W-CSRD`) — locked against administrators too.

---

## The Ace vertical slice (`B5`)

The demo, and the proof the platform works for a real operator.

```
Tenant     ace
Company    islabhasolutions
Branch     spain
Industry   biomethane
Units      digester-1, upgrading-1, injection-point-1
```

Seeded with:

- A feedstock mix — manure (Annex IX A) plus food processing residues
- Two batches: one comfortably above threshold, one **marginal**, so `R-SUS-01` is visible doing
  its job rather than passing silently
- A certificate approaching expiry, so `W-CERT` fires on schedule
- One allocation that the UDB stub rejects, so `W-ALLOC`'s compensation path runs
- An offset plan from [OFFSET.md](./OFFSET.md) — see the biomethane measure library

---

## Data model summary

| Entity | Holds | Notes |
|---|---|---|
| `Batch` | Volume, period, site, status | **The unit of account** — not a meter reading |
| `Feedstock` | Category, Annex IX class, origin, quantity | Drives `eec` and double counting |
| `GhgIntensity` | Each Annex VI term, with provenance | Per term: value, unit, source, evidence, actor |
| `ChainOfCustody` | Mass-balance movements | Input/output per period and site |
| `Certificate` | Scheme, number, validity window | Drives `R-SUS-03` and `W-CERT` |
| `ProofOfSustainability` | Issued PoS, linked batch and certificate | The evidence artefact |
| `Claim` | Register, counterparty, volume, status | **The single uniqueness point** — `R-DC-01` |
| `UdbTransaction` | Payload, submission state, retries | Reconciles against `Claim` |
| `Disclosure` | Gross and market-based figures, methodology, lock state | ESRS E1 output |

---

## Open questions for `R0`

Recorded rather than guessed. Each needs a primary source and compliance sign-off.

1. Exact GHG-saving thresholds by installation commissioning date and end use under RED III as
   transposed in **Spain** (the Ace branch's jurisdiction).
2. The Annex IX Part B national cap applicable in Spain, and how it is measured.
3. UDB transaction schema, required fields, and the mandated submission window.
4. ETS2 obligated-party definition and the 2027 commencement detail.
5. Whether the ESRS E1 datapoints require gross figures before *or* after market-based instruments
   at each of the three scopes.
6. Which voluntary scheme Ace certifies under — decides certificate fields and audit cadence
   (open decision #3 in [Deliverables.md](./Deliverables.md#open-decisions)).
7. Whether manure credits producing >100% savings are accepted by the chosen scheme without
   additional evidence.

---

## Sources

Primary and secondary references used to shape this document. `R0` verifies against the primary set.

- European Commission — Climate Action: <https://climate.ec.europa.eu/index_en>
- European Biogas Association — Sustainability policy: <https://www.europeanbiogas.eu/policies/sustainability/>
- Bioenergy Insight — biomethane in the carbon market, ETS1 & ETS2:
  <https://www.bioenergy-news.com/news/biomethane-in-the-carbon-market-what-operators-need-to-know-about-ets1-and-ets2/>
- Bioenergy Insight — policy feed: <https://www.bioenergy-news.com/news_feed/policy/>
- ECB environmental statement 2026: <https://www.ecb.europa.eu/ecb/climate/green/html/ecb.environmentalstatement202607.en.html>
- Commodity Inside — carbon pricing and credit rules: <https://commodityinside.com/carbon-market-weekly-regulators-corporates-and-registries-recalibrate-carbon-pricing-and-credit-rules/>
- Persefoni — carbon offset programs: <https://www.persefoni.com/blog/carbon-offset-programs>
- Is Labha Solutions — biomethane service line: <https://islabhasolutions.com/services/biomethane/>

> Three PDFs previously in `EU_Biomethane/` (an EP research briefing `ECTI_BRI(2026)780417_EN`, an
> IFRI European biomethane sector report of June 2026, and one further paper) informed the shape of
> this document and are being removed from the repository. `R0` must re-source their content from
> the publishers rather than relying on this summary.
