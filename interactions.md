# Interactions

This document explains **how the main architectural concepts interact** and how responsibilities flow through the system.

Its purpose is to make boundaries explicit and avoid hidden coupling between layers.

---

## High-Level Interaction Flow

A typical execution flow looks like this:

    Controller / CLI / Worker
                ↓
            Use Case
                ↓
             Entity
                ↓
        Service / Repository

This flow represents **control direction**, not object ownership.

---

## Control vs Dependency

Two ideas are important:

- **Control flow**: who calls whom at runtime
- **Dependency direction**: who depends on whose abstractions

In this architecture:
- Control flows **from the outside in**
- Dependencies point **toward abstractions**

---

## Entry Points

Entry points include:
- HTTP controllers
- Console commands
- Message handlers
- Background workers

Entry points are responsible for:
- Receiving input
- Mapping input to a use case
- Triggering execution

Entry points do **not** contain business logic.

---

## Use Case as the Orchestrator

The **use case** is the central coordinator.

It:
- Defines the execution order
- Coordinates entities
- Calls services when needed
- Persists changes through repositories

All application flow decisions live here.

---

## Entity Interaction

Entities:
- Are loaded by repositories
- Are manipulated by use cases
- Enforce their own rules

Entities do **not**:
- Call repositories
- Call services
- Know about application flow

This keeps entities focused and reusable.

---

## Service Interaction

Services:
- Are called by use cases
- Execute a single capability
- Interact with external systems

Services do **not**:
- Decide execution order
- Coordinate multiple steps
- Modify application state directly

They act as tools, not controllers.

---

## Repository Interaction

Repositories:
- Are used by use cases
- Load and persist entities
- Hide persistence details

Repositories do **not**:
- Contain business logic
- Call services
- Coordinate workflows

---

## Typical Interaction Example

    Controller
        ↓
    Use Case
        ↓
    Repository → Entity
        ↓
    Service
        ↓
    Repository

In this example:
- The controller triggers the use case
- The use case loads the entity
- The entity executes behavior
- The service performs an external action
- The repository persists the result

---

## Interaction Rules Summary

Use cases:
- Coordinate everything
- Depend on interfaces

Entities:
- Contain rules and behavior
- Depend on nothing else

Services:
- Provide capabilities
- Do not orchestrate

Repositories:
- Handle persistence
- Do not contain logic

---

## Recognizing Boundary Violations

Common warning signs:

- Business logic inside controllers
- Services calling repositories and entities together
- Repositories modifying entity state
- Entities calling services
- Use cases calling other use cases

These usually indicate blurred responsibilities.

---

## Design Feedback Loop

If interactions feel unclear or hard to follow:
- Responsibilities are likely misplaced
- Boundaries may need adjustment
- Logic may belong in a different layer

Clear interactions lead to simpler tests, easier refactoring, and predictable behavior.

---

## Summary

- Controllers trigger use cases
- Use cases orchestrate
- Entities enforce rules
- Services provide capabilities
- Repositories persist state

Keeping interactions explicit is key to maintainability.
