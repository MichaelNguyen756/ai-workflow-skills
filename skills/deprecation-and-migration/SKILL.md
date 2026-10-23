---
name: deprecation-and-migration
description: Manages deprecation and migration — treating code as a liability to be removed once it stops earning its keep, and moving consumers safely from an old implementation to a new one. Covers the strangler, adapter, and feature-flag patterns, expand/contract for any in-place rename across a rollout window, database schema migrations, and zombie-code removal. Use when replacing an old system, sunsetting a feature, consolidating duplicate implementations, planning a new system's eventual removal, or migrating consumers across a version boundary.
---

# Deprecation and Migration

## Overview

Code is a liability, not an asset. Every line has ongoing cost — bugs, dependency
updates, security patches, and mental overhead for anyone working nearby. Deprecation is
the discipline of removing code that no longer earns its keep; migration is moving
consumers safely from old to new. Most teams are good at building and bad at removing.
This skill addresses that gap.

## When to Use

- Replacing an old system, API, or library with a new one
- Sunsetting a feature that's no longer needed
- Consolidating duplicate implementations
- Removing dead code nobody owns but something depends on
- Planning a new system's lifecycle (deprecation planning starts at design time)
- Migrating consumers across a version boundary

## Core Principles

### Code is a liability

The value is the functionality, not the code. When the same functionality needs less
code, less complexity, or a better abstraction, the old code should go.

### Hyrum's Law makes removal hard

With enough consumers, every observable behavior gets depended on — including bugs and
undocumented side effects. This is why deprecation requires active migration, not just
announcement.

### Deprecation planning starts at design time

When building something new, ask "how would we remove this in 3 years?" Clean interfaces,
feature flags, and minimal surface area are easier to deprecate than systems that leak
implementation everywhere.

## The Deprecation Decision

```
1. Does this still provide unique value?        → if yes, maintain it
2. How many consumers depend on it?             → quantify the scope
3. Does a replacement exist?                    → if no, build it first
4. What's the migration cost per consumer?      → automate if trivial; weigh if manual
5. What's the cost of NOT deprecating?          → security risk, engineer time, complexity
```

## Compulsory vs Advisory

| Type | When | Mechanism |
|------|------|-----------|
| **Advisory** | Migration optional, old system stable | Warnings, docs, nudges; consumers migrate on their own timeline |
| **Compulsory** | Old system has security issues, blocks progress, or is too costly | Hard deadline + migration tooling |

**Default to advisory.** Compulsory requires providing tooling, docs, and support — you
can't just announce a deadline.

## The Migration Process

### Step 1: Build the replacement

Don't deprecate without a working alternative. It must cover the critical use cases, have
a migration guide, and be proven in production.

### Step 2: Announce and document

A notice with status, replacement, removal date (or "advisory — no deadline"), reason, and
a concrete migration guide.

### Step 3: Migrate incrementally

One consumer at a time: identify touchpoints → update → verify behavior matches → remove
old references → confirm no regressions.

**The churn rule:** if you own the infrastructure being deprecated, you own migrating your
consumers — or you ship a backward-compatible change that needs no migration.

### Step 4: Remove the old system

Only after every consumer has migrated: verify zero usage, remove the code, its
tests/docs/config, and the deprecation notices.

## Migration Patterns

### Strangler

Run old and new in parallel, route traffic incrementally, remove the old at 0%.

```
0% new / 100% old → 10% (canary) → 50% → 100% new / old idle → remove old
```

### Adapter

An adapter translates the old interface to the new implementation; consumers keep the old
interface while you migrate the backend.

```javascript
class LegacyTaskService {
    constructor(newService) { this.newService = newService; }
    getTask(id) {
        return this.toOldFormat(this.newService.findById(String(id)));
    }
}
```

### Feature-flag migration

Switch consumers one at a time behind a flag.

```javascript
function getTaskService(userId) {
    return featureFlags.isEnabled('new-task-service', { userId })
        ? new NewTaskService()
        : new LegacyTaskService();
}
```

### Expand/contract — never rename in place across a rollout

Any in-place rename is risky during the rollout window: old and new code run at once and
one references a name the other doesn't have. Migrate additively so both are valid at
every step.

```
EXPAND ──────────────→ MIGRATE ──────────────→ CONTRACT
add the new form       dual-write / dual-read  once nothing reads the
beside the old,        old + new, migrate       old form, remove it in a
nothing reads it yet   consumers in batches     separate, later change
```

Sequence it as blocked batches: expand first, migrate call sites in batches (each its own
unit, blocked by the expand), contract last. Treat each phase as a thin vertical slice.

**Rules:** additive first, destructive last and alone; every migration has a tested reverse
path; batch bulk work off the hot path.

### Database schema migrations (expand/contract)

A schema change is the riskiest migration — the data is the one thing a deploy-revert can't
roll back. Apply expand/contract: add a new nullable column, dual-write old+new, backfill in
throttled batches, switch reads, then drop the old column in a separate later deploy. Never
couple a destructive schema change to the code change that needs it. Build large indexes
without blocking writes; write and run the reverse migration before merging.

### Version-coupled migration

When a consumer pins a specific version of the thing you're migrating (a served package
version, a pinned dependency, a deployed artifact tag), the pin itself is your strangler
switch — the old version keeps serving until you move the pin. Stage the new version where
it can bake, repin, redeploy, confirm zero usage of the old, then remove the old in a
separate later change. The repin is the cutover; removal is expand/contract applied to the
coupling.

## Zombie Code

Code nobody owns but something depends on — no recent commits, no maintainer, accruing
vulnerabilities. **Response:** assign an owner and maintain it, or deprecate it with a
concrete migration plan. It can't stay in limbo.

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "It still works, why remove it?" | Unmaintained working code accrues security debt and complexity silently |
| "Someone might need it later" | If needed later, rebuild it. Keeping it "just in case" costs more |
| "The migration is too expensive" | Compare to 2–3 years of maintenance; migration is usually cheaper |
| "We'll deprecate after the new system ships" | By then you'll have new priorities. Plan removal at design time |
| "Consumers will migrate on their own" | They won't. Provide tooling, or do it yourself (the churn rule) |
| "We can maintain both indefinitely" | Two systems doing one job is double everything |
| "Just rename it, it's one line" | During rollout, old and new run together — one breaks. Expand/contract |

## Red Flags

- Deprecated system with no replacement available
- A deprecation announcement with no migration tooling or docs
- "Soft" deprecation advisory for years with no progress
- Zombie code with no owner and active consumers
- New features added to a deprecated system
- Deprecation without measuring current usage
- Removing code without verifying zero consumers
- A consumer cut over before the new version was staged and baked
- An in-place rename rather than expand/contract across a rollout
- A schema change shipped in the same deploy as the code that depends on it

## Verification

- [ ] Replacement is production-proven and covers the critical use cases
- [ ] Migration guide exists with concrete steps
- [ ] All active consumers migrated (verified by metrics/logs)
- [ ] Old code, tests, docs, config fully removed
- [ ] No references to the deprecated system remain
- [ ] Any in-place rename went through expand/contract, not a single edit
- [ ] Any schema change shipped additively with a tested reverse path

## See Also

- A ticket-breakdown skill — sequences expand/contract as blocked batches
- An implement-from-spec skill — each migration phase as a thin vertical slice
- A debugging skill — when a migration surfaces a behavior difference

## Adapted from

[addyosmani/agent-skills](https://github.com/addyosmani/agent-skills) (MIT).
