# Rules & Workflows

The taxonomy that lets seven industry verticals share one engine — so adding a vertical is
configuration and a measure library, not a rebuild.

← [Deliverables.md](./Deliverables.md) · see also [EU-BIOMETHANE.md](./EU-BIOMETHANE.md) · [OFFSET.md](./OFFSET.md)

---

## The design claim

Carbon Atlas serves seven sustainable-energy verticals. They look unrelated — a digester, an
electrolyser, a heat pump, a capture unit, a wastewater plant, a gasifier, a SaaS twin. But the
*shape* of what the platform does for each is the same:

> Take a **measured quantity**, attach **evidence** to it, run it through **rules** that decide
> whether it qualifies, move it through an **approval chain**, and claim it **once** in a register.

Seven verticals × bespoke engines is seven products. One engine × seven configurations is a
platform. This document is the taxonomy that makes the second one true.

**The test of whether it holds:** a new vertical should need a measure library, a rule set, a
workflow set and a form definition — and no new engine code. Where that is not true, it is called
out below as a gap.

---

## Rule taxonomy

Every rule the platform needs falls into one of seven shapes. The condition builder (`R1`) must
express all seven, and the fixtures in `R3` prove it.

| # | Shape | Example | What the engine needs |
|---|---|---|---|
| **1** | **Threshold against a scope-resolved value** | GHG saving ≥ 65% *for this installation's start year* | The comparison operand is resolved by scope, not a literal. **The one people hard-code.** |
| **2** | **Presence and validity** | A valid, unexpired PoS must exist | Null check plus a date window |
| **3** | **Category lookup driving a branch** | Annex IX Part A ⇒ double count; Part B ⇒ capped | Reference-data join inside a condition |
| **4** | **Nested boolean** | `feedstock IN (…) AND (country = X OR country = Y) AND share > cap` | Arbitrary AND/OR nesting. **The acceptance test.** |
| **5** | **Aggregate over a window** | Mass balance reconciles for the period and site | Period aggregation — engine or scheduled job, but one implementation |
| **6** | **Cross-register uniqueness** | This volume is not already claimed anywhere | A single register every consumer reads |
| **7** | **Time-based escalation** | Not submitted within the mandated window ⇒ escalate | A scheduler that can start a rule, not only a user action |

### Severity is part of the rule, not the caller

```
Block     the action cannot proceed
Flag      proceeds, but enters a review queue
Warn      proceeds, shown once, recorded
```

A catalogue where everything blocks trains people to route around it. The biomethane set
deliberately includes one flag (`R-SLIP-01`) and one warn (`R-DIG-01`) so the distinction is
exercised from day one rather than retrofitted when the first customer complains.

### Rules resolve by scope like everything else

A rule is assigned at any level — platform, tenant, company, branch, industry, unit — and
most-specific-wins. That is how "branch A's biomethane line has a stricter slip ceiling than branch
B's" is configuration rather than a code branch.

**Not disableable** is a property some rules carry (`R-DC-01`). It is enforced in the resolver, not
by greying out a checkbox: a UI-only guard is one API call away from being bypassed.

---

## Workflow taxonomy

Six patterns cover every vertical. `W1`–`W4` implement the capabilities; each vertical composes them.

| # | Pattern | Capability required | Vertical example |
|---|---|---|---|
| **1** | **Linear approval** | Stages, transitions, assignee by role and scope | Batch sustainability approval |
| **2** | **External party stage** | A stage owned by somebody outside the tenant | Certifier verification |
| **3** | **Conditional auto-approval** | Guard on a transition; auto when evidence is complete, manual otherwise | ETS attribution |
| **4** | **Asynchronous failure & compensation** | A completed stage reversed by a later external confirmation | UDB rejects an allocation already marked done |
| **5** | **Time-triggered start** | A workflow that begins because a date arrived | Certificate renewal |
| **6** | **Immutable terminal state** | Locked against everyone, administrators included | A filed disclosure |

**Patterns 4, 5 and 6 are the ones a naive stage designer lacks**, and they are not optional:
every vertical has a certificate that expires (5), an external registry that can reject after the
fact (4), and a filed report that must not change (6).

> **Reuse the condition builder for transition guards.** `W1` must consume `R1`'s component. Two
> condition editors will drift, and then the same expression means different things in a rule and in
> a workflow — a defect nobody can reproduce because both screens look correct.

---

## The seven verticals

Each row is what changes; the engine does not.

### 1. Biomethane

Anaerobic digestion, biogas upgrading (membrane / PSA), grid injection and virtual pipelines,
digestate and feedstock platforms.

Fully specified in **[EU-BIOMETHANE.md](./EU-BIOMETHANE.md)** — 14 rules, 7 workflows, the Annex VI
calculation chain.

| | |
|---|---|
| **Unit of account** | Batch (volume, period, site) |
| **Key regulation** | RED III, UDB, ETS1/2, CSRD |
| **Certification** | PoS via ISCC EU / REDcert EU |
| **Distinctive rule** | Mass balance over a period (`R-MB-01`) |
| **Distinctive workflow** | Certifier verification as an external stage |

### 2. Sustainable fuels — green H₂, PtX, e-methanol, SAF

| | |
|---|---|
| **Unit of account** | Production batch, by electrolyser and period |
| **Key regulation** | RED III **RFNBO** delegated acts — additionality, temporal and geographic correlation ⚠️ |
| **Certification** | RFNBO compliance certificates; CertifHy-style GOs |
| **Distinctive rule** | **Temporal correlation** — renewable electricity matched hourly to production. Shape 5 (aggregate over a window), but the window is an *hour*, not a quarter |
| **Distinctive workflow** | Grid-connection evidence with a PPA counterparty |
| **Gap** | Hourly matching is a heavier aggregate than mass balance. Confirm `R1`/`R2` handle it before committing a delivery date |

### 3. Industrial heat decarbonisation

High-temperature heat pumps (80–160 °C), thermal energy storage, solar thermal / CSP-LPT, power-to-heat.

| | |
|---|---|
| **Unit of account** | Site energy baseline and delivered heat, per period |
| **Key regulation** | Energy Efficiency Directive; national incentive schemes |
| **Certification** | Measurement & verification protocol (IPMVP-style) |
| **Distinctive rule** | **Baseline integrity** — a claimed saving is only valid against an agreed, unmodified baseline. Shape 1, where the operand is the baseline |
| **Distinctive workflow** | Baseline agreement, then periodic M&V with re-baselining on process change |
| **Note** | The rule that matters is *detecting* a process change that invalidates a baseline. Without it savings drift upward and nobody notices |

### 4. CCUS

Capture at industrial point sources, utilisation and offtake pathways, heat-integrated capture.

| | |
|---|---|
| **Unit of account** | Tonne of CO₂ captured, by source and destination |
| **Key regulation** | ETS monitoring & reporting; CO₂ transport/storage rules ⚠️ |
| **Certification** | Chain-of-custody to a storage or utilisation endpoint |
| **Distinctive rule** | **Permanence and fate** — captured ≠ stored. Utilisation that re-releases within a period does not qualify the same way as geological storage |
| **Distinctive workflow** | Custody transfer between capture, transport and storage operators, each a separate legal entity |
| **Overlap** | `eccs`/`eccr` in the biomethane chain. A capture unit on an upgrader is *both* a CCUS asset and a term in a biomethane intensity — one physical fact, two claims, and `R-DC-01` must see both |

### 5. Water & wastewater treatment

Industrial and municipal treatment, algal photobioreactors, nutrient recovery, water reuse, odour control.

| | |
|---|---|
| **Unit of account** | Volume treated and load removed, per period and outfall |
| **Key regulation** | Urban Wastewater Treatment Directive; discharge permits (local) |
| **Certification** | Permit compliance, sampling regime |
| **Distinctive rule** | **Consent limits per outfall** — genuinely per-location, set by the local permit, not by the company. Shape 1 with a *branch-level* operand |
| **Distinctive workflow** | Exceedance → notification → root cause → regulator report, on a statutory clock |
| **Note** | The clearest case for branch-scoped rules: two plants of one company have different legal limits |

### 6. Waste management & waste-to-energy

MBT sorting optimisation, pyrolysis and gasification, chemical recycling, RDF valorisation, sensorised logistics.

| | |
|---|---|
| **Unit of account** | Input tonnage by waste code, output by product and residue |
| **Key regulation** | Waste Framework Directive; end-of-waste criteria; shipment rules |
| **Certification** | Waste transfer notes, end-of-waste declarations |
| **Distinctive rule** | **Mass and energy balance across the process** — inputs must account for outputs plus residues. Shape 5, but per process line rather than per site |
| **Distinctive workflow** | End-of-waste determination with regulator sign-off |
| **Overlap** | RDF and biogenic fractions feed both this and biomethane/ETS accounting |

### 7. Digital intelligence (SaaS)

Decarbonisation and circular SaaS, digital twins, IT/OT and MRV, AI optimisation, automated
compliance, traceability.

Not a vertical with its own rules — **it is how the other six are delivered**. Its requirements land
on the platform as: continuous telemetry ingestion rather than batch upload, model-versioned
calculations, and traceability from a reported figure back to a sensor reading.

| | |
|---|---|
| **Gap** | Telemetry ingestion is a different class from file upload. Deferred as `X7`; do not promise a digital twin on the current ingestion path |

---

## What the taxonomy tells us to build

Reading the seven verticals together, three engine capabilities are load-bearing and one is missing.

| | |
|---|---|
| **Scope-resolved operands** | Five of seven verticals need a threshold that differs by branch or by installation vintage. Build it into the condition builder from the start — retrofitting it means revisiting every configured rule. |
| **Aggregation windows** | Four of seven need a balance or match over a period. Windows range from an hour (H₂ temporal correlation) to a quarter (mass balance). One implementation, configurable window. |
| **One claim register** | Three verticals can claim the same physical fact (a captured tonne on a biomethane upgrader feeding an ETS installation). Uniqueness cannot live in any one vertical. |
| **⚠️ Missing: cross-vertical claim reconciliation** | `B3`'s register handles biomethane. The CCUS/biomethane overlap means it must be built vertical-agnostic from the start. Flagged now because retrofitting a shared register after two verticals have their own is a migration, not a change. |

---

## Authoring rules and workflows — the operating model

**Who writes them.** Business analysts and compliance staff, in the builder — not developers in
code. That is the point of the engine, and it only holds if two things stay true:

1. **Dry-run before enable.** `R2` requires it. Without a dry-run people test in production, and a
   badly scoped rule that blocks every allocation is discovered by a customer.
2. **A rule change is versioned and audited.** Which rule, at what scope, changed by whom, and what
   it was before. A certification decision made under a rule that has since been edited is
   indefensible without it.

**Where they live.** Rules and workflows are tenant configuration, scope-resolved. Platform-owned
baseline sets ship in the `GLOBAL` scope so a new tenant is usable on day one, and a tenant narrows
rather than starts blank — the same pattern the UDF static-field catalogue uses.

**What developers still own.** The seven rule shapes, the six workflow patterns, and the engines.
If a customer requirement does not fit a shape, that is a platform change with a design discussion —
not a rule somebody forces into the wrong shape. Those requests are the roadmap for the engine.
