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
