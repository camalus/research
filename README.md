@ -1,118 +0,0 @@
# ADR Blueprint for AI-First Development
### Part of the BHIL AI Architecture & Methodology Toolkit

> **"If a decision in an ADR requires a human to clarify it for the AI, the ADR has failed."**

---

## What this is

Architecture Decision Records (ADRs) were designed to inform human developers. In an AI-assisted development environment — where coding agents like Claude Code, Cursor, Devin, and GitHub Copilot are primary implementors — that is no longer sufficient.

This toolkit provides a **gold-standard ADR framework** purpose-built for teams using human-led architecture with AI coding swarms for implementation. It includes a structured template, a filled example, and an AGENTS.md starter that wires your ADR library into the context layer that AI agents actually consume.

The core shift:

| Traditional ADR | AI-First ADR |
|---|---|
| Informs human developers | Instructs autonomous agents |
| Prose-based context | YAML frontmatter + structured constraints |
| Historical record | Executable governance artifact |
| No enforcement mechanism | Paired CI fitness tests |
| Implicit scope | Explicit file path declarations |

---

## Repository structure

```
/
├── README.md                          ← You are here
├── AGENTS.md                          ← AI agent context layer (root)
├── templates/
│   └── adr-template.md               ← Gold-standard ADR template
└── examples/
    └── ADR-001-api-authentication.md  ← Filled example: JWT auth decision
```

---

## The three-tier ADR stack

AI coding agents consume architecture context in layers. This toolkit is designed around that model:

**Tier 1 — Always-loaded session context**
The `AGENTS.md` at repo root. Under 200 lines. References the ADR library but does not inline it. Every agent session begins here.

**Tier 2 — On-demand architecture context**
Individual ADR files in `/docs/adr/` (or your equivalent). Agents retrieve these when working in relevant code paths. YAML frontmatter defines scope so tooling can auto-attach relevant ADRs.

**Tier 3 — Structural enforcement**
Architecture tests (ArchUnit, dependency-cruiser, Archgate) that validate ADR constraints in CI. The safety net that catches what the agent missed.

---

## Key principles

1. **Quantify every constraint.** Replace "should be fast" with "P95 < 200ms." Agents cannot act on qualitative guidance.

2. **Declare scope with file paths.** The YAML `scope.paths` field tells agents exactly which code an ADR governs. Without it, agents guess — and they guess wrong.

3. **Use ALWAYS / NEVER / ALLOWED / FORBIDDEN.** This three-tier boundary pattern maps directly to how all major AI coding tools process instructions.

4. **Enforce structurally, not just documentally.** Pair every critical constraint with a CI-enforced fitness test. Documentation alone is not a guarantee.

5. **Treat ADRs as code.** Version-control them. Validate them in CI. Review them in PRs. ADRs that live in a wiki are invisible to agents.

6. **Run a feedback flywheel.** Agent output and CI failures reveal where documentation is imprecise. Update AGENTS.md and ADR constraints accordingly. Repeat.

---

## Tooling ecosystem

| Layer | Recommended Tool |
|---|---|
| ADR format | MADR 4.0 + extended YAML frontmatter (this template) |
| ADR enforcement | [Archgate](https://archgate.dev) |
| Architecture testing (Java) | [ArchUnit](https://www.archunit.org) |
| Architecture testing (TS/JS) | [dependency-cruiser](https://github.com/sverweij/dependency-cruiser) |
| PR-level surfacing | [Decision Guardian](https://github.com/nicktindall/decision-guardian) |
| Agent context standard | [AGENTS.md](https://agents.md) |

---

## Quick start

1. Copy `templates/adr-template.md` to your ADR directory (e.g., `docs/adr/`)
2. Rename it `ADR-NNN-short-title.md`
3. Fill in all sections — especially **Constraints and Enforcement** and **Acceptance Criteria**
4. Add an entry to `AGENTS.md` referencing the new ADR and its scope
5. Wire acceptance criteria to CI (ArchUnit, dependency-cruiser, or Archgate)

See `examples/ADR-001-api-authentication.md` for a complete filled example.

---

## About

**Author:** Barry Hurd
**Organization:** Barry Hurd Intelligence Lab (BHIL)
**Web:** [barryhurd.com](https://barryhurd.com)

This toolkit synthesizes current best practices from Anthropic, GitHub, Google, Amazon (Kiro), Shopify, ThoughtWorks, and the open-source ADR community (MADR, Archgate, structured-madr). It is updated as the field evolves.

---

## License

This work is licensed under a [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/).

You are free to use, adapt, and redistribute this framework — including commercially — with attribution.

**Attribution format:** *ADR Blueprint for AI-First Development by Barry Hurd / BHIL (barryhurd.com)*

---

## Contributing / feedback

Issues and PRs welcome. If you adapt this framework for a specific tech stack, language ecosystem, or industry context, consider submitting it as an example.
