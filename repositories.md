# Repositories

Repositories represent the **persistence boundary** of the system.

They are responsible for retrieving and storing entities while hiding the details of how and where data is persisted.

This document explains how repositories are understood, designed, and used in this architecture.

---

## What Is a Repository

A **Repository** represents access to a collection of entities.

It answers the question:

> “How do we retrieve and persist entities?”

Repositories abstract away:
- Databases
- ORMs
- Query languages
- Storage mechanisms

---

## Responsibilities

A repository is responsible for:

- Loading entities
- Persisting entities
- Translating between storage and domain objects
- Hiding persistence details from the rest of the system

A repository is **not responsible** for:

- Business rules
- Domain behavior
- Application flow
- Coordinating multiple actions

---

## Repositories and Entities

Repositories work **with entities**, not with raw data.

A typical interaction is:

1. A use case requests an entity from a repository
2. The entity behavior is executed
3. The updated entity is persisted by the repository

Repositories should not modify entity state internally.

---

## Read and Write Patterns

### Reads

Read operations typically:
- Accept identifiers or criteria
- Return entities or collections of entities

Examples:
- `getById`
- `findByEmail`
- `findActive`
- `searchByCriteria`

---

### Writes

Write operations:
- Receive entities
- Persist entity state
- Do not accept partial data

Repositories should not expose methods like:
- `updateStatus`
- `changeField`
- `saveRawData`

State changes belong to entities.

---

### Example: Repository Interface

    interface OrderRepository
    {
        public function get(int $id): Order;

        public function save(Order $order): void;
    }

The repository:
- Loads the entity
- Persists the entity
- Does not know *why* changes happen

---

## Repositories and ORM

When using an ORM:
- The repository implementation may use ORM features
- ORM specifics remain inside the repository
- Domain code does not depend on ORM APIs

ORM usage is considered an **implementation detail**.

---

## Repositories and Use Cases

Use cases:
- Depend on repository interfaces
- Use repositories to load and persist entities
- Do not perform persistence logic themselves

Repositories:
- Do not call use cases
- Do not contain orchestration logic

This keeps responsibilities clearly separated.

---

## Repositories and Services

Repositories and services serve different purposes:

- Repositories handle **persistence**
- Services handle **capabilities**

They should not replace each other.

A repository should not:
- Call external services
- Integrate third-party tools

---

## Testing Repositories

Repositories are typically tested separately from domain logic.

Testing strategies may include:
- Integration tests
- ORM-level tests
- Database-specific tests

In unit tests:
- Repositories are usually mocked or faked
- Use cases and entities are tested without persistence

---

## Design Guidelines

When designing or reviewing a repository, ask:

- Does it only deal with persistence?
- Does it return entities, not raw data?
- Is all business logic outside the repository?
- Is the interface simple and intention-revealing?

If a repository contains business rules, responsibilities are likely misplaced.

---

## Summary

- Repositories define the persistence boundary
- They load and save entities
- They hide storage details
- They do not contain business logic
- They are used by use cases, not entities

Clear repositories keep domain logic clean and persistence concerns isolated.
