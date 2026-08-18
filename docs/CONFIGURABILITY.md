# Configurability

**Binding, not advisory.** Every new user-visible field goes through this document before it is built.

---

## What this answers

[ARCHITECTURE.md](./ARCHITECTURE.md#the-rule-for-choosing) answers *which store* — relational or
document. This is the sibling rule one level down: given the store, does this thing become a **column**,
a **configurable field**, a **rule**, a **catalogue row**, or **code**?

It is a different question from the one the SDK reference answers.
[`al.net.udf/README.md`](../../al.net.packages/al.net.udf/README.md) has a table called *"Which
variation to reach for"* — that is about which UDF mechanism to use once you have decided to use UDF.
**This document is about whether to use UDF at all**, and the honest answer is usually no.

---

## Why this document exists

The platform has a well-built UDF engine: scope resolution across five dimensions, per-culture labels
with overlay merge, sections and steps, server-enforced conditions, and a validator that gates on those
conditions so a hidden field is never required.

It also has a form builder — twenty-five files, roughly 2,900 lines, drag-and-drop grid layout, live
preview — that **has never contained the string "UDF"** and saves to `localStorage`.

Nobody did anything wrong. There was simply no point in the process where somebody had to answer
*"where does this value go?"*, so a form designer was built to completion without one. That question is
this document, and the six destinations below are the only permitted answers.

---

## The six destinations

| | Destination | Lives in | Changed by |
|---|---|---|---|
| **A** | Static column, **uncatalogued** | a Postgres column, nothing else | a developer, in a release |
| **B** | Static column, **catalogued** | the column **+** an `IUdfStaticFieldSource` entry | dev owns the column; tenant owns label, order, visibility, language |
| **C** | Custom UDF field | a `UdfDefinition` + a value in `custom_fields` | the tenant, at runtime, with no release |
| **D** | Hard-coded, deliberately not configurable | C# | nobody, without a release and a review |
| **E** | A **rule** | `al.rules`, scoped | the tenant, through rule authoring |
| **F** | **Reference / master data** | a catalogue row with its own lifecycle | platform or tenant, through a master-data console |

### A is a defect, not a choice

`UdfStaticFields` already has `.System()` for columns that exist but must never render — `id`,
`tenantId`, `connectionMode`. So a column is either **placeable** or **`System()`**, and both are B.
"A" is only what exists *before* somebody wrote the resource's static-field source.

**The standing instruction: every user-visible resource has an `IUdfStaticFieldSource`, and every column
on it is either placeable or `System()`.**

Today exactly one source exists — `al.udf/Data/MasterStaticFields.cs`, covering `master`/Tenant,
Company and Branch. Everything else on the platform is in A by omission rather than by decision,
including `carbon`/`biomethane-calculation`, which the seeder writes definitions against and which has
no catalogue at all. A form there is all-custom by accident, not by design.

---

## The seven gates

Work down the list. **The first "yes" is the answer.** Order matters — safety before queryability
before variance. Run it in reverse and you get a configurable permission flag.

### Gate 1 — Is it a control?

> If a tenant administrator set this to a nonsense value at 02:00, who finds out, and how long does it
> take?

If the answer is "nobody", or "an auditor, in eighteen months" → **D**.

If it grants access, moves money, or determines a regulatory outcome, it is not configuration. Covers:
role and permission semantics, application gating, `IsPlatform`, the Annex VI calculation chain, claim
uniqueness, discount-before-credit ordering.

> A form that lets a tenant administrator define what "role" means is a privilege-escalation surface
> with a drag handle.

### Gate 2 — Does our own shipped code read it by name?

> Would I write a `WHERE`, a `JOIN`, a `GROUP BY` or a `switch` on this, in code that ships with the
> product?

Yes → **B**. Never C.

`CustomFields["taxId"]` compiles perfectly. The cost arrives eighteen months later as
`custom_fields ->> 'taxId'` in a compliance report: unindexed, untyped, and keyed on a string a tenant
was free to name differently.

### Gate 3 — Does it need integrity, uniqueness, or an audit trail?

A jsonb column has **no foreign key, no check constraint, no unique index**. If the field is an FK, must
be unique, or must appear in an immutable snapshot → **B**.

**Regulatory corollary.** Anything reproduced in an ESRS E1, RED III or UDB submission needs a value
whose type *and definition history* are guaranteed. `UdfDefinition` carries no version history today, so
a jsonb value read through a definition that has since been edited cannot be defended to an auditor.
**Until that lands, every disclosure input is B. No exceptions.**

### Gate 4 — Is it a shape question or a policy question?

> Is this *"what does the form hold"* or *"when does something happen"*?

Thresholds, ratios, "when X then Y", validity windows, approval triggers → **E**.

UDF holds **shape**. A GHG-saving threshold expressed as a UDF field's default is a threshold nobody
can dry-run, monitor, or replay — all three of which `al.rules` gives you for free.

### Gate 5 — Is it a field on a thing, or a thing?

Emission factors, industries, data types, offset measures, certification schemes, location factors →
**F**.

These have their own lifecycle, their own version, and their own console. Modelling "per-tenant emission
factor override" as custom fields on a calculation is the single most expensive misfiling available
here, because an audit snapshot has to record *which factor version was used* and a bag cannot answer
that.

### Gate 6 — What kind of variation is it?

This is the B-versus-C discriminator, and the one people get wrong.

| The variation across tenants | Destination |
|---|---|
| None — every tenant has it, the platform breaks without it | **B** |
| Same value, different **name / position / language / visibility / required-here** | **B** |
| Same value, different **presence** (some industries capture it, others don't) | **B** + a `UdfCondition` |
| Tenants invent the field itself; the keys differ between tenants | **C** |
| One tenant asked for it and no roadmap item reads it | **C** |

**The common error is seeing *any* variation and reaching for C.** Most variation on an enterprise form
is presentational, and B already handles all of it: per-scope label overlay, translation with fallback,
order, colspan, per-application visibility, condition gating, and required-tightening.

The cost of B is one line in a builder chain. The cost of skipping it is a support ticket per tenant,
per label, forever.

### Gate 7 — What is the decommissioning path?

Only reached if the answer so far is C. Answer all three **before** creating the definition:

1. **Renaming `FieldKey` orphans every value already written under it.** It is immutable in practice
   once anything is saved — which is why the update request type does not expose it. Are we certain of
   the name?
2. **Deleting the definition strands the values.** They stay in the bag, the validator sees keys it does
   not recognise, and the record cannot be saved again. **Set `isRetired` instead** — a retired field
   stops rendering and stops being validated, but its key is still recognised, so those records keep
   saving. Delete only once a sweep confirms no values remain.
3. **Tightening a rule does not reach backwards.** See versioning below.

No confident answer to all three → **B**.

---

## How a changed rule affects data already captured

Every edit to a definition creates a new revision. The live document keeps its id and holds the
current revision; the outgoing one is archived beside it.

```
def-digestate            Version 3, IsCurrent true    ← forms resolve this, new capture uses it
def-digestate::v2        Version 2, IsCurrent false   ← records captured under v2 validate against this
def-digestate::v1        Version 1, IsCurrent false
```

Values record the revision they were captured under, in a reserved `__v` key inside the same JSON bag:

```json
{ "digestateOutputTonnes": 412.5,
  "__v": { "digestateOutputTonnes": 2 } }
```

| Situation | Behaviour |
|---|---|
| New value | Validated by the current revision, stamped with it |
| Existing value, edited | Keeps its original revision and is **still validated by it** |
| Value with no stamp | Reads as revision 1 — what pre-versioning records were captured under |
| Archived revision missing | Falls back to the live definition; losing history must not lose the record |
| Stamp for a removed value | Dropped, so the map does not accumulate every field the record ever held |

**What this does and does not buy you.** It means tightening a bound today cannot invalidate a record
saved last year, and that an auditor can be told which rule produced a figure. It does **not** mean two
versions of a form render side by side — a form always resolves the current revision. Versioning
governs *interpretation of stored values*, not presentation.

**Gate 3 still stands.** Definition history makes a jsonb value defensible; it does not give it a
foreign key, a unique index or a check constraint. Disclosure inputs remain **B**.

---

## Two structural constraints

**A condition source must be B or C, never A.** A `UdfCondition` keyed on `industryKey` needs that key
in the catalogue, or the client cannot render the gating and the server has nothing to evaluate against.
Uncatalogued columns cannot gate anything.

**No field participating in an authorization decision may be C.** The validator is the only integrity a
jsonb bag has. An authorization input in a bag is a vulnerability, not a configuration choice.

---

## The promotion path

```
C → B   Expected, and healthy. A key that 80% of tenants define identically is a column
        the product has not admitted to yet. Promote: add the column, add the catalogue
        entry, backfill from the bag, retire the definition.

B → C   Essentially never. It surrenders the foreign key, the index and the type, and it
        is a data migration. If you want per-tenant labels, B already gives you them.

C → E   Common. "Threshold" fields discovered in the bag belong in al.rules.
```

Promotion should be a decision, not a hunch. That needs a cross-tenant report of custom keys by
definition count — scheduled, not yet built.

---

## Worked examples

A framework with no worked examples gets re-derived differently by every reader.

### Biomethane plant fields

| Field | → | Why |
|---|---|---|
| `feedstockType`, `digestateHandling`, `transportDistanceKm` | **B** | The Annex VI chain reads them by name and the audit snapshot records them. Gates 2 and 3. The intuition "it varies per plant" is right, but the variation is *values*, not *shape*. |
| `plantPermitNumber`, `gridConnectionRef`, `shiftOperatorContact` | **C** | Nothing in our code reads them; they appear on a form and a tenant's own report. Gate 6, row 4. |
| GHG-saving threshold (65% / 70%) | **E** | A policy with a date-bound cut-over. In `al.rules` it can be dry-run before it bites. |
| Mass-balance period boundary; claim uniqueness | **D** | Not disableable. Gate 1. |
| Certification scheme (ISCC EU, REDcert) | **F** | A scheme has validity dates and a certificate lifecycle. It is a catalogue row, not a dropdown value. |

### Company onboarding — `master` / `Company`

`name`, `code`, `domain`, `taxId`, `type`, `status` → **B**, all already catalogued.

`taxId` is the instructive one. It looks like "just a string", but its own tooltip says it appears on
generated compliance documents — so Gate 3 pins it to a column before Gate 6 ever gets a chance to
argue about variation.

`id`, `tenantId`, `createdAt` → **B via `.System()`**.
`internalCostCentre`, `parentGroupRef`, `localSicCode` → **C**.

`industryKey` deserves its own note: it is deliberately *not* a column on `CompanyCatalog` — it lives on
the branch-industry relationship and is computed as a union. It is nonetheless the platform's canonical
condition source, which makes the first structural constraint concrete: **that key has to reach the form
as a catalogued field, or "selecting biomethane reveals four more fields" cannot resolve server-side.**

### User onboarding and role grants

`email`, `firstName`, `lastName`, `phoneNumber` → **B**.

Role, scope, application access, validity dates, grant type → **D**. Edited through a purpose-built
screen against real tables.

`employeeId`, `department`, `costCentre` → **C** — no authorization code reads them.

This is worth stating plainly because the opposite is tempting: the onboarding form *looks* like one
form, so it looks like one destination. It is two. The identity half is B and C; the granting half is D,
and the tell is Gate 1 — ticking no applications grants *every* application, so a hideable applications
selector silently grants the mobile app to everyone onboarded through it.

### Emission factors

**F**, and they already live in the document store. Value, source, vintage, region, version → catalogue
rows. *Which* factor set a branch uses → a **scoped preference**, because that is a resolution question
with a provenance view attached. Not a UDF field on any form.

### Compliance report fields

Every disclosed figure → **B**, and locked at submission.

Qualitative narrative a tenant authors → **C is acceptable only if the snapshot copies the resolved
field definition alongside the value at lock time.** That is a one-line addition where the snapshot is
written, and it is the whole difference between a reproducible disclosure and a liability.

---

## The register

The gates decide new fields. **The register is what a reviewer checks a pull request against.**

| Module | Resource | Owner | Static-field source | Status |
|---|---|---|---|---|
| `master` | Tenant | al.master | `MasterStaticFields` | ✅ catalogued |
| `master` | Company | al.master | `MasterStaticFields` | ✅ catalogued · **C enabled** |
| `master` | Branch | al.master | `MasterStaticFields` | ✅ catalogued · **C enabled** |
| `master` | Unit | al.master | `MasterStaticFields` | ⬜ pending the unit scope dimension |
| `auth` | User | al.auth | — | ⬜ identity half only; grants are D |
| `carbon` | biomethane-calculation | al.carbon | `CarbonStaticFields` | ✅ catalogued |
| `carbon` | digestate-entry | al.carbon | `CarbonStaticFields` | ✅ catalogued |
| `carbon` | Batch, Feedstock | al.carbon | `CarbonStaticFields` | ⬜ entities not built yet |
| `ingestion` | ImportTemplate | al.ingestion | `IngestionStaticFields` | ✅ catalogued |
| `ingestion` | ActivityData | al.ingestion | `IngestionStaticFields` | ⬜ scheduled |
| `offset` | Measure, Plan | al.offset | — | ⬜ measures are **F**, not fields |

"C enabled" means the entity implements `IUdfExtensible` **and** its `DbContext` maps
`custom_fields` — both, or custom values have nowhere to land.

---

## What getting it wrong costs

Each of these is a real property of the system today, not a hypothetical.

| Mistake | What it costs |
|---|---|
| C where B belonged | An unindexed `->>` in every report that touches it, against a key a tenant can rename |
| C where D belonged | A permission or a calculation input that a tenant administrator can edit at 02:00 |
| C where E belonged | A threshold that cannot be dry-run, monitored, or replayed |
| C where F belonged | An audit that cannot say which version of a factor produced a figure |
| A (no catalogue) | Every label change is a release; the field cannot gate a condition |
| Deleting a definition instead of retiring it | Values stay in the bag as unrecognised keys and the record cannot be saved again |
| Renaming a `FieldKey` | Every value written under the old key is orphaned, silently |
| **Hiding** a placement to withdraw a field | Same as deleting, and it looks harmless. Fixed — a hidden field is now known-but-ungoverned — but the instinct is still wrong: use `isRetired` |

---

## How a service that holds values but not definitions validates them

Only al.udf is wired to the document store. Every other service sends the payload to
`POST /api/v1/udf/validate/{module}/{resource}` before writing, and stamps the returned
`currentVersions` onto newly-captured values.

Enforcement does not rely on remembering to do this. A `SaveChangesInterceptor` refuses to persist a
modified `CustomFields` that was not validated during the request, matched **by content** — so
validating one payload and persisting a different one is refused too. The rule was previously stated
in three READMEs and followed in none; it is now enforced by the thing that performs the write.

---

## Related

| Document | Owns |
|---|---|
| [`al.net.udf/README.md`](../../al.net.packages/al.net.udf/README.md) | How the mechanism works — scope resolution, `LocalizedText`, conditions, cost model |
| [`al.udf/README.md`](../../Carbon-Atlas-Services/al.udf/README.md) | The service: endpoints, seeding, platform behaviour |
| [`packages/react/src/forms/udf/README.md`](../packages/react/src/forms/udf/README.md) | Client rendering, type mapping, and why the client decides nothing |
| [ARCHITECTURE.md](./ARCHITECTURE.md) | Relational versus document — the store question, one level up from this one |
| [INTEGRATION_STANDARD.md](./INTEGRATION_STANDARD.md) | How a screen talks to a service |
