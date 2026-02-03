# Third-Party Data Integration: Current Problems & Recommended Approach

## Current Approach: Ad-Hoc Stateful Partial Sync

Webhooks from third-party services (Stripe, ODX, Medplum) trigger stateful, non-idempotent code that validates, transforms, and writes partial data directly to normalized database tables—executing business logic and side effects immediately.

### Problems

- **Stateful complexity** — Sync logic must track "have I seen this before?" leading to branching code paths for insert vs update
- **Non-idempotent writes** — Re-processing the same event can corrupt data or cause duplicate side effects
- **Partial sync** — Only syncs specific fields the handler knows about; new features require new sync code
- **Data corruption is permanent** — Bugs during ad-hoc handling corrupt data; fixing requires migrations and re-fetching from the source
- **Schema coupling** — Third-party schema changes require code updates, migrations, and redeployment
- **Uptime dependency** — If our server is down during a webhook, that data is lost forever
- **State drift between services** — Updates in one service don't reflect in others if handlers aren't wired correctly; drift accumulates silently over time
- **Hard to retry** — No clean way to replay or recover from errors without risking corruption
- **Scattered logic** — Business rules spread across ad-hoc handlers make the system hard to reason about

---

## Recommended Approach: Idempotent Stateless Full Sync

### Core Idea

1. **Stateless sync jobs** — Cron jobs (or webhook-triggered) that perform full, idempotent syncs without tracking previous state
2. **Full data capture** — Store complete raw third-party entities as JSONB (`stripe_entities_cache`, `odx_entities_cache`, etc.)
3. **Query-time derivation** — Derive canonical state ("is user paid?", "latest lab results") from cache tables at read time
4. **Webhooks as hints** — Webhooks trigger sync jobs but contain no business logic; they're enhancements, not critical paths

### Why This Is Better

| Ad-Hoc Stateful Partial Sync (Current) | Idempotent Stateless Full Sync (Proposed) |
|----------------------------------------|-------------------------------------------|
| Bug = corrupted data | Bug = fix the query, data intact |
| Stateful branching (insert vs update) | Stateless upsert, always safe to re-run |
| Partial sync = new feature needs new sync code | Full sync = data already available |
| Non-idempotent = can't safely retry | Idempotent = retry anytime, same result |
| Schema change = migration | Schema change = transparent (raw JSONB) |
| Webhook fails = data lost | Cron catches it next run |
| State drift accumulates silently | Drift auto-corrected on next sync |
| Complex multi-table writes | Single JSONB upsert |
| Side effects inline (non-idempotent) | Side effects from diff (replayable) |
| Local dev needs ngrok/tunnels | Just run the sync job locally |
| Swapping services = rewrite + migration | Swapping services = new cache table + query differently |
| New feature = new integration work | New feature = just write a query |
| Thousands of lines of handler code | One simple sync function per service |
| High complexity: state checks, error recovery, partial failures | Low complexity: fetch all, upsert, done |
| Duplicated logic across handlers | Single reusable sync pattern |
| Hard to understand: scattered across files | Easy to understand: one sync, queries derive state |

### Cross-Service Sync

Each entity type has a **single source of truth**. Idempotent sync jobs replicate to dependent services:

```
Medplum (source of truth for patients)
    ↓ idempotent sync job
    ├── Upsert user in Postgres
    ├── Upsert patient in ODX
    └── Upsert customer in Stripe

ODX (source of truth for lab results)
    ↓ idempotent sync job
    ├── Upsert to odx_entities_cache
    └── Diff state → queue side effects (PDF generation, Slack alerts)
```

Adding a new service? Just extend the sync job—no ad-hoc webhook rewiring needed.

### Example Flow

1. ODX webhook arrives → triggers idempotent sync job for that patient (webhook contains no logic)
2. Job fetches full data from ODX, upserts to `odx_entities_cache` (stateless, idempotent)
3. Job diffs before/after state → queues side effects (PDF generation, Slack alerts)
4. Cron runs hourly as fallback to catch any missed webhooks (self-healing)

---

## Performance & Scaling

### Storage Estimates (Stripe)

| Entity | Per User | JSON Size | 2k Users | 100k Users |
|--------|----------|-----------|----------|------------|
| Customer | 1 | ~3KB | 6MB | 300MB |
| Subscription | 1-2 | ~2KB | 8MB | 400MB |
| Payment Method | 1-2 | ~1KB | 4MB | 200MB |
| Invoice | ~10 | ~3KB | 60MB | 3GB |
| Payment Intent | ~10 | ~2KB | 40MB | 2GB |
| **Total** | | | **~120MB** | **~6GB** |

### Sync Time (Stripe API: 100 req/s, 100 objects/page)

| Users | API Requests | Full Sync Time | Recommended Frequency |
|-------|--------------|----------------|----------------------|
| 2k | ~100 | ~1 second | Every minute |
| 10k | ~500 | ~5 seconds | Every 5 minutes |
| 50k | ~2,500 | ~25 seconds | Every 15 minutes |
| 100k | ~5,000 | ~1 minute | Every 30 minutes |

**At 2k users, you can run a full idempotent sync every minute.** The pattern scales to 100k+ users before needing optimization.

When you outgrow naive full sync:
- Use `created[gte]` filters for incremental sync
- Webhooks become primary trigger, cron becomes weekly full verification
- Parallel workers for higher throughput

---

## Declarative Webhook Management

Manage webhook configuration as code, not through UIs.

| Ad-Hoc UI Configuration | Declarative Setup Scripts |
|-------------------------|---------------------------|
| No version control, no audit trail | Config in git, changes tracked |
| Environment drift (dev ≠ staging ≠ prod) | Same script = same result everywhere |
| New environment = manual clicking | `pnpm run sync:webhooks` |
| Local dev = can't test webhooks | Local dev = tunnel + script, real webhooks locally |

**How it works:** Define webhooks in code, run an idempotent sync script (delete all → create desired) in CI/CD on every deploy. Zero drift, instant new environment setup, easy local testing with tunnels.

---

## Migration Path

The idempotent full sync can run **in parallel** with the existing ad-hoc implementation—no big bang migration required.

### Step 1: Add New Infrastructure (No Breaking Changes)

- Add cache tables (`stripe_entities_cache`, `odx_entities_cache`, etc.)
- Add new webhook handlers that just trigger sync jobs (alongside existing ad-hoc handlers)
- Add idempotent cron jobs to sync full data into cache tables
- Add new query functions/views that read from cache tables

### Step 2: Run Both Systems in Parallel

```
Existing ad-hoc handler  → writes to normalized tables (unchanged)
New stateless handler    → triggers idempotent sync job
Cron job                 → runs full idempotent sync into cache tables
```

Both systems populate their respective tables. Old code keeps working.

### Step 3: Feature Flag at API Level

```typescript
async getSubscriptionTier(userId: string) {
  if (featureFlags.useNewDataLayer) {
    return this.queryFromCache(userId);  // New: stateless query from full data
  }
  return this.queryFromNormalizedTables(userId);  // Old: ad-hoc partial data
}
```

### Step 4: Gradual Rollover

- Enable flag for internal users → validate
- Enable for 10% of users → monitor
- Enable for 100% → confirm stability
- **Instant rollback**: just flip the flag

### Step 5: Cleanup (Once Confident)

- Remove old ad-hoc webhook handlers
- Remove old normalized tables
- Remove feature flags
- Delete thousands of lines of stateful, non-idempotent code

### Why This Works

- **Zero downtime** — new idempotent code runs alongside old ad-hoc code
- **Instant rollback** — feature flag, not a migration
- **Validate before committing** — compare old vs new results
- **No data loss risk** — old tables stay until you're confident

---

**TL;DR:** Move from ad-hoc stateful partial sync to idempotent stateless full sync. Store facts (raw data), derive interpretations (queries). Webhooks become hints, not critical paths. Pick a source of truth for each entity, replicate with idempotent jobs. Integrations become simpler, bugs are fixable without migrations, and the system self-heals.
