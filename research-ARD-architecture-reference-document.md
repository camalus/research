# Architecture Decision Records for the age of AI coding agents

**ADRs are becoming the critical control plane for AI-assisted development**, but traditional formats were never designed for machine consumption. The field is rapidly converging on a dual-readable paradigm: Markdown prose for humans paired with YAML frontmatter and executable companion rules for AI agents. Teams that treat ADRs as enforceable, machine-parseable governance artifacts — rather than passive documentation — are seeing dramatically fewer human clarification loops in agentic coding pipelines. This guide synthesizes emerging best practices from across the industry (Google, Shopify, GitHub, Amazon, ThoughtWorks) and presents a gold-standard template purpose-built for AI-first development teams.

The strategic insight is deceptively simple: **an AI coding agent can only be as autonomous as the architecture documentation it receives is precise**. Vague prose, ambiguous constraints, and implicit assumptions are the primary sources of agent drift, rework, and escalation. The practices in this guide aim to eliminate that ambiguity.

---

## Traditional ADR formats and where they break down

Michael Nygard's original 2011 ADR format — Title, Status, Context, Decision, Consequences — remains the most widely adopted template, supported by tools like adr-tools (5.3k GitHub stars) and baked into engineering culture at dozens of major companies. Its genius is simplicity: five sections, one to two pages, stored in version control alongside code. **MADR 4.0** (Markdown Architectural Decision Records) extends Nygard with explicit Decision Drivers, Considered Options with structured pros/cons, a Confirmation section, and — critically — YAML frontmatter for machine-readable metadata. RFC-style documents, used at companies like Uber, Spotify, and Stripe, serve a complementary pre-decision role but lack standardized templates. Y-statements compress decisions into a single structured sentence: *"In the context of X, facing Y, we decided for Z to achieve Q, accepting D."*

Every one of these formats was designed for human readers. When evaluated against what AI coding agents actually need, they share common failure modes:

- **Ambiguity in natural language**: "We should avoid heavy frameworks" gives an agent nothing actionable. What constitutes "heavy"? What's the threshold?
- **No structured constraint fields**: Technology choices, dependency rules, and API contracts are buried in prose rather than exposed as parseable metadata.
- **Missing acceptance criteria**: Nygard has no validation mechanism. MADR 4.0's Confirmation section is a step forward but remains optional free text.
- **Scope ambiguity**: ADRs rarely specify which files, modules, or services they apply to. An agent working in `service-A/` cannot determine which ADRs from a shared repository are relevant.
- **No inter-ADR dependency graph**: When ADRs conflict or supersede each other, the precedence chain is informal at best.

The gap is clear: traditional ADRs tell agents *what was decided* but not *how to verify compliance*, *which code paths are affected*, or *what specific constraints are enforceable*.

---

## How AI coding agents actually consume architecture documentation

Understanding the consumption model of major AI coding tools is essential to writing ADRs they can act on. Each tool has converged on a similar pattern: a **root-level Markdown file loaded into every session**, with hierarchical overrides and progressive disclosure for deeper context.

**Claude Code** reads `CLAUDE.md` at session start as a constitutional document. It supports nested `CLAUDE.md` files in subdirectories, a skills system (`.claude/skills/SKILL.md` with YAML frontmatter), and `@import` references for progressive disclosure. Anthropic recommends keeping CLAUDE.md under **200 lines** — every token consumed by instructions is a token unavailable for code reasoning. The practical ceiling for instructions that frontier LLMs follow consistently is approximately **150–200 discrete directives**.

**Cursor** evolved from `.cursorrules` to a richer `.cursor/rules/*.mdc` system supporting YAML frontmatter, glob-pattern auto-attachment (e.g., rules that activate only for `*.ts` files), and semantic codebase embeddings for RAG-style retrieval. **GitHub Copilot** uses `.github/copilot-instructions.md` for project-wide instructions and `.github/instructions/*.instructions.md` for path-specific rules, plus the emerging Copilot Spaces feature for curated context collections. **Devin** relies on AGENTS.md files, automatic repository indexing, persistent knowledge learned from interactions, and structured playbooks with step-by-step instructions and success criteria.

The cross-tool standard emerging under Linux Foundation stewardship is **AGENTS.md**, now adopted by **60,000+ open-source repositories** and supported by OpenAI Codex, Cursor, Copilot, Devin, Gemini CLI, and others. GitHub's analysis of 2,500+ agent configuration files revealed six core content areas that consistently improve agent performance: **executable commands** (placed early with full flags), **testing instructions**, **code style and architecture patterns**, **context boundaries** (what to modify vs. what not to touch), **quality requirements**, and **dependency constraints**.

Seven characteristics make architecture documentation parse well for AI agents:

1. **Clear Markdown hierarchy** with H2/H3 headers that agents use as semantic navigation anchors
2. **Code examples over prose** — one real snippet showing your pattern beats three paragraphs describing it
3. **Explicit three-tier boundaries**: Always do / Ask first / Never do
4. **YAML frontmatter** for machine-readable metadata (status, scope, affected files, tags)
5. **Quantified constraints** ("P95 latency < 200ms") rather than qualitative goals ("should be fast")
6. **Backtick-wrapped commands** that agents can copy-paste and execute
7. **Progressive disclosure** — point to detailed docs rather than inlining everything

Documents that parse poorly share opposite traits: vague hedging language ("maybe consider"), conflicting instructions, stale file paths, excessive length without summarization, and auto-generated content that buries signal in noise.

---

## What leading engineering organizations are doing right now

**Shopify** represents the most publicly advanced AI-first engineering culture. CEO Tobi Lütke's 2025 memo declared "reflexive AI usage a baseline expectation." The Augmented Engineering DX team open-sourced **Roast**, a convention-oriented workflow orchestration framework that interleaves deterministic code execution with AI behavior via YAML configuration and Markdown prompts. Shopify builds internal LLM proxies, MCP integrations, and requires all engineers — including directors and above — to pass coding interviews ensuring technical depth for AI supervision.

**Google** has layered AI assistance onto existing design doc processes rather than replacing them. Over **one-third of Google's code is now AI-generated** (confirmed by CEO Sundar Pichai). Google Cloud published explicit best practices for AI coding assistants: plan before coding, document early to produce higher-quality AI output, and create context files (GEMINI.md) that serve as "cheat sheets" including architecture diagrams and dependency versions. Addy Osmani, engineering leader on Google's Gemini team, advocates **spec-driven development** — "waterfall in 15 minutes" — where structured specifications precede any code generation.

**Amazon** launched **Kiro**, an agentic coding IDE with spec-driven development as its core workflow. Kiro follows a three-phase pipeline: Specify → Plan → Execute. It generates requirements in EARS notation, designs architecture, and creates implementation plans with sequenced tasks. Its "powers" system loads specialized capabilities on demand to keep context windows uncluttered.

**GitHub** open-sourced **spec-kit**, a toolkit for spec-driven development that adds "constitutions" defining immutable principles. Their thesis: "In this new world, maintaining software means evolving specifications. The lingua franca of development moves to a higher level."

**ThoughtWorks Technology Radar (November 2025)** marked a significant maturity shift. CTO Rachel Laycock noted: "Vibe coding has practically disappeared; we now see a concerted and serious effort to think through problems of context, infrastructure and security." The Radar assessed **context engineering** — the systematic design of information provided to LLMs during inference — as the critical emerging discipline, and flagged a promising new pattern: **anchoring coding agents to a reference application** via MCP servers that expose template code and detect drift.

However, Martin Fowler's team raised an important caution about spec-driven development: the term is "not very well defined yet, and it's already semantically diffused." They observed agents frequently ignoring parts of specifications despite large context windows. The practical takeaway: **documentation alone is insufficient — structural enforcement through tests and CI is essential**.

---

## The emerging AI-first architecture documentation stack

A clear technology stack is crystallizing for AI-consumable architecture governance:

| Layer | Tool / Practice | Maturity |
|---|---|---|
| Agent context | AGENTS.md / CLAUDE.md / GEMINI.md | Production (60K+ repos) |
| ADR format | MADR 4.0 + extended YAML frontmatter | Stable standard |
| ADR enforcement | Archgate (executable ADRs with `.rules.ts`) | Early production |
| ADR surfacing | Decision Guardian (GitHub Action on PRs) | Production, open source |
| Architecture models | Structurizr DSL / C4 model | Mature |
| Architecture testing | ArchUnit / ArchUnitTS / dependency-cruiser | Production-grade |
| Documentation for LLMs | llms.txt standard | Growing adoption |
| Structured decision format | Decision Reasoning Format (YAML/JSON) | Experimental |

**Archgate** is the most significant new entrant. It turns ADRs into an executable governance layer: each ADR Markdown file can have a companion `.rules.ts` file with automated TypeScript checks that run in CI, pre-commit hooks, and via editor plugins for Claude Code, Cursor, and Copilot. The governance loop is: ADRs loaded as context → agents read before writing code → `archgate check` validates every change → AI reviews what lint can't catch → violations become new automated rules. This creates a **self-improving ratchet** where the gap between documented architecture and implemented architecture continuously narrows.

**Structured MADR** (zircote/structured-madr) extends MADR with rich YAML frontmatter — including type, category, tags, technologies, audience, and audit trail fields — plus JSON Schema validation and a GitHub Action validator. It explicitly targets "compliance-driven projects (SOC2, HIPAA), AI-assisted development (Claude Code, GitHub Copilot, Cursor), large codebases, enterprise architecture."

The **llms.txt** standard, proposed by Jeremy Howard in September 2024, provides a `/llms.txt` Markdown file at website roots for LLM-friendly content. Fern reports it **reduces token consumption by 90%+** compared to HTML. The same principle applies to internal documentation: optimizing format for LLM consumption is not a nice-to-have but a performance multiplier.

---

## Best practices for acceptance criteria and testable constraints

The principle linking ADRs to enforcement is borrowed from evolutionary architecture: **"A decision record documents the decision, while a fitness function assures the decision."** Fitness functions — any mechanism providing objective integrity assessment of architectural characteristics — are the bridge between passive documentation and active governance.

**Every ADR should pair its decision statement with at least one enforceable constraint.** These constraints fall into three categories inspired by design-by-contract thinking:

**Preconditions** define what must be true before a component or pattern is used. Example: "API requests to the order service must include a valid JWT token with the `orders:read` scope." These translate directly into middleware validation rules or API gateway policies.

**Postconditions** define what the component guarantees after execution. Example: "All state mutations in the order domain must produce an event on the `order-events` Kafka topic." These translate into ArchUnit-style tests asserting that all command handlers call `eventStore.publish()`.

**Invariants** define constraints that must hold at all times. Example: "No direct dependency from the UI layer to the database layer." These translate into dependency-cruiser rules or ArchUnit layered architecture assertions.

The tooling ecosystem for enforcement is mature: **ArchUnit** (Java), **ArchUnitNET** (C#), **ArchUnitTS** (TypeScript), **dependency-cruiser** (JavaScript/TypeScript), and **import-linter** (Python) all support architecture-as-test patterns. ArchUnit's `freezeArchRules` feature is particularly valuable for brownfield adoption — it baselines existing violations and reports only new ones.

For ADR acceptance criteria specifically, follow these practices:

- Write **quality attribute scenarios** with stimulus, response, and response measure: "Under 1,000 concurrent users, checkout flow P95 latency < 200ms with zero data loss"
- Include **validation/revisit triggers** — specific signals that would invalidate the decision: "Revisit if monthly cost exceeds $5,000 or team grows beyond 15 engineers"
- Reference **specific fitness function implementations** by filename: "See `tests/arch/order-boundaries.test.ts` for enforcement"
- Avoid pseudo-accuracy (arbitrary numerical scoring of qualitative factors) and fuzzy wording ("appropriate," "reasonable")

---

## Integrating ADRs into agentic coding pipelines

The most effective integration pattern positions ADRs within a **three-tier context architecture** that agents consume at different granularities:

**Tier 1 — Session context (always loaded):** The AGENTS.md or CLAUDE.md file at repo root. This should be under 200 lines and contain only universally applicable constraints. It references ADRs but does not inline them. Example directive: *"Before implementing any new service, read `/docs/adr/` for architectural constraints. Follow ADR-007 (event sourcing) for all state mutations."*

**Tier 2 — On-demand architecture context:** The `/docs/adr/` directory containing individual ADR files with YAML frontmatter. Agents retrieve these when working in relevant code paths. File-scoped rules (Cursor `.mdc` with glob patterns, Copilot path-specific instructions) can auto-attach relevant ADRs. Example: a rule that auto-loads `ADR-012-api-conventions.md` whenever the agent edits files in `src/api/`.

**Tier 3 — Executable enforcement:** Architecture tests (ArchUnit, dependency-cruiser) and Archgate `.rules.ts` files that run in CI. These catch violations that escaped the agent's attention, creating a safety net. The results feed back into Tier 1 context — a **data-driven flywheel** where CI failures inform CLAUDE.md improvements.

**Microsoft's VS Code team** recommends complementary context documents alongside ADRs: `PRODUCT.md` (max 2 pages — product vision and goals), `ARCHITECTURE.md` (max 2 pages — system architecture and design principles), and `CONTRIBUTING.md` (max 1 page — developer guidelines). This separation prevents any single document from overwhelming the context window.

For multi-agent systems, Google's Agent Development Kit pattern is instructive: context is a **compiled view over a richer stateful system**. Sessions, memory, and artifacts are the sources; flows and processors transform state per invocation; working context is the compiled view shipped to the LLM for one call. In practice this means ADRs are the source of truth, but what the agent actually sees is a **compiled subset** relevant to its current task.

A research paper on multi-agent coding context (arXiv 2508.08322) found that **structuring external knowledge as bullet points or Q&A pairs makes it dramatically easier to integrate than dumping long paragraphs**. This directly informs how ADR sections should be written for agent consumption.

---

## Gold-standard ADR template for AI-first development

The following template synthesizes MADR 4.0 structure, Archgate patterns, design-by-contract thinking, and empirical findings on what AI agents parse well. It is designed so that an AI coding agent can read it, understand the constraints, implement accordingly, and self-verify compliance.

```markdown
---
# === MACHINE-READABLE METADATA (YAML FRONTMATTER) ===
id: ADR-NNN
title: "Use [chosen approach] for [specific problem]"
status: proposed | accepted | deprecated | superseded
superseded_by: ADR-NNN  # if applicable
date: YYYY-MM-DD
decision_makers: [name1, name2]
informed: [team1, team2]
tags: [database, security, api-design]  # for cross-referencing
scope:
  services: [order-service, inventory-service]
  paths: ["src/orders/**", "src/inventory/**"]
  layers: [domain, application]  # architectural layers affected
enforcement:
  fitness_tests: ["tests/arch/adr-nnn-enforcement.test.ts"]
  archgate_rules: [".archgate/adrs/ADR-NNN.rules.ts"]  # if using Archgate
  ci_required: true
review_trigger: "2026-Q3"  # scheduled review date
---

# ADR-NNN: Use [chosen approach] for [specific problem]

## Context and problem statement

[2-4 sentences. State the problem as a question. Include only facts — 
technological forces, business constraints, team capabilities. 
No opinions, no solution hints.]

What approach should we use for [specific problem] given [key constraints]?

## Decision drivers

- [Driver 1: quantified where possible, e.g., "P95 latency must stay < 200ms"]
- [Driver 2: e.g., "Team has 3 engineers with Go experience, 0 with Rust"]
- [Driver 3: e.g., "Must comply with SOC2 audit trail requirements"]
- [Driver 4: e.g., "Monthly infrastructure budget capped at $X"]

## Considered options

1. **[Option A]** — [one-sentence description]
2. **[Option B]** — [one-sentence description]
3. **[Option C]** — [one-sentence description]

## Decision outcome

**Chosen option: "[Option B]"**, because [1-2 sentence crisp rationale 
linking back to decision drivers].

### Consequences

**Positive:**
- [Concrete positive outcome with expected metric]
- [Concrete positive outcome]

**Negative:**
- [Concrete negative outcome with mitigation strategy]
- [Accepted tradeoff with quantified impact]

**Neutral:**
- [Side effect that is neither clearly positive nor negative]

## Constraints and enforcement

<!-- THIS SECTION IS THE KEY DIFFERENTIATOR FOR AI AGENTS -->
<!-- Write constraints as precise, testable rules -->

### Invariants (must ALWAYS hold)
- `NEVER` import directly from `src/database` in any file under `src/api/`
- `ALWAYS` use the `createRoute()` factory for API route handlers
- `ALWAYS` emit domain events for state mutations via `eventBus.publish()`

### Preconditions (must be true before using this pattern)
- Request must include valid JWT with required scopes
- Service must be registered in the service mesh before deployment

### Postconditions (guaranteed after correct implementation)
- All API responses conform to `schemas/api-response.schema.json`
- Audit log entry created for every write operation

### Boundaries
```
ALLOWED dependencies:
  src/orders/ → src/shared/events/, src/shared/types/
  src/orders/ → external: @company/event-bus, zod

FORBIDDEN dependencies:
  src/orders/ ✗ src/inventory/internal/
  src/orders/ ✗ external: any ORM except prisma
  src/api/   ✗ src/database/
```

## Acceptance criteria

<!-- Specific, measurable conditions for "done" -->
- [ ] `tests/arch/adr-nnn-enforcement.test.ts` passes in CI
- [ ] P95 latency < 200ms under 1,000 concurrent users (load test)
- [ ] Zero direct DB imports in `src/api/` (dependency-cruiser rule)
- [ ] All state mutations produce events (ArchUnit/grep verification)
- [ ] Schema validation middleware present on all new API routes

## Validation and revisit triggers

<!-- Conditions that would invalidate this decision -->
Revisit this decision if:
- Monthly infrastructure cost exceeds $X
- P95 latency exceeds 300ms for 3 consecutive days
- Team composition changes significantly (e.g., > 50% turnover)
- [Technology] releases version N with [specific capability]

## Alternatives analysis

### Option A: [Name]
**How it works:** [2-3 sentences]
**Rejected because:** [specific reason tied to decision drivers]
```
// Example code showing this approach (helps agents understand what NOT to do)
```

### Option C: [Name]  
**How it works:** [2-3 sentences]
**Rejected because:** [specific reason tied to decision drivers]

## Context for AI agents

<!-- Optional section: explicit instructions for coding agents -->
When implementing code affected by this ADR:
1. Check existing patterns in `src/orders/commands/` for reference implementations
2. Run `npm run test:arch` after changes to verify compliance
3. If uncertain about boundary violations, check `FORBIDDEN dependencies` above
4. Do NOT refactor existing code to match this pattern unless explicitly asked

## Related decisions

- Supersedes: [ADR-NNN: previous approach]
- Depends on: [ADR-NNN: event bus selection]
- Related: [ADR-NNN: API versioning strategy]
```

---

## Nine principles for maximum agent autonomy from ADRs

Distilling all findings into actionable principles for a Principal Solutions Architect leading AI coding swarms:

**1. Quantify every constraint.** Replace "should be fast" with "P95 < 200ms." Replace "avoid tight coupling" with a specific dependency rule in the Boundaries section. AI agents cannot act on qualitative guidance — they need thresholds, patterns, and explicit rules.

**2. Declare scope with file paths.** The YAML frontmatter `scope.paths` field tells agents exactly which code this ADR governs. Without it, agents must guess — and they guess wrong. Glob patterns (`src/orders/**`) are universally understood by all major AI coding tools.

**3. Show, don't just tell.** One code snippet showing the correct pattern eliminates more agent confusion than three paragraphs of prose. Include reference implementations in the ADR or point to canonical examples in the codebase.

**4. Enforce structurally, not just documentally.** Martin Fowler's team observed agents frequently ignoring parts of specifications. The only reliable guarantee is a failing CI check. Pair every critical constraint with an ArchUnit test, dependency-cruiser rule, or Archgate `.rules.ts` file.

**5. Design progressive disclosure.** Keep the root AGENTS.md/CLAUDE.md under 200 lines. Reference ADRs by path rather than inlining them. Use Cursor's glob-pattern auto-attachment or Copilot's path-specific instructions to load relevant ADRs only when the agent is working in affected code.

**6. Use YAML frontmatter consistently.** Every ADR gets machine-readable metadata: status, date, scope, tags, enforcement file references. This enables tooling to filter, route, and validate ADRs programmatically. YAML outperforms JSON for LLM comprehension by up to **54%** in benchmarks and supports inline comments that provide critical context.

**7. Close the feedback loop.** Run a data-driven flywheel: review agent output and CI failures → update CLAUDE.md and ADR constraints → measure improvement. Shopify and multiple practitioner teams report this as the highest-leverage practice for improving agent autonomy over time.

**8. Apply the "three-tier boundary" pattern.** For every architectural concern, define what agents must ALWAYS do, what they should ASK about first, and what they must NEVER do. This maps directly to how all major AI coding tools process instructions.

**9. Treat ADRs as code.** Version-control them alongside application code. Validate them in CI (Archgate, JSON Schema, markdownlint). Review them in PRs. Auto-surface them when affected code changes (Decision Guardian). The moment ADRs live in a wiki or Google Doc, they become invisible to agents and stale for humans.

## Conclusion

The field of architecture documentation for AI-assisted development has moved from "vibe coding" through supervised agents to **context engineering as a discipline** in under 18 months. The convergence point is clear: ADRs must become dual-readable artifacts — human-friendly prose with machine-parseable metadata and enforceable companion rules. The organizations seeing the highest AI agent autonomy share three characteristics: they quantify constraints rather than describing them qualitatively, they enforce structurally through CI rather than relying on documentation alone, and they run continuous feedback loops between agent behavior and documentation quality.

The gold-standard template above encodes these principles. The YAML frontmatter provides machine-readable routing and metadata. The Constraints and Enforcement section uses precise `ALWAYS/NEVER` directives with explicit dependency boundaries. The Acceptance Criteria section defines verifiable conditions. The Context for AI Agents section provides implementation-time guidance. Together, these sections transform an ADR from a passive historical record into an **active governance instrument** that AI coding swarms can interpret, follow, and self-verify against — with minimal human intervention.

The key open question, flagged by both ThoughtWorks and Martin Fowler's team, is whether heavy upfront specification scales or risks reintroducing waterfall-like rigidity. The pragmatic answer: start with structural enforcement of the three to five most critical architectural invariants, expand incrementally as the feedback loop reveals where agents drift, and resist the temptation to over-specify. The goal is not perfect documentation — it is documentation precise enough that an AI agent's most likely failure mode is caught by a CI check before a human ever sees it.