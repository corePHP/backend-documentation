# Architecture

This document describes **how backend code is structured** and **how the main parts interact**.

It defines a **single, shared approach** used by the team to keep systems easy to understand, easy to change, and consistent over time.

The approach is inspired by **Clean Architecture and DDD**, but intentionally simplified to avoid unnecessary layers and overhead.  
The goal is to provide one clear way of thinking and coding as a team.

The initial design and consolidation of these ideas were proposed by **Vinicius Rangel**.  
If questions arise about decisions or trade-offs, they can be discussed directly.

---

## Table of Contents

1. [Architectural Approach](#architectural-approach)
2. [Core Concepts](#core-concepts)
3. [Interactions](#interactions)
4. [Testing](#testing)
5. [Examples](#examples)
6. [Frameworks](#frameworks)
   - 6.1 [Symfony](#framework-symfony)

---

## Architectural Approach

This architecture borrows the most valuable ideas from Clean Architecture while reducing conceptual fragmentation.

In classic Clean Architecture, concepts such as **Entity**, **Model**, **Use Case**, and **Interactor** are often separated.  
In practice, this separation can lead to duplicated responsibilities, higher cognitive load, and extra maintenance cost.

This approach favors:

- A small number of well-defined concepts
- Clear ownership of behavior
- Predictable interaction between layers

The intention is consistency and maintainability, not theoretical purity.

---

### About Entities in This Architecture

In traditional Clean Architecture:

- **Entities** represent enterprise business rules
- **Models** represent persistence concerns

In this architecture, these concepts are **merged into a single Entity**.

An **Entity**:

- Represents a domain concept
- Holds state and behavior
- May include persistence mapping
- Is not treated only as a storage structure

This removes ambiguity and encourages a **behavior-first mindset**, rather than a database-first one.

When external references define entities differently, the definition in this document represents the standard used by the team.

---

## Core Concepts

### Entity

Represents a domain object with state and behavior.

- Encapsulates business rules related to itself
- Controls how its state changes
- Independent of application flow

→ Details and examples: [entities.md](entities.md)

---

### Use Case

Represents an application action or intent.

- Coordinates entities
- Defines application flow
- Receives input and produces an outcome

→ Details and examples: [use-cases.md](use-cases.md)

---

### Service

Represents a **pluggable external or third-party capability**.

Services are used whenever something is integrated into the system, such as:

- External APIs
- Messaging systems
- Payment providers
- File storage
- Any replaceable external tool

Characteristics:

- Always defined by an interface
- Implementation can be swapped without affecting core logic
- Executes a capability but **does not orchestrate application flow**

→ Details and examples: [services.md](services.md)

---

### Repository

Represents access to persisted entities.

- Loads and saves entities
- Hides persistence details
- Does not contain business rules

→ Details and examples: [repositories.md](repositories.md)

---

## Interactions

A common interaction flow looks like this:

    Controller → Use Case → Entity
                  ↓
              Service / Repository

This flow keeps responsibilities explicit and dependencies predictable.

→ Interaction patterns and variations: [interactions.md](interactions.md)

---

## Testing

Testing in this architecture focuses primarily on **unit tests**.

Unit tests provide fast feedback, protect behavior, and help keep responsibilities clear across entities and use cases.

The goal of testing is **confidence**, not coverage metrics or test quantity.

---

### Unit Tests

Unit tests are the main and preferred form of testing.

They focus on:

- **Entity behavior**
- **Use case execution**
- **Clear inputs and outputs**
- **State transitions and invariants**

Well-written unit tests should:

- Run fast
- Avoid infrastructure
- Be easy to read
- Be easy to maintain

---

### What We Unit Test

#### Entities

- Test behavior through public methods
- Validate state changes
- Ensure invariants are enforced

#### Use Cases

- Test execution flow
- Validate interactions with entities
- Use mocks or fakes for repositories and services

---

### What We Avoid in Unit Tests

Unit tests should **not**:

- Boot the framework
- Use a real database
- Use the service container
- Depend on external systems

These concerns belong to other types of tests and are intentionally out of scope here.

---

### About TDD

Test-Driven Development (TDD) is **encouraged but not required**.

When used, TDD can help:

- Clarify intent before implementation
- Drive better APIs for entities and use cases
- Keep code focused on behavior

TDD is a technique, not a rule.

The important part is that **behavior is testable and tested**, regardless of the order in which code and tests are written.

---

### Testing as Design Feedback

Tests are not only a safety net, but also a design tool.

If a piece of code is **impossible to unit test**, it is usually a sign of **bad design**.

Common indicators include:

- Too many responsibilities
- Hidden dependencies
- Tight coupling to infrastructure
- Unclear boundaries

Improving the design usually improves testability as well.

---

### Summary

- Unit tests are the primary testing strategy
- Entities and use cases are the main focus
- Infrastructure is excluded from unit tests
- TDD is optional, clarity is not

Testing exists to support maintainability and confidence — not to slow development down.

---

## Examples

Examples illustrate common situations and design decisions.

They are intentionally small and focused, serving as reference rather than templates.

→ Browse examples: [examples/README.md](examples/README.md)

---

## Frameworks

Core architectural concepts remain the same across frameworks.  
This section documents how those concepts map to specific frameworks.

---

### Framework: Symfony

Symfony-specific guidance focused on applying **entities, use cases, services, and repositories** within the framework.

Topics include:

- Controllers as entry points
- Dependency injection and the service container
- Doctrine usage in relation to entities
- Console commands and background processes

→ Symfony architecture guide: [frameworks/symfony/README.md](examples/symfony.md)
