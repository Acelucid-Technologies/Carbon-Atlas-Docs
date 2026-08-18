# Offset & Reduction Planning

How the platform turns a measured footprint into a ranked, costed plan of action — and why the plan
differs at every branch.

← [Deliverables.md](./Deliverables.md) · see also [EU-BIOMETHANE.md](./EU-BIOMETHANE.md) · [RULES-AND-WORKFLOWS.md](./RULES-AND-WORKFLOWS.md)

---

## "Offset" is the wrong word for most of what this does

The engine is named for offsets and spends most of its time on **reductions**. The distinction is
not pedantry — it is the difference between a defensible disclosure and a rejected one.

```
Reduction    the emission does not happen        →  lowers gross Scope 1/2/3
Removal      CO₂ is taken out and stored         →  disclosed separately, never netted
Credit       somebody else reduced, you paid     →  disclosed separately, never netted
```

Under **ESRS E1** all three are reported, and only the first changes the gross figure. A tool that
lets a customer subtract purchased credits from gross emissions is helping them file an incorrect
regulatory disclosure.

So the plan is ordered: **reduce first, then remove, then credit the residual** — and the UI says so,
because the ordering is the professional advice embedded in the product, not a default somebody can
reverse without noticing.

---

## Prioritisation

```
Priority score  =  (annual abatement tCO₂e  ×  ROI)  /  implementation complexity
```

| Term | Source | Notes |
|---|---|---|
| **Annual abatement** | Measure library × site baseline | Where the emissions actually are, not where they are easiest to find |
| **ROI** | `(annual saving − annual cost) / capex`, over the measure's life | Negative-cost measures exist and should rank top |
| **Complexity** | 1–5: permits, downtime, integration, supply lead time | The term that stops the ranking being a spreadsheet of good intentions |

Grouped into a roadmap the customer can act on:

| Horizon | Window | Typical |
|---|---|---|
| **Quick wins** | 0–6 months | Controls tuning, leak repair, tariff and contract changes, metering |
| **Medium term** | 6–18 months | Heat recovery, motor and drive replacement, PPA, process optimisation |
| **Long term** | 18–36 months | Fuel switching, electrification, capture, major plant |

**Rule-based for the MVP** (`O2`), deliberately. AI ranking needs outcome data — which measures were
actually implemented and what they actually saved — and that does not exist until pilots have run
for a year. Deferred as `X4`. Shipping an AI ranking with no outcome data is a confident guess
wearing a lab coat.

---

## Why the plan differs by branch and location

This is the requirement — *"branches/locations would have different rules"* — and there are five
independent reasons, each of which changes the answer for the same measure.

| Factor | Effect | Example |
|---|---|---|
| **Grid emission factor** | Decides whether electrification helps or hurts | A heat pump in France (low-carbon grid) abates far more than the same unit on a coal-heavy grid, where it can *increase* emissions |
| **Energy prices and tariffs** | Drives ROI | The same measure can pay back in three years at one site and eleven at another |
| **Incentives and subsidies** | Change capex | National and regional schemes differ, have deadlines, and are often the deciding factor |
| **Permits and consents** | Change complexity, sometimes to "impossible" | An urban site may be unable to site a digester regardless of economics |
| **Physical constraints** | Gate feasibility | Grid connection capacity, land, roof loading, heat sink availability |

**Consequence for the model:** a measure's abatement, cost and complexity are **not properties of
the measure**. They are properties of *(measure × site)*. The library holds the shape and the
defaults; the site holds the answer.

```
MeasureDefinition        vertical, description, mechanism, defaults          ← platform, scoped
SiteMeasureAssessment    abatement, capex, opex, complexity, feasibility     ← per site
```

Getting this backwards — one global abatement figure per measure — produces a plan that is confidently
wrong at every site except the one it was calibrated on, and the customer discovers it after
committing capital.

---

## Measure libraries

Platform-owned defaults in the `GLOBAL` scope so a tenant is useful on day one; tenants and branches
narrow and override. Same pattern as the UDF static-field catalogue.

### Biomethane

| Measure | Mechanism | Typical horizon | Notes |
|---|---|---|---|
| Methane slip reduction | Upgrading maintenance, membrane replacement | Quick win | Often the highest-ROI item, and it *also* improves `ep` in the intensity chain — abatement and certification value together |
| Digester heat recovery | CHP or upgrading waste heat to digester | Medium | Reduces parasitic load |
| Feedstock mix shift | More Annex IX A, less crop | Medium | Changes `eec` **and** double-counting eligibility. Cross-check `R-FEED-02` |
| Digestate valorisation | Nutrient recovery instead of disposal | Medium | Revenue as well as abatement |
| Grid injection vs virtual pipeline | Remove trucking | Long | Large `etd` reduction; capex and consent heavy |
| CO₂ capture on the upgrader | Capture the separated stream | Long | `eccs`/`eccr`. **Overlaps CCUS — claim once** |
| Electrify site loads on a clean grid | Replace on-site fossil | Medium | Grid-factor dependent; can be negative on a dirty grid |

### Sustainable fuels (H₂ / PtX)

| Measure | Mechanism | Horizon |
|---|---|---|
| Hourly PPA matching | Align production to renewable availability | Medium — also an RFNBO compliance requirement |
| Electrolyser efficiency | Stack replacement, operating-point tuning | Medium |
| Waste heat recovery from electrolysis | Feed a district or process heat sink | Long |

### Industrial heat

| Measure | Mechanism | Horizon |
|---|---|---|
| Controls and setpoint optimisation | No capex | **Quick win — usually top of the list** |
| Steam trap and insulation programme | Maintenance | Quick win |
| High-temperature heat pump | Replace fossil boiler, 80–160 °C | Medium |
| Thermal energy storage | Shift load to cheap/clean hours | Medium |
| Solar thermal / CSP-LPT | Process heat | Long |

### CCUS

| Measure | Mechanism | Horizon |
|---|---|---|
| Heat-integrated capture design | Cut the capture energy penalty | Medium |
| Utilisation offtake | Replace fossil CO₂ supply | Medium — **check permanence; utilisation that re-releases is not storage** |
| Geological storage | Permanent | Long |

### Water & wastewater

| Measure | Mechanism | Horizon |
|---|---|---|
| Aeration control | The dominant electrical load at most plants | Quick win |
| Sludge to anaerobic digestion | Energy recovery | Medium — **feeds the biomethane vertical** |
| Nutrient recovery | Struvite and similar | Medium |
| Water reuse | Cut abstraction and treatment | Medium |

### Waste management

| Measure | Mechanism | Horizon |
|---|---|---|
| Sorting-line optimisation | Higher recovery, less residue | Quick win |
| RDF valorisation | Displace fossil fuel in cement or similar | Medium |
| Pyrolysis / gasification | Divert from landfill | Long |
| Sensorised logistics | Route and fill optimisation | Quick win |

---

## Cross-vertical measures

The most valuable ones cross boundaries, and they are also where double counting happens.

| Measure | Verticals | Watch |
|---|---|---|
| CO₂ capture on a biogas upgrader | Biomethane + CCUS | One physical tonne, two possible claims. `R-DC-01` must see both |
| Sludge digestion at a wastewater plant | Water + Biomethane | Produced gas may also be certified — one volume, one claim |
| RDF into cement | Waste + Industrial heat | Abatement belongs to one party; contract decides which |
| Waste heat to a district network | Heat + any process | Both sides may claim; the register decides |

**A single claim register is the only workable answer**, and it must be vertical-agnostic from the
start — see the gap flagged in [RULES-AND-WORKFLOWS.md](./RULES-AND-WORKFLOWS.md).

---

## The credit half

For the residual that reduction and removal cannot reach.

Recommendations, never automatic purchases. The system:

- ranks credit types by **permanence, additionality, verification standard and price**;
- records the standard (Gold Standard, Verra, Article 6, national scheme) against the claim;
- registers every credit in the **claim register** so it cannot also be counted elsewhere;
- keeps credits **out of the gross figure**, always — `R-CSRD-01` enforces it at write time.

The platform is not a credit marketplace and should not become one. It records what a customer holds
and ensures it is disclosed correctly. Brokering introduces a conflict of interest into a tool whose
value is being trusted about numbers.

---

## Model

| Entity | Holds | Scope |
|---|---|---|
| `MeasureDefinition` | Vertical, mechanism, description, default abatement basis | Platform / tenant |
| `SiteMeasureAssessment` | Abatement, capex, opex, complexity, feasibility, incentives | **Per site** |
| `OffsetPlan` | A ranked set for a scope and period | Per scope |
| `PlanItem` | Measure, horizon, score, status, owner, target date | |
| `LocationFactor` | Grid factor, energy price, incentive scheme, permit regime | **Per branch/location** |
| `CreditHolding` | Type, standard, vintage, quantity, retirement state | Registers into `Claim` |

`LocationFactor` is the entity that makes the per-branch requirement real. Without it, location
differences live in a consultant's spreadsheet and the plan is generic.

---

## Pitfalls

- **Never net credits against gross.** `R-CSRD-01`. This is the one that makes a disclosure wrong.
- **Electrification is not always abatement.** On a carbon-intensive grid a heat pump can raise
  emissions. The grid factor must be per location and current, and the engine must be able to
  return a *negative* abatement rather than clamping it to zero.
- **Do not rank by abatement alone.** The largest measure is usually the one that never gets funded.
  Complexity is in the formula for that reason.
- **Cross-vertical measures need one claim.** Capture on an upgrader is the worked example.
- **A plan is a snapshot, not a live view.** It is presented to a board and funded against. Version
  it, date it, and keep what was recommended and on what basis — the same reasoning as the
  calculation audit snapshot (`E4`).
