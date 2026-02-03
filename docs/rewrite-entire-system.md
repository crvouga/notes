# Rewrite Entire System

Migrate from [geviti-web-app](https://github.com/GoGeviti/geviti-web-app) to [geviti](https://github.com/GoGeviti/geviti) while maintaining service continuity.

## Why Rewrite

- Third-party integration bugs causing data corruption
- No CI/CD or automated testing
- Can't support multiple frontends (web + Flutter)
- Messy codebase with competing standards

## Core Principle: Shared Data

Both apps connect to the **same data sources**—no data migration required:

- Same Postgres database
- Same Medplum FHIR server
- Same Stripe account

This eliminates the riskiest part of most rewrites.

---

## Database Migration Handoff

**Migrations currently live in the old repo.** This is the critical handoff.

### Process

1. **Copy migration history** to new repo (files + lock table state)
2. **Freeze old repo** — remove migration commands from CI/CD, block new migrations
3. **New repo becomes authority** — runs migrations on deploy

### Rules During Transition

- **Additive only** — no drops, no renames (both apps read same tables)
- **Nullable new columns** — old app won't populate them
- **Feature flags** — new repo uses new columns; old repo ignores them

---

## Execution Phases

| Phase             | Goal                  | Key Milestones                                    |
| ----------------- | --------------------- | ------------------------------------------------- |
| **0. Prep**       | Set up infrastructure | CI/CD, staging env, feature parity checklist      |
| **1. Foundation** | Core functionality    | DB migrations copied, auth working, API structure |
| **2. Parity**     | Match old app         | All features rebuilt, tracked via checklist       |
| **3. DB Handoff** | Transfer authority    | Old repo frozen, new repo runs migrations         |
| **4. Parallel**   | Internal testing      | Both apps deployed, team dogfoods new app         |
| **5. Rollout**    | Gradual migration     | Internal → 5% → 25% → 100% of users               |
| **6. Deprecate**  | Shut down old app     | Archive repo, clean up unused tables              |

---

## Safety Requirements

- [ ] All DB migrations in new repo
- [ ] New repo is DB authority
- [ ] CI/CD with automated tests
- [ ] Staging environment operational
- [ ] Feature flags working
- [ ] Rollback procedure tested

---

## Risk Mitigation

| Risk             | Mitigation                                      |
| ---------------- | ----------------------------------------------- |
| Schema conflicts | Additive-only changes, freeze old repo          |
| Feature gaps     | Parity checklist, weekly tracking               |
| Scope creep      | No new features until parity                    |
| Webhook issues   | Run both handlers in parallel during transition |

---

## Rollback Plan

1. **Code**: Revert deployment or flip feature flag
2. **Database**: Run migration rollback in new repo
3. **Full rollback**: Route traffic back to old app via DNS

---

## Success Criteria

- 100% feature parity
- Error rate ≤ old app
- Zero data discrepancies
- Rollback tested and documented

## Why We Should Rewrite

### The Codebase Is Beyond Incremental Fixes

- **Technical debt compounds** — every fix in a messy codebase adds more mess; you're fighting the architecture
- **Competing standards can't be unified incrementally** — you'd have to touch every file anyway
- **The integration layer is fundamentally broken** — ad-hoc stateful webhook handlers can't be patched into idempotent sync jobs
- **No tests means no safe refactoring** — changing old code without tests is just as risky as rewriting

### Incremental Fixes Have Already Failed

- **We've been "fixing" for how long?** — if incremental improvements worked, we wouldn't be having this conversation
- **Same bugs keep coming back** — symptoms of architectural problems, not code problems
- **Team velocity is suffering** — developers spend more time fighting the codebase than building features

### A Rewrite Lets Us Do It Right

- **CI/CD from day one** — not bolted on later
- **Tests as a requirement** — not an afterthought
- **Single standard** — one way to do things, enforced by linting
- **Clean integrations** — idempotent sync pattern instead of ad-hoc webhooks
- **Multi-frontend support built in** — shared API layer designed for web + Flutter

### This Rewrite Is Lower Risk Than Most

| Typical Rewrite Risk         | Why We Don't Have It                              |
| ---------------------------- | ------------------------------------------------- |
| Data migration               | **No migration** — same Postgres, Medplum, Stripe |
| Lost business logic          | Feature parity checklist + old code as reference  |
| Extended downtime            | Parallel deployment, gradual rollout              |
| No rollback                  | Instant rollback via DNS/routing                  |
| Team doesn't know the domain | Same team, same domain knowledge                  |

### The Cost of Not Rewriting

- **Continued data corruption** from broken integrations
- **Continued developer frustration** and potential attrition
- **Continued slow velocity** as codebase gets worse
- **Technical debt interest** — the longer you wait, the harder it gets
- **Flutter app blocked** — can't build multi-frontend support on a broken foundation

### When Incremental Improvement Works vs. When It Doesn't

| Incremental Works                | Rewrite Works                        |
| -------------------------------- | ------------------------------------ |
| Codebase is messy but structured | Codebase has no coherent structure   |
| Problems are localized           | Problems are architectural/systemic  |
| Tests exist for safe refactoring | No tests, refactoring is blind       |
| Team agrees on standards         | Competing standards, no consensus    |
| One frontend                     | Multiple frontends need shared layer |

**Our situation:** No tests, no standards, systemic integration issues, multi-frontend needs. This is a rewrite case.
