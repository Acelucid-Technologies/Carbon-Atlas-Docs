# Identity & Scope

How one person belongs to many tenants, many companies and many branches — and how "this user may
use the web application but not the mobile one" is enforced rather than merely displayed.

← [Deliverables.md](./Deliverables.md)

---

## The question

> A user can belong to multiple tenants; within a tenant to multiple companies; and can have access
> to multiple branches/locations within a company. And: a user may be enabled for the Web
> application but not the Mobile app.

Both halves have the same answer, and it is the one thing to take from this document:

> **Identity is one row. Access is many rows. The token names one active context.**

Everything below follows from that.

---

## Three concepts people conflate

Most of the ways this goes wrong come from merging two of these.

| | What it is | Where it lives | How many per person |
|---|---|---|---|
| **Identity** | Who somebody is, and how they prove it | `al.auth` — `AuthUser` | Exactly one, globally |
| **Membership (grant)** | A slice of the platform they may act in, and as what | `al.master` — grant rows | Many |
| **Active context** | The single slice they are acting in *right now* | The signed token | One at a time |

**Identity is global, deliberately.** One person, one password, one MFA enrolment, one lockout
counter. Making identity per tenant means the same human has four passwords, four reset flows and
four sets of failed-login counters — and a lockout in one tenant tells the attacker nothing about
the others, which sounds like isolation and is actually just four weaker accounts.

**Access is a set of grants, not a field on the user.** The moment access is a `TenantId` column on
the user, a second tenant is a schema change. It is a join table because the relationship is
many-to-many, and pretending otherwise is the thing that has to be undone later.

**The token names one context.** Never a token that spans tenants — see
[Why the token is single-context](#why-the-token-is-single-context).

---

## The grant

One row says: *this person, in this slice, as this role, from these applications, for this period.*

```
Grant
├─ userId          required   the identity
├─ tenantId        required   the only mandatory scope — no grant floats above a tenant
├─ companyId       null = every company in the tenant
├─ branchId        null = every branch in the company
├─ industryKey     null = every industry line at the branch
├─ unitId          null = every unit in the industry line
├─ applicationId   null = every application            ← the web/mobile answer
├─ roleId          required   what they may do in that slice
├─ includesDescendants        for a parent tenant, reaches sub-tenants
├─ grantType       Standard | Support | Service
├─ validFrom / validUntil     null = open-ended
└─ status          Active | Suspended | Revoked
```

**Null means "everything below", not "nothing".** That is what makes the common cases one row:

| Requirement | Rows |
|---|---|
| Administers a whole tenant | 1 — everything below tenant null |
| Administers two companies in one tenant | 2 — one per company |
| Reads three branches of one company | 3 — one per branch |
| Belongs to four tenants | 4 — one per tenant |
| Web only | the grant carries `applicationId = web` |
| Web everywhere, mobile at one branch | 2 — a web grant at tenant level, a mobile grant at branch level |

The four Ace users are exactly this, one row each:

```
AceTenantAdmin   tenant=ace                                                → the whole tenant
AceAdmin         tenant=ace  company=islabhasolutions                      → every branch of it
Santiago_Ace     tenant=ace  company=islabhasolutions  branch=spain        → every industry there
Amar_Ace         tenant=ace  company=islabhasolutions  branch=spain  industry=biomethane
```

### Why validity dates are on the grant and not implied

A contractor with access "until the end of the project" is otherwise a calendar reminder somebody
forgets. An expiry the resolver enforces is the difference between access that ends and access that
is *supposed* to end.

---

## Application gating — the "web but not mobile" requirement

This is the part most systems get wrong, and the failure is silent.

**The wrong way, twice over:**

| | Why it fails |
|---|---|
| Hide the feature in the mobile UI | The API does not know. Anyone with the token and `curl` has full access. It is a cosmetic control described as a security one. |
| Put the application on the **role** | The role means the same thing in both applications — a Branch Admin administers a branch either way. What differs is whether *this person* may use *that application*. Putting it on the role forces a `BranchAdminWeb` and a `BranchAdminMobile` that must be kept identical forever, and one day are not. |

**The right way: application is a property of the grant, and it is checked at three points.**

```
1. Token issuance      login and context-switch REFUSE to issue a token for an application
                       the user holds no grant in. There is no token to misuse.

2. The token itself    carries an `app` claim naming the single active application.

3. Route authorization every request checks the `app` claim against the route's allowed
                       applications, and the effective-access resolver only considers grants
                       whose applicationId is null or matches.
```

Point 1 is the one that matters. **A user with no mobile grant never receives a mobile token**, so
there is nothing to hide in the mobile UI and nothing to leak through the API. The mobile app shows
a clear refusal — *"Your account is not enabled for the mobile app"* — rather than an empty screen
or a generic 403, because the user has done nothing wrong and needs to know who to ask.

Point 3 exists because grants change while a token is alive. A token issued this morning says
nothing about a grant revoked at lunchtime, which is why the resolver re-reads rather than trusting
the claim for anything but the application identity.

### Why not simply a per-user flag

`user.canUseMobile` is one boolean and it is wrong the first time somebody needs mobile access *at
one branch only* — a field engineer who uses the app on site and the console at head office. On the
grant it is free; as a user flag it is a redesign.

---

## Effective access — the rule that is easy to get backwards

Given a user and an active context, resolution runs over **the grants that apply to that context**
(scope matches, application matches, dates valid, status active). Then:

> **Permissions union. Configuration resolves most-specific-wins. Denials beat both.**

That split catches people, so it is worth being explicit.

| | Rule | Why |
|---|---|---|
| **Permissions** | **Union** every applicable grant | Somebody with a read role at company level and an admin role at one branch should be an admin at that branch *and* still a reader elsewhere. Most-specific-wins would silently take the read access away. |
| **Configuration** — quota, providers, labels, preferences | **Most specific wins, outright** | A branch that sets a 10 GB limit must beat the company's 100 GB, or a narrower scope could never be stricter. Union is meaningless here: two values, one has to win. |
| **Explicit denial** | **Wins over everything** | An explicit "not this" exists precisely to carve an exception out of a broad grant. If a union could re-grant it, it would not be a denial. |

Merging is never an answer for configuration. A value assembled from whichever properties each level
happened to set is unexplainable to the person who configured it — and "why is this 40 GB when I set
30" is a support ticket nobody can close.

**Resolution is server-side, always.** The client renders what it is given. A second implementation
of precedence in the browser eventually disagrees with the server, and the disagreement shows up as
a control that renders and then 403s — which reads as a broken permission system rather than as two
implementations drifting.

---

## Why the token is single-context

A token could carry every grant. It should not.

| | |
|---|---|
| **Size** | A user in forty branches carries forty grants in every request header, on every call, forever. |
| **Revocation** | A token listing grants is stale the moment one is revoked. A token naming one context lets the resolver re-read the grants each request. |
| **Query isolation** | The row-level filter needs *one* tenant. Given a set, every query needs an explicit choice, and the day somebody forgets is a cross-tenant read. One active tenant makes the safe path the default path. |
| **Auditability** | "What was this person acting as when they did that" has one answer instead of a set. |

So: **the token names one tenant, and optionally one company, branch, industry and unit.** Switching
is an explicit act — `POST /auth/context/switch` — that re-checks access and issues a **new token
and a new session**. The tenant is a signed claim; there is no way to change it on a token that is
already signed, and anything that appeared to would be reading the tenant from somewhere other than
the signature.

### The cache is the dangerous part of switching

Reissuing the token is easy. The trap is that **everything already cached in the browser was fetched
under the previous tenant**, and a query cache will happily serve it under the new tenant's heading
until it goes stale.

That is a cross-tenant leak in the UI requiring no server bug, no missing check and no attacker — it
is simply what a cache does. The switch therefore **clears** the cache rather than invalidating it:
invalidation leaves entries mounted and readable while the refetch is in flight, so the previous
tenant's rows stay on screen for as long as the network takes.

Already implemented in `useSwitchContext`.

---

## Why branch and scope path travel as headers

The token carries tenant, application and company as signed claims. **Branch and scope path travel as
`x-branch-id` and `x-scope-path` headers.**

**Not because the token cannot carry them.** It is worth being precise, because the gap is smaller
than it looks:

| | |
|---|---|
| `TokenClaims.BranchId` | **exists**, and `EcdsaTokenService` already emits `["branchId"]` into the payload |
| `ApiContextMiddleware` | already reads the `branchId` claim, **claim-first** with the header as fallback |
| `TokenClaims.Extra` | merges arbitrary claims, so `scopePath` needs no new property either |
| `ITokenService.IssuePair(...)` | **has no `branchId` parameter** — this, and only this, is why the claim is never populated |

So closing it is one parameter on `IssuePair`, passed through to the `TokenClaims` it already builds.
`ITokenService.Issue(TokenClaims, ttl)` would even allow al.auth to do it today with no SDK change —
but that means rebuilding pair issuance (session id, refresh TTL, the access/refresh pairing) outside
the SDK, which is precisely the second implementation that drifts. Not worth it.

**Decided 2026-08-16: headers stay for now.** Not because a claim is hard, but because the header is
already safe — and the reason it is safe is worth writing down, because it is not obvious.

It looks alarming at first, because those headers do real work. `MultiTenantDbContext` reads
`context.BranchId` for both halves of isolation:

```
CurrentBranchId = () => _apiContextAccessor.Context?.BranchId      filters every read
entry.Entity.BranchId = context.BranchId                           stamps every write
```

So a caller-supplied header decides which rows they see and where new records land. What makes that
safe is that **the same value is what permission resolution compares against**:

```
AppliesTo:  Matches(assignment.BranchId, context.BranchId)
            ScopePath.Parse(assignment.ScopePath).Covers(context.ScopePath)
```

Name a branch your assignment does not cover and `Matches` returns false — you resolve **zero
permissions** and every endpoint refuses you. Name a scope path your assignment does not cover and
the same happens, because coverage is a prefix match: a deeper path narrows, a shallower one drops
the assignment entirely.

> **A header can only select among the assignments a person already holds. It cannot fabricate one.**
> Setting it wrongly locks you out; it never opens anything.

Two things this does not solve, recorded rather than fixed:

| | |
|---|---|
| **The audit trail attests a client-supplied value** | "Which branch was this done in" is asserted by the caller rather than signed. Not an escalation — they had the right either way — but a claim would be stronger evidence. |
| **A stale header is a data-correctness problem** | A tenant-wide administrator whose client sends the wrong `x-branch-id` files records into the wrong branch, and is entitled to. This is why the client must echo what `/auth/login` returned rather than remembering its own value. |

Both are arguments for eventually moving them into claims, not for treating the current design as a
hole. Revisit it when the SDK changes for another reason.

---

## Parent and sub-tenants

A parent tenant may create sub-tenants one level deep — divisions, regions, or a consultancy's
clients.

```
Parent tenant admin    own data + all sub-tenant data, aggregated
Sub-tenant user        own data only — not the parent's, not a sibling's
Standalone tenant      own data only
```

Modelled as a `parentTenantId` on the tenant plus `includesDescendants` on the grant. **Both are
required, and the flag is the important one:** a parent tenant existing does not mean every grant in
it should reach downward. A finance role at the parent may need aggregated visibility; a data-entry
role at the parent should stay at the parent.

**Aggregation is a read, never a write.** A parent admin can see a sub-tenant's totals; acting *in*
a sub-tenant means switching context into it, which produces its own token and its own audit trail.
Blurring that is how an action taken by head office shows up in the record as having been taken by
the subsidiary.

The three directions all need testing, and the sibling one is the one that gets missed: sub-tenant A
must not see sub-tenant B, even though both are visible to the parent.

---

## Support access

Support staff will need to see a customer's data. Two ways to arrange it:

| | |
|---|---|
| A standing grant into every tenant | Invisible in the audit trail — support activity is indistinguishable from the customer's own. |
| **A distinct, time-boxed, audited grant type** ✅ | `grantType = Support`, with `validUntil` required, a reason recorded at creation, every action tagged, and a persistent banner while it is active. |

The second costs a column and an afternoon. The first costs the ability to answer "who looked at
this record" — which is the question asked after an incident, and the one a customer contract
usually requires an answer to.

---

## What this replaces

The platform currently derives access from `TenantAccess` (which tenants a user belongs to) plus
`TenantUserRole` (role assignments carrying company, branch and scope path). Between them they hold
most of this already.

`A6` consolidates them into one grant table, because the split has two costs: the application
dimension has nowhere to live, and "what can this person do" needs two queries and a join that each
caller writes slightly differently.

| Today | After `A6` |
|---|---|
| `TenantAccess` — user ↔ tenant, `IsOwner` | `Grant` with company/branch/industry/unit null |
| `TenantUserRole` — role at company/branch/scope path | `Grant` with those scopes set |
| *(nowhere)* | `applicationId`, `validFrom/Until`, `grantType`, `includesDescendants` |

Migration is mechanical: every `TenantAccess` becomes a tenant-level grant, every `TenantUserRole`
becomes a scoped one, application stays null (meaning "every application", preserving today's
behaviour), and nothing changes for an existing user until an administrator narrows it.

---

## Pitfalls

- **Never send tenant, company, branch or application from the client.** They come from the token.
  This API shipped once with a route that read the tenant from the path; a caller who could name
  their own tenant could read another customer's configuration.
- **Re-check access on every context switch.** The presented token proves the user authenticated,
  not that they still have access to the tenant they are naming.
- **403, not 404, for a tenant they cannot reach.** A 404 is also a way to probe which tenant ids
  exist.
- **Union permissions, do not most-specific-wins them.** The single most likely error in `A8`, and
  it fails closed in a way people report as "my access disappeared".
- **A grant with `applicationId` set does not restrict the role.** It restricts *where the person may
  exercise it*. Reading it the other way produces per-application role duplication.
- **Clear the cache on switch; do not invalidate it.** Invalidated entries stay readable.
- **Test the sibling direction** of sub-tenant isolation. Parent-to-child and child-to-parent get
  tested; child-to-sibling is the one that ships broken.
