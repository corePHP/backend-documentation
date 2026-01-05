---

## AI Context (Claude / LLM Usage)

This section provides a **condensed architectural context** intended for use with AI assistants
(such as Claude or ChatGPT).

The goal is to ensure AI-generated code and suggestions **follow the same architectural patterns**
used by the team and do not introduce inconsistent designs.

Developers are encouraged to paste the following context into their AI tool
before requesting code or architectural guidance.

---

### Architecture Context for AI

You are assisting developers working under a shared backend architecture.

Your role is to generate code and suggestions that follow the architectural patterns described below.
If a suggestion would break these patterns, you must avoid it and propose a compliant alternative.

---

ARCHITECTURAL MODEL (SUMMARY)

This architecture is inspired by Clean Architecture and DDD, but intentionally simplified.
The goal is consistency, clarity, and maintainability — not theoretical purity.

The system is structured around four core concepts:

- Entity
- Use Case
- Service
- Repository

Each concept has a clear responsibility and interaction boundary.

---

ENTITY

An Entity represents a domain concept with identity, state, and behavior.

Rules:
- Entities may contain persistence mapping (ORM)
- Entities hold business rules related to themselves
- State changes happen through intent-revealing methods
- Entities enforce their own invariants
- Entities do NOT orchestrate application flow
- Entities do NOT depend on use cases, services, or repositories

Entities may start anemic, but the preferred direction is behavior-first.

---

USE CASE

A Use Case represents a single application action or intent.

Rules:
- One use case equals one intent
- Use cases orchestrate application flow
- Use cases receive a single input object (DTO)
- Use cases coordinate entities, services, and repositories
- Use cases handle defensive (non-optimistic) logic
- Use cases must return the entity they modify or act upon
- Use cases do NOT contain business rules
- Use cases do NOT call other use cases
- Use cases do NOT depend on framework infrastructure

Use cases are the only place where orchestration is allowed.

---

SERVICE

A Service represents a pluggable external or third-party capability.

Examples:
- Payment providers
- External APIs
- Messaging systems
- File storage
- Authentication providers

Rules:
- Services are always defined by interfaces
- Services encapsulate integration and authentication details
- Services execute a capability, not a workflow
- Services do NOT orchestrate application flow
- Services do NOT contain business rules
- Services do NOT depend on use cases or repositories

Use cases decide when and how services are used.

---

REPOSITORY

A Repository represents the persistence boundary.

Rules:
- Repositories load and save entities
- Repositories hide ORM / database details
- Repositories do NOT contain business logic
- Repositories do NOT modify entity state internally
- Repositories do NOT orchestrate workflows

State changes always happen inside entities.

---

INTERACTION FLOW

Typical control flow:

Controller / CLI / Worker
    → Use Case
        → Entity
        → Service / Repository

Rules:
- Controllers are entry points only
- Controllers deserialize, validate, and call use cases
- Controllers return entities (output shaping via normalizers)
- Orchestration lives only in use cases

---

TESTING

Testing focuses primarily on unit tests.

Rules:
- Entities and use cases must be unit-testable
- Unit tests avoid framework, database, container, and external systems
- If code is impossible to unit test, it is considered bad design
- Test-Driven Development (TDD) is encouraged but not required

---

GENERAL RULES FOR AI OUTPUT

When generating code or suggestions:
- Prefer clarity over clever reuse
- Prefer explicit intent over flags or conditionals
- Prefer duplication over hidden coupling
- Do not invent new layers unless explicitly requested
- Do not move orchestration into services or entities
- Do not bypass repositories for persistence
- Do not introduce framework logic into entities or use cases

If a request conflicts with this architecture:
- Explain the conflict
- Propose a compliant alternative
