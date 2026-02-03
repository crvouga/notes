# Geviti Rewrite Entire System

Migrate from [geviti-web-app](https://github.com/GoGeviti/geviti-web-app) to [geviti](https://github.com/GoGeviti/geviti) while maintaining service continuity.

**Problems to solve:** Third-party integration bugs, multi-frontend support (web + Flutter), CI/CD gaps, competing standards.

## Execution Strategy

1. **Feature freeze** on old app (bug fixes only)
2. **Shared data sources** (same Postgres, Medplum, Stripe—no data migration)
3. **Database ownership** migrated to new repo (copy all migration history, new repo becomes DB authority)
4. **Parallel deployment** with bidirectional user switching
5. **Gradual rollout:** internal team → external beta → full migration

## Safety Requirements

- [ ] All DB migrations migrated to new repo
- [ ] New repo is DB authority (old repo no longer modifies schema)
- [ ] CI/CD pipeline operational
- [ ] Automated tests on every PR
- [ ] Linting/formatting enforced
- [ ] Staging environment + feature flags
- [ ] Documented coding standards
- [ ] Auto-rollback triggers defined

## Risk Mitigation

| Risk              | Mitigation                           |
| ----------------- | ------------------------------------ |
| Schema conflicts  | Additive-only changes, feature flags |
| Feature gaps      | Parity checklist, weekly tracking    |
| Scope creep       | No new features until parity         |
| Production issues | Auto-rollback triggers               |
