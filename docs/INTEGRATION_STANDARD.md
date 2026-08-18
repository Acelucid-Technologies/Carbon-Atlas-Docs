# Integrating a microservice

How every screen in this app talks to the backend. **Follow this for new work**; the UDF modules in
`apps/web/src/api` are the worked reference.

> This exists because the app was once written against an API nobody had built. Of the sixteen paths
> it called, **four matched a real endpoint** — and nothing revealed that, because the client pointed
> at a port where nothing listens, so every call failed identically regardless of cause. The rules
> below each close one way that happened again.

---

## 1. There is no gateway. Name your service.

Carbon Atlas is twelve independently deployed services. Locally they are twelve ports; deployed they
sit behind one host. **A call must say which one it means.**

```ts
const plans = await httpRaw<Plan[]>('/api/v1/subscriptions/plans', { service: 'subscription' })
```

| `VITE_API_BASE_URL` | Behaviour |
|---|---|
| empty | Each service resolves to its own `localhost` port, 7001–7012 |
| set | Every service resolves to that host, path-routed |

Call sites are identical in both. **Never write a port** into a feature module.

## 2. Paths are absolute and live in `routes.ts`

Write `/api/v1/auth/login`, not `/auth/login`. The path is the real server path — paste-able into
Swagger, greppable in the service that owns it.

Every path goes in [`apps/web/src/api/routes.ts`](apps/web/src/api/routes.ts), grouped by service.
Paths scattered across a dozen call sites cannot be audited against a backend; collected, the file
diffs against `al.master/Data/PermissionCatalogSeeder.cs` — the platform's own record of what exists.

**Before adding one, check it against that seeder.** A route with no rule is *denied*, so an endpoint
the catalogue does not know returns 403 rather than 404 — a confusing way to learn you guessed.

## 3. If it has no backend, say so

Anything the app calls that no endpoint serves goes under `UNIMPLEMENTED` in `routes.ts`, with what
it would take. The feature can run on mock data — that is fine, and it is honest **only because it is
written down.** A mock nobody recorded is indistinguishable from a working feature until a demo.

## 4. Adapt wire shapes; never cast them

Declare what the service actually returns, separately from the app's own type, and map between them
in one function.

```ts
interface TokenResponse { accessToken: string; expiresInSeconds: number /* … */ }

function toLoginResponse(tokens: TokenResponse, profile: UserProfileResponse): LoginResponse { … }
```

When the backend changes, exactly one place fails to compile. A cast moves that failure to runtime,
inside a component.

Expect the shapes to differ in ways that look like mistakes and are not. Login returns tokens and
**no user** — issuing a token is not reading a profile, and folding the profile in would re-serialise
it on every refresh. Signing in is therefore two calls.

## 5. Cache keys live in `queryKeys.ts`

A key is a contract between a query and every mutation that invalidates it. Inline, that contract is
a repeated string literal, and when the two drift the failure is not an error — it is a screen
showing data that was correct a minute ago, which the developer never sees and the user reports as
"it didn't save".

Keys are hierarchical, broad to narrow, so a partial key invalidates everything beneath it:

```ts
udfKeys.forms()                              // every resolved form
udfKeys.form('carbon', 'Facility', 'biomethane')   // one form, one industry
```

**Every parameter that changes the response belongs in the key.** The scope one catches people:
without `industryKey`, moving between two lines at the same branch serves the first one's fields from
cache and renders a form the server will refuse.

## 6. Staleness comes from `STALE`, by what the data *is*

| | |
|---|---|
| `STALE.vocabulary` | Platform reference data. Changes on a release. |
| `STALE.configuration` | Tenant configuration. Changes when an administrator edits it, never mid-session. |
| `STALE.operational` | Data the user is actively working with. |

Not per screen. Two screens reading one endpoint with different staleness is a bug that presents as
an intermittent one.

## 7. Invalidate what you affected — broadly when you cannot enumerate

```ts
onSuccess: () => {
    queryClient.invalidateQueries({ queryKey: udfKeys.forms() })   // can't enumerate
    queryClient.invalidateQueries({ queryKey: udfKeys.definitions() })
}
```

A definition change can affect forms the mutation has no way to list — the field may be placed on
several, at several scopes. Enumerating them would mean reconstructing the server's resolution on the
client, which is exactly what rule 9 forbids. Drop the broad key; it costs one refetch of what is on
screen.

## 8. Never send the tenant

Tenant, company, branch and application come from the **token**. The client does not send them and
must not be trusted to: a caller able to name its own tenant can read another customer's data. That
was a real defect in the UDF API — `GET /definitions/{tenantId}` read the tenant off the path — and
it is why no request type in this app has a `tenantId` field.

Parameters that describe *what is being asked for* rather than *who is asking* are fine.
`industryKey` is one: a branch runs several industry lines, and which one a form belongs to is a
property of the form, not the session.

## 9. The server resolves; the client renders

Where the backend applies precedence — most-specific-wins across tenant, company, branch, application
and industry — **do not re-implement it.** A second implementation eventually disagrees with the
first, and the disagreement surfaces as a field that renders and then fails validation.

What arrives is already resolved and already ordered. Draw it.

## 10. Map server validation onto fields

The client schema saves a round trip. **The server decides.** For custom fields it is the only
integrity there is — those values land in a JSON column with no foreign keys, check constraints or
unique indexes.

```tsx
try {
    await save(values)
} catch (error) {
    if (!applyServerErrors(error, setError)) throw error
}
```

`ValidationException` returns every failure keyed by field, and the field key is the control's
`name`, so it is a lookup and not a translation table. Use `unmatchedServerErrors` for keys this
bundle does not know — a form one release behind its configuration will hit that, and without
surfacing it the message vanishes and the save button appears to do nothing.

## 11. Token refresh is the client's, not yours

`configureApiClient` handles it: proactive before each request, reactive on a 401 with one replay,
and concurrent 401s share a single in-flight refresh. **Do not add retry logic to a feature module.**

Refresh tokens are single-use and rotate, so six parallel refreshes get five *reuse* responses — and
reuse detection correctly revokes every session the user has.

---

## The shape of a new integration

```
routes.ts          add the path, grouped by service, checked against the seeder
queryKeys.ts       add the key hierarchy
{feature}.api.ts   wire types, calls, adapters to app types
use{Feature}.ts    hooks: queries with STALE, mutations with invalidation
```

### Checklist

- [ ] Path copied from the backend, not invented, and present in the permission catalogue
- [ ] `service` named on every call
- [ ] Wire type declared separately from the app type, with one adapter between them
- [ ] Key in `queryKeys.ts`, including every parameter that changes the response
- [ ] Staleness from `STALE`
- [ ] Mutations invalidate what they affect
- [ ] No `tenantId` in any request
- [ ] No precedence logic on the client
- [ ] Server validation reaches the fields
- [ ] Anything unbacked recorded in `UNIMPLEMENTED`

---

## Reference

| | |
|---|---|
| Routing and refresh | [`packages/api-client/README.md`](packages/api-client/README.md) |
| This app's API layer | [`apps/web/src/api/README.md`](apps/web/src/api/README.md) |
| Rendering resolved fields | [`packages/react/src/forms/udf/README.md`](packages/react/src/forms/udf/README.md) |
| What UDF is, and why values are `jsonb` | `al.net.packages/al.net.udf/README.md` |
| The service | `Carbon-Atlas-Services/al.udf/README.md` |
