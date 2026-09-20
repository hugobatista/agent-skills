---
name: saas-transition-strategy
description: >
  Decision framework for projects that start self-hosted and evolve into SaaS.
  Encodes invariants, a decision tree, and strategic anti-patterns to guide
  every architecture choice. Load alongside the saas-anti-pattern-review skill.
  Triggered by: saas transition strategy, hybrid strategy, hybrid review,
  self-hosted strategy.
author: hugobatista
---

# SaaS Transition Strategy: Self-Hosted → SaaS

This skill encodes a decision framework for projects that begin as self-hosted and later add a managed hosting / SaaS offering.

**Audience:** LLMs guiding refactoring decisions alongside the team.
**Purpose:** Quick compass for every architecture choice — does it help or harm the hybrid model?

## North Star

Single product. Two deployment modes. Same build artifact.

- Self-hosted is first-class, never second-class. It is not a "free tier."
- SaaS infrastructure is additive and always optional.
- Every refactoring must benefit both models, or at minimum leave self-hosted untouched.
- The same build artifact runs both modes. No separate build, no separate branch.

## Ground Truths

Each project may define its own ground truths in `AGENTS.md` at the project root, under a `## Ground Truths` section. If the section exists, the agent uses those values. If missing, the agent uses these defaults:

```markdown
## Ground Truths

| Topic | Current state |
|---|---|
| Deployment | `docker compose up`, 3 containers (app + postgres + redis) |
| Infrastructure dependencies | Postgres required, Redis optional (in-memory fallback) |
| SaaS model | Per-organization dedicated stacks → shared multi-tenant later. An organization is a group of users sharing one isolated stack (one tenant unit). Evolve to shared multi-tenant only when needed. |
| First offering | Pure managed hosting, no premium features |
| Phase 1→2 trigger | Multi-worker is safe (durable queues, shared state) |
| Data portability | Users migrate freely between self-hosted and hosted |
| Scaling philosophy | Don't over-engineer. Handle capacity when it arrives. |
```

At the end of a review that used defaults, advise the user to add a `## Ground Truths` section to `AGENTS.md` if the defaults don't match their project. The format is a markdown table with the columns `Topic` and `Current state`.

## Invariants

Non-negotiable. If a change breaks one, it doesn't go in.

1. **The local deploy works.** The standard run command must always start and run without errors.
2. **No regression for self-hosted.** Existing features remain. No feature loss, no performance degradation, no added deploy complexity.
3. **Nothing requires SaaS-only infra.** Every external dependency (Redis, S3, queues, object storage) must have a documented fallback — in-memory, local filesystem, no-op.
4. **Same artifact, both modes.** The published build artifact runs self-hosted and managed hosting identically. Configuration determines the mode.
5. **SaaS features are additive.** Every SaaS feature must work with the self-hosted deploy (possibly degraded or no-op). No self-hosted user sees "this requires a subscription."

## Decision Tree

Facing a change? Run this:

```
Q1: Does this change affect both models the same way?
  → YES: Proceed. It's core. No gating needed.
  → NO (SaaS-only or SaaS-different): Go to Q2.

Q2: Can this be optional infrastructure with a fallback?
  → YES: Build with self-hosted fallback (memory://, DB, local FS, no-op).
  → NO (requires SaaS infra with no reasonable fallback): Go to Q3.

Q3: Is this actually necessary for the first SaaS offering?
  → YES: Gate it behind configuration. Keep it in separate modules
         that core never imports. Self-hosted defaults to disabled.
  → NO: Don't build it. Defer until a customer needs it.
```

Default to **smaller, simpler, optional**.

## Strategic Anti-Patterns

These are thinking traps — not code issues (those are covered by the `saas-anti-pattern-review` skill).

| Anti-pattern | Why it's dangerous |
|---|---|
| "Let's make {SaaS infra} required — simpler code" | Self-hosted must never require SaaS infra. Code can handle optional dependencies. |
| "We'll fix the self-hosted path later" | Later never comes. Ship both paths at once or don't ship. |
| "This SaaS feature behind a feature flag is harmless" | Flags hide complexity, they don't remove it. Prefer separate modules core never imports. |
| "Build it for the scale we'll have in 2 years" | You don't know the scale yet. Build for now. Capacity answers "we'll handle it." |
| "Self-hosted users can live without this" | Self-hosted isn't a free tier — it's the product. Every loss is a loss. |
| "We can add the fallback later" | Fallbacks need to be designed, not bolted on. Design both paths from day one. |

## Phase Roadmap (Template)

Phases are triggered by milestones, not dates.

### Phase 1 — Ops Readiness

**Trigger:** Decision to proceed with managed hosting pre-work.
**Goal:** Make the app safe to run horizontally, without changing the self-hosted experience.

**What changes:**
- Durable job queue (with fallback for single-worker self-hosted)
- Shared state abstraction for multi-worker (pub/sub, auth state)
- Object storage abstraction (local fs default, S3-compatible opt-in)
- DB pool tuning for multi-worker safety
- Multi-worker safety: scheduler, rate limits, auth state, provider sync

**What does NOT change:**
- The self-hosted deploy
- Feature set for self-hosted users
- The artifact structure

**Self-hosted impact:** Zero.

### Phase 2 — Managed Hosting MVP

**Trigger:** Phase 1 is stable. Multi-worker is safe.
**Goal:** Ship managed hosting as a service.

**What changes:**
- Instance provisioning scripts (one isolated stack per organization)
- Admin tooling (list instances, health checks, restart)
- Billing integration
- Domain/subdomain routing per organization
- Data import tool (self-hosted → hosted migration)

**What does NOT change:**
- Core product code
- Self-hosted features
- No premium feature gating

### Phase 3 — Maturity

**Open-ended.** Only when there's a concrete need:

- Fleet observability and monitoring
- Possible multi-tenant consolidation
- Possible premium feature evaluation (must not degrade self-hosted)
- Mobile strategy

## Open Decisions

Tracked for future resolution, not blocking anything now:

| Topic | Status |
|---|---|
| {decision} | {TBD / Deferred / Not yet evaluated} |

## Related Skills

- `saas-anti-pattern-review` — technical checklist for SaaS-unfriendly patterns
