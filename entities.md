# Entities

Entities represent **domain concepts** with **state and behavior**.

They are the core of the system and model **what the system is**, not how it is used or accessed.

This document explains how entities are understood, designed, and used in this architecture.

---

## What Is an Entity

An **Entity** represents a meaningful concept in the domain.

An entity:

- Has identity
- Holds state
- Exposes behavior
- Enforces its own rules

Entities are not simple data structures.  
They express **business meaning through behavior**.

---

## Responsibilities

An entity is responsible for:

- Holding its own state
- Defining how that state can change
- Enforcing invariants related to itself
- Exposing intent-revealing methods

An entity is **not responsible** for:

- Application flow
- Persistence logic
- External integrations
- Orchestrating multiple actions

---

## Behavior Over Data

Entities should be designed around **what they do**, not only around what they store.

Instead of thinking:

> “Which fields does this entity have?”

Prefer thinking:

> “What actions make sense for this entity?”

---

### Example: Behavior-Oriented Entity

    final class Order
    {
        private OrderStatus $status;

        public function ship(): void
        {
            if ($this->status !== OrderStatus::PAID) {
                throw new DomainException('Only paid orders can be shipped.');
            }

            $this->status = OrderStatus::SHIPPED;
        }
    }

Here:

- State changes are controlled
- Business rules live close to the data
- Invalid transitions are prevented

---

## About Anemic Entities

An **anemic entity** is an entity that only contains data and no behavior.

Anemic entities often appear:

- In early stages of development
- In simple CRUD scenarios
- When rules are not yet clear

This is not automatically wrong.

As rules emerge, behavior should naturally move **into the entity**, reducing logic elsewhere.

The preferred direction over time is:

> Data → Behavior → Clear Domain Model

---

## Entities and Persistence

Entities may include persistence mapping when using an ORM.

Persistence details:

- Do not define entity behavior
- Do not replace domain rules
- Are considered a technical concern

Entities should remain understandable even without knowledge of how they are stored.

---

## Entities and Other Layers

Entities:

- Are **used by use cases**
- May be **used by services** when appropriate
- Do **not** depend on repositories
- Do **not** depend on services
- Do **not** depend on use cases

This keeps entities independent and reusable.

---

## Design Guidelines

When designing or reviewing an entity, ask:

- Does this entity protect its own state?
- Are invalid states impossible or prevented?
- Are behaviors named after intent?
- Is logic living here instead of being duplicated elsewhere?

If most logic related to an entity lives outside of it, the entity is likely missing behavior.

---

## Testing Entities

Entities are ideal candidates for **unit testing**.

Tests should:

- Call public methods
- Assert state changes
- Validate invariants

If an entity is difficult or impossible to unit test, this is usually a sign of poor design.

---

## Summary

- Entities represent domain concepts
- Behavior is more important than structure
- Entities protect their own invariants
- Persistence is secondary
- Entities are easy to unit test

Well-designed entities simplify use cases, reduce duplication, and make the system easier to reason about.
