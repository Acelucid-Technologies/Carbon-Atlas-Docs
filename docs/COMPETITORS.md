# Competitive Landscape

Twenty-one carbon accounting and climate platforms, what they are strong at, and the gaps worth
taking.

← [Deliverables.md](./Deliverables.md)

---

> **Sourced from public positioning, not from usage.** Pricing is rarely published and feature claims
> come from vendors' own marketing. Treat this as a map of where the market says it is, and verify
> anything a deal depends on. Nothing here should be repeated to a customer as fact about a
> competitor.

---

## The landscape

| Company | HQ | Focus | Target | Pricing |
|---|---|---|---|---|
| [Normative](https://normative.io) | Sweden | Carbon accounting, Scope 3 | Mid-market, enterprise | Custom |
| [Watershed](https://watershed.com) | USA | Enterprise carbon accounting | Large enterprise | Custom |
| [Persefoni](https://persefoni.com) | USA | Accounting & climate disclosure | Enterprise | Free tier + enterprise |
| [Greenly](https://greenly.earth) | France | Footprint measurement | SMB, mid-market | From ~€4k/yr |
| [Sweep](https://www.sweep.net) | France | Accounting & supplier engagement | Enterprise | Custom |
| [Plan A](https://plana.earth) | Germany | Accounting + ESG | Mid-market | Custom |
| [Emitwise](https://emitwise.com) | UK | Supply chain emissions | Enterprise | Custom |
| [SINAI](https://www.sinai.com) | USA | Decarbonisation planning | Enterprise | Custom |
| [Salesforce Net Zero Cloud](https://www.salesforce.com/products/net-zero-cloud) | USA | Accounting & reporting | Enterprise | Custom |
| [IBM Envizi](https://www.ibm.com/products/envizi) | USA | Accounting & ESG | Enterprise | Custom |
| [Green Project](https://greenprojecttech.com) | UK | Accounting & reporting | SMB, enterprise | Custom |
| [Sustain.Life](https://www.sustain.life) | USA | Accounting & net-zero planning | SMB, mid-market | Custom |
| [CarbonChain](https://carbonchain.com) | UK | Supply chain, commodities | Manufacturing | Custom |
| [Carbon Direct](https://www.carbondirect.com) | USA | Accounting + advisory | Enterprise | Custom |
| [CarbonCloud](https://carboncloud.com) | Sweden | Product footprints | Food & beverage | Custom |
| [Ecochain](https://ecochain.com) | Netherlands | Product LCA | Manufacturing | Custom |
| [Muir AI](https://www.muir.ai) | USA | AI-powered accounting | SMB | Custom |
| [Connect Earth](https://connectearth.com) | UK | Transaction-based APIs | Financial services | Custom |
| [ClimatePartner](https://www.climatepartner.com) | Germany | Footprinting & offsetting | All sizes | Custom |
| [Greenplaces](https://www.greenplaces.com) | USA | Accounting & ESG | SMB | Custom |
| [Cloverly](https://www.cloverly.com) | USA | Calculation & offset APIs | Developers | Custom |

---

## What the market is, in one paragraph

Almost all of these are **horizontal corporate carbon accounting**: ingest activity data across any
industry, apply factors, produce Scope 1/2/3 figures and a disclosure. They compete on data coverage,
Scope 3 methodology, supplier engagement and reporting polish. Four are specialists — CarbonCloud
and Ecochain on product-level LCA, Connect Earth on financial transactions, Cloverly on offset APIs.

**Almost none are operational systems for renewable-energy production assets.** That is the gap.

---

## Where the crowd is strong

Being honest about this matters more than the gap analysis, because these are the areas where
"we'll build it too" is a bad plan.

| Strength | Who | Why it is hard to match |
|---|---|---|
| **Emission factor breadth** | Watershed, Normative, Persefoni | Years of curation across geographies and sectors. **Buy this, do not build it** — see `E1`'s third-party-plus-cache design |
| **Scope 3 supplier engagement** | Emitwise, Sweep, CarbonChain | Network effects: value grows with suppliers already on the platform |
| **Enterprise distribution** | Salesforce, IBM | Sold into an existing seat. Cannot be out-competed on reach |
| **Disclosure polish** | Persefoni, Watershed | Mature, audited report templates across frameworks |
| **Product LCA depth** | CarbonCloud, Ecochain | Deep, narrow methodology in food and manufacturing |

---

## The gaps worth taking

Five, ordered by how defensible each is.

### 1. Operational compliance for renewable production — the strongest

Every competitor answers *"what were your emissions?"* Nobody in this list answers *"is this batch of
biomethane certifiable under RED III, and has this volume already been claimed?"*

That needs a batch ledger, mass balance, GHG intensity along the Annex VI chain, PoS and certificate
lifecycle, UDB submission, and a **claim register preventing double counting** — see
[EU-BIOMETHANE.md](./EU-BIOMETHANE.md). It is operational and regulatory rather than reporting.

**Why it is defensible:** it is domain depth, not features. A horizontal platform cannot add it
without acquiring the expertise, and the expertise is scarce. It is also the thing an operator
*cannot* run the business without — which makes it a different purchase from a reporting tool.

### 2. Configurable rules and workflows

Competitors ship fixed methodologies and fixed approval flows. Carbon Atlas ships an **engine** — a
business analyst authors a nested-condition rule and a multi-stage workflow at any scope level.

**Why it matters commercially:** every regulated customer has a compliance step nobody anticipated.
For the crowd that is a support ticket and a release; here it is an afternoon in the builder. It is
also how six further verticals become configuration rather than six products
([RULES-AND-WORKFLOWS.md](./RULES-AND-WORKFLOWS.md)).

**The risk:** an engine is worthless without a good authoring surface. `R1`–`R4` and `W1`–`W4` are
the product, not a feature.

### 3. Scope depth — six levels plus application

Competitors typically model organisation → site. Carbon Atlas resolves across
**PLATFORM → TENANT → COMPANY → BRANCH → INDUSTRY → UNIT**, with application orthogonal, and applies
it to *everything*: permissions, quota, wallets, providers, rules, form fields, list columns, labels
and language.

**Why it matters:** a multi-site operator with different permits, grid factors and incentives per
branch is the normal case in this industry. "Ten form fields at one site, eight at another, in
Spanish, with a stricter methane ceiling" is one configuration here and a custom build elsewhere.

### 4. Sector-specific offset planning

Most competitors stop at measurement, or hand off to consultants. [OFFSET.md](./OFFSET.md) specifies
per-vertical measure libraries with **per-site assessment** — because a heat pump's abatement depends
on the local grid factor, and its ROI on the local tariff.

**The insight worth holding:** measurement is becoming a commodity; knowing what to *do* is not.

### 5. Technology-integrator positioning

Is Labha Solutions deploys certified Indian OEM equipment into the EU under EU industrial standards,
leveraging the EU–India FTA. Carbon Atlas is the digital layer over that — pre-FID modelling, then a
live operational twin, with **RED III, Guarantees of Origin and Proof of Sustainability built in from
day one**.

No pure-software competitor can offer the plant. No equipment integrator offers the software. The
combination is the position.

---

## Where we are behind, and what to do

| Gap | Reality | Response |
|---|---|---|
| **Factor database** | Years behind Watershed and Normative | Buy it. `E1` is third-party plus cache, behind an interface. Do not build |
| **Scope 3 supplier network** | No network | Do not compete here. Focus on Scope 1/2 and production compliance, where the value is operational |
| **Disclosure template breadth** | ESRS E1 only | Correct for now — EU biomethane is the beachhead. Add frameworks on customer demand, not speculatively |
| **Brand and references** | None | The Ace vertical slice (`B5`) is the first reference. Make it real, not a demo |
| **Enterprise distribution** | None | Sell through the integrator relationship, not against Salesforce |

---

## Positioning

> **The others tell you what you emitted. Carbon Atlas runs the plant that emits less, proves it to
> the regulator, and shows you what to fix next.**

Three sentences for a sales conversation:

1. **Compliance is operational, not reporting.** RED III certification, mass balance and UDB
   submission are things you do weekly, not annually. Nobody else does them.
2. **Your sites are not the same.** Different permits, grids, tariffs and incentives — so different
   rules, different forms, different plans. Configured, not custom-built.
3. **We do the plant as well as the software.** Certified OEM equipment deployed to EU standards,
   with the digital twin and the bankable dossier from day one.

---

## What to watch

| Signal | Why it matters |
|---|---|
| A horizontal platform adding RED III certification | Directly attacks gap #1. Most likely from a European vendor — Greenly, Plan A or Normative |
| ETS2 commencement (2027) | Pulls buildings and road transport into scope; expands the addressable market and every vendor will chase it |
| GHG Protocol / ISO consolidation | Changes market-based accounting rules. `E2`'s dual-figure storage is the hedge |
| UDB enforcement tightening | Raises the compliance bar and the value of gap #1 |
| A specialist emerging in biomethane certification | The real competitor is not in this table yet |

That last row is the honest one. The dangerous competitor is not Watershed adding a feature — it is a
European biogas-industry specialist with fifteen years of certification expertise deciding to build
software. The defence is to be operationally embedded before they arrive.
