# Fix existing system

## Links

[Repo](https://github.com/GoGeviti/geviti-web-app)

## Problems we're having

- Bugs
- Third party integration bugs
- Third party integration coupling
- Lack of flexibility
- Supporting multiple frontends between web app and flutter
- Competing standards
- Lack of documentation

### Secondary problems

- Messy database
- Messy code

## Solutions

- Move to a monorepo
- Rewrite parts of the system not the entire system.
- Delete old unused code
- Identify and rewrite the buggy parts of the system. Mainly the third party integrations
- Fix the causes not the symptoms
- Have a good CI pipeline
- Remove unused code
- Add documentation
- Clean up messy database
- Clean up messy code

## Why we shouldn't rewrite the entire system

### Business Continuity

- **No feature freeze required** — business can keep shipping new features
- **No parallel maintenance burden** — not maintaining two codebases simultaneously
- **No migration risk** — users never have to switch apps or deal with bugs in a new system
- **Revenue stays uninterrupted** — no risk of breaking payments, subscriptions, or core flows

### Effort & Time

- **Rewrites always take longer than expected** — the "90% done, 90% to go" problem
- **Hidden complexity emerges late** — edge cases, integrations, and business logic that nobody remembers
- **Feature parity is a moving target** — old app keeps getting fixes while new app plays catch-up
- **You're solving the same problems twice** — bugs fixed in old app must be anticipated in new app

### Risk

- **Rewrites fail more often than they succeed** — many companies have killed themselves with rewrites (Netscape, etc.)
- **Second-system effect** — tendency to over-engineer the new system
- **Institutional knowledge gets lost** — code comments, workarounds, and "why did we do this?" context disappears
- **Database migration handoff is complex** — even with shared data, transferring schema authority is risky

### The Problems May Follow

- **Bad patterns will creep back in** — without discipline, the new codebase degrades the same way
- **Third-party integrations are still hard** — Stripe/Medplum/ODX complexity doesn't disappear with a rewrite
- **Team still needs to learn better practices** — rewriting doesn't teach the team; fixing does

### Alternative: Incremental Improvement

The problems we have (bugs, integration issues, competing standards) can all be fixed incrementally:

| Problem                      | Fix Without Rewrite                           |
| ---------------------------- | --------------------------------------------- |
| Third-party integration bugs | Rewrite just that module                      |
| No CI/CD                     | Add it to existing repo                       |
| Competing standards          | Adopt one, lint for it, migrate incrementally |
| Messy code                   | Refactor file-by-file                         |
| Multi-frontend support       | Extract shared API layer                      |

**The Strangler Fig pattern** — gradually replace bad parts while keeping the system running. No big bang, no feature freeze, no risk of total failure.
