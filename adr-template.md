---
# === MACHINE-READABLE METADATA ===
# Fill in all fields. Omit nothing — agents use this frontmatter for routing.
id: ADR-NNN
title: "Use [chosen approach] for [specific problem]"
status: proposed            # proposed | accepted | deprecated | superseded
superseded_by:              # ADR-NNN (leave blank if not applicable)
date: YYYY-MM-DD
decision_makers: [name1, name2]
informed: [team-name, stakeholder-name]
tags: [tag1, tag2]          # e.g. database, security, api-design, messaging
scope:
  services: [service-a, service-b]
  paths: ["src/module-a/**", "src/module-b/**"]
  layers: [domain, application]  # e.g. domain | application | infrastructure | api
enforcement:
  fitness_tests: ["tests/arch/adr-nnn-enforcement.test.ts"]
  archgate_rules: [".archgate/adrs/ADR-NNN.rules.ts"]  # remove if not using Archgate
  ci_required: true
review_trigger: "YYYY-QN"   # scheduled review date e.g. 2026-Q3
---

# ADR-NNN: Use [chosen approach] for [specific problem]

## Context and problem statement

<!--
2–4 sentences only. State the problem as a question.
Include only facts: technological forces, business constraints, team capabilities.
No opinions, no solution hints, no hedging.
-->

[Describe the architectural challenge being addressed. What is the system doing today, and what is the specific problem or gap this decision is designed to solve?]

**Decision question:** What approach should we use for [specific problem] given [key constraints]?

---

## Decision drivers

<!--
Be specific and quantified wherever possible.
Agents cannot act on "should be fast" — write "P95 latency must stay < 200ms."
-->

- [Driver 1 — e.g., "P95 latency must remain < 200ms under 1,000 concurrent users"]
- [Driver 2 — e.g., "Team has 3 engineers with Go experience, 0 with Rust"]
- [Driver 3 — e.g., "Must comply with SOC2 audit trail requirements by Q3"]
- [Driver 4 — e.g., "Monthly infrastructure budget capped at $X"]

---

## Considered options

1. **[Option A]** — [one-sentence description]
2. **[Option B]** — [one-sentence description]
3. **[Option C]** — [one-sentence description]

---

## Decision outcome

**Chosen option: "[Option B]"**, because [1–2 sentence crisp rationale linking directly back to the decision drivers above].

### Consequences

**Positive:**
- [Concrete positive outcome — include expected metric where possible]
- [Concrete positive outcome]

**Negative:**
- [Concrete negative outcome + mitigation strategy]
- [Accepted tradeoff + quantified impact]

**Neutral:**
- [Side effect that is neither clearly positive nor negative]

---

## Constraints and enforcement

<!--
THIS IS THE MOST CRITICAL SECTION FOR AI AGENTS.
Write constraints as precise, testable rules — not narrative prose.
Use ALWAYS / NEVER / ALLOWED / FORBIDDEN as signal words.
Include actual file paths, module names, and package identifiers.
-->

### Invariants — must ALWAYS hold

- `NEVER` import directly from `[module-x]` in any file under `[src/layer-y/]`
- `ALWAYS` use the `[factoryMethod()]` pattern for [component type]
- `ALWAYS` emit domain events for state mutations via `[eventBus.publish()]`

### Preconditions — must be true before using this pattern

- [e.g., Request must include valid JWT with required scopes before reaching handlers]
- [e.g., Service must be registered in the service mesh before deployment]

### Postconditions — guaranteed after correct implementation

- [e.g., All API responses conform to `schemas/api-response.schema.json`]
- [e.g., Audit log entry created for every write operation]

### Dependency boundaries

```
ALLOWED:
  [src/module-a/] → [src/shared/events/], [src/shared/types/]
  [src/module-a/] → external: [approved-package-name]

FORBIDDEN:
  [src/module-a/] ✗ [src/module-b/internal/]
  [src/api/]      ✗ [src/database/]
  [any layer]     ✗ external: [forbidden-package-name]
```

---

## Acceptance criteria

<!--
Each criterion must be independently verifiable.
Link to the test file that enforces it wherever possible.
-->

- [ ] `[tests/arch/adr-nnn-enforcement.test.ts]` passes in CI
- [ ] [Quantified performance criterion, e.g., "P95 latency < 200ms under load test"]
- [ ] [Structural rule, e.g., "Zero direct DB imports in src/api/ (dependency-cruiser rule)"]
- [ ] [Behavioral rule, e.g., "All state mutations produce domain events (ArchUnit verification)"]
- [ ] [Code pattern rule, e.g., "Schema validation middleware present on all new routes"]

---

## Validation and revisit triggers

<!--
Define the conditions that would invalidate this decision.
This section helps agents flag when an ADR may be stale.
-->

Revisit this ADR if any of the following occur:
- [e.g., Monthly infrastructure cost exceeds $X for two consecutive months]
- [e.g., P95 latency exceeds 300ms for 3 or more consecutive days]
- [e.g., Team composition changes by > 50% within 6 months]
- [e.g., [Technology] releases version N with [specific capability that removes the original constraint]]

---

## Alternatives analysis

### Option A: [Name]

**How it works:** [2–3 sentences describing the approach]

**Rejected because:** [Specific reason tied directly to the decision drivers]

```
// Optional: brief code sketch showing this approach
// This helps agents understand what NOT to do
```

### Option C: [Name]

**How it works:** [2–3 sentences describing the approach]

**Rejected because:** [Specific reason tied directly to the decision drivers]

---

## Context for AI agents

<!--
Optional but recommended for complex ADRs.
Write as direct, numbered instructions — not narrative prose.
This section is read by agents immediately before implementation.
-->

When implementing code affected by this ADR:

1. Check `[src/module-a/commands/]` for reference implementations of this pattern
2. Run `[npm run test:arch]` after any changes to verify compliance
3. If uncertain about dependency boundary violations, re-read the **Dependency boundaries** section above before proceeding
4. Do NOT refactor existing code to conform to this pattern unless explicitly instructed
5. [Any other agent-specific guidance]

---

## Related decisions

- **Supersedes:** [ADR-NNN: previous approach this replaces, if any]
- **Depends on:** [ADR-NNN: upstream decisions this relies on]
- **Related:** [ADR-NNN: adjacent decisions in the same domain]

---

*Template version 1.0 — BHIL AI Architecture Toolkit — [barryhurd.com](https://barryhurd.com)*
