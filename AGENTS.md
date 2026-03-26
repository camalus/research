# AGENTS.md — AI Agent Context Layer

> This file is read by AI coding agents (Claude Code, Cursor, Devin, GitHub Copilot, etc.) at the start of every session.
> Keep this file under 200 lines. Reference docs — never inline them.
> Last updated: 2026-03-26

---

## Project overview

[Replace with 2–3 sentences describing your project, its primary language/stack, and the development model.]

Example: *"This is a Node.js/TypeScript microservices platform for [domain]. We use human-led architecture with AI coding swarms for implementation. The codebase follows a domain-driven design structure with strict layer boundaries."*

---

## CRITICAL: Read ADRs before writing code

**Architecture Decision Records govern this codebase.** Before implementing any new feature, service, or significant change, check whether an ADR applies to the code you are about to write.

ADR directory: `docs/adr/`

### Active ADRs by domain

| ADR | Title | Scope (file paths) |
|---|---|---|
| ADR-001 | JWT authentication with refresh token rotation | `src/api/**`, `src/middleware/auth/**` |
| [ADR-NNN] | [Title] | [src/module/**] |
| [ADR-NNN] | [Title] | [src/module/**] |

**How to use this table:**
1. Identify the file paths you plan to modify
2. Find any ADRs whose scope overlaps with those paths
3. Read those ADRs in full before writing code
4. If uncertain whether an ADR applies, read it anyway

---

## Always / Ask / Never

### ALWAYS do these things

- Run `npm run test:arch` (or equivalent) after changes to verify architecture constraints
- Use the factory methods and established patterns shown in `src/[reference-module]/` — do not hand-roll equivalents
- Emit domain events for all state mutations via the designated event bus
- Add `@RequiresAuth` or `@Public` to every new API route — no auth-ambiguous routes
- Write unit tests alongside new code — do not defer to a separate testing pass

### ASK before doing these things

- Adding a new external dependency (package/library)
- Creating a new service or module not covered by an existing ADR
- Modifying database schema or migrations
- Changing an existing API contract (adding/removing/renaming fields)
- Refactoring code that is not directly related to the task at hand

### NEVER do these things

- Import across forbidden dependency boundaries (see individual ADRs for specifics)
- Store secrets, API keys, or credentials in source code or committed config files
- Disable or bypass authentication middleware on any route
- Delete or rename existing ADR files — open an issue instead
- Use `algorithm: 'none'` for JWT (see ADR-001)

---

## Tech stack and conventions

```
Language:    TypeScript [version]
Runtime:     Node.js [version]
Framework:   [Express / Fastify / NestJS]
Database:    [PostgreSQL / MySQL / MongoDB]
ORM:         [Prisma / TypeORM / Drizzle]
Testing:     [Jest / Vitest] + [Supertest for integration]
Linting:     ESLint + Prettier (config in .eslintrc, .prettierrc)
Architecture tests: [dependency-cruiser / ArchUnitTS / Archgate]
```

### Code style essentials

- Functions over classes for stateless utilities
- Explicit return types on all exported functions
- `async/await` over promise chains
- Error types live in `src/errors/` — do not throw raw `Error` objects
- [Add your most important project-specific conventions here]

---

## Test commands

```bash
npm run test          # Unit tests
npm run test:int      # Integration tests
npm run test:arch     # Architecture constraint tests (run after any structural change)
npm run lint          # ESLint + Prettier check
npm run build         # TypeScript compile check
```

> Run `test:arch` whenever you modify files in `src/api/`, `src/services/`, or `src/middleware/`.

---

## Repository structure

```
src/
├── api/           # Route handlers only — no business logic
├── services/      # Domain services
├── middleware/    # Cross-cutting concerns (auth, logging, validation)
├── database/      # Repositories and migrations only
├── shared/        # Types, events, utilities shared across services
├── errors/        # Error types and error handling utilities
└── types/         # Global TypeScript type extensions

docs/
└── adr/           # Architecture Decision Records ← READ THESE

tests/
├── unit/
├── integration/
└── arch/          # Architecture enforcement tests
```

---

## When you are uncertain

1. Check the relevant ADR in `docs/adr/`
2. Check the reference implementation in `src/[nearest analogous module]/`
3. Check existing tests for examples of the expected pattern
4. If still uncertain, **stop and ask** rather than making an assumption

The goal is zero human clarification loops. If an ADR is ambiguous enough that you needed to stop here, that is valuable signal — note it so the ADR can be improved.

---

*AGENTS.md — BHIL AI Architecture Toolkit — [barryhurd.com](https://barryhurd.com)*
*Maintained by: [Barry Hurd](https://barryhurd.com)*
