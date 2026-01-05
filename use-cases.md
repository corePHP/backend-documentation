# Use Cases

Use cases represent **application actions**.

They define **what the system does** by coordinating entities and invoking external capabilities through services and repositories.

This document explains how use cases are understood, designed, and used in this architecture.

---

## What Is a Use Case

A **Use Case** represents a single, explicit intent of the system.

Examples of intents:
- Create something
- Update something
- Execute a business action
- Trigger a workflow

A use case answers the question:

> “What is happening?”

---

## Responsibilities

A use case is responsible for:

- Receiving input
- Loading required entities
- Invoking entity behavior
- Calling services when needed
- Persisting resulting state

A use case is **not responsible** for:

- Holding business rules
- Implementing domain behavior
- Performing persistence logic
- Acting as a reusable utility

---

## One Intent per Use Case

Each use case should represent **one intent**.

If behavior starts to diverge with conditions such as:
- flags
- modes
- special cases

This usually means **another use case is needed**.

Clarity is preferred over reuse.

---

## Input Objects

Use cases should receive a **single input object** describing everything they need to execute.

Input objects:
- Make the contract explicit
- Improve readability
- Simplify testing
- Reduce method signature changes over time

---

### Example: Use Case with Input

    final class ShipOrderInput
    {
        public function __construct(
            public readonly int $orderId
        ) {}
    }

    final class ShipOrder
    {
        public function __construct(
            private OrderRepository $orders
        ) {}

        public function execute(ShipOrderInput $input): void
        {
            $order = $this->orders->get($input->orderId);

            $order->ship();

            $this->orders->save($order);
        }
    }

Here:
- The use case defines the flow
- The entity enforces the rule
- The repository handles persistence

---

## Orchestration

Use cases are responsible for **application orchestration**.

Orchestration means:
- Defining the execution order
- Coordinating entities
- Deciding when services are called
- Handling the overall flow of an action

Use cases act as the **recipe**, while entities and services act as **ingredients**.

---

## Use Cases and Services

Services represent **pluggable external capabilities**.

Use cases decide:
- When a service is called
- Why it is called
- How it fits into the flow

Services do not decide application flow.

---

### Example: Orchestration in a Use Case

    final class PayOrder
    {
        public function __construct(
            private OrderRepository $orders,
            private PaymentGateway $paymentGateway,
            private EmailSender $emailSender
        ) {}

        public function execute(PayOrderInput $input): void
        {
            $order = $this->orders->get($input->orderId);

            $order->validate();

            $this->paymentGateway->charge($order->total());

            $order->markAsPaid();

            $this->emailSender->sendConfirmation($order);

            $this->orders->save($order);
        }
    }

The use case defines the flow.  
Services only perform their specific tasks.

---

## Reuse and Duplication

Use cases should not be reused by calling one use case from another.

If two use cases share logic:
- That logic likely belongs in an entity, or
- In a service used by both

Duplicating small amounts of orchestration logic is acceptable if it preserves clarity.

---

## Use Cases and Other Layers

Use cases:

- Are invoked by controllers, CLI commands, or workers
- Depend on repositories and services via interfaces
- Use entities to apply domain behavior
- Do not depend on framework infrastructure directly

This keeps use cases focused and testable.

---

## Testing Use Cases

Use cases are ideal candidates for **unit testing**.

Tests should:
- Provide an input object
- Mock or fake repositories and services
- Assert expected outcomes and interactions

If a use case is difficult or impossible to unit test, it is usually a sign of:
- Too many responsibilities
- Hidden dependencies
- Blurred boundaries

---

## Summary

- Use cases represent application intent
- One use case equals one action
- Use cases orchestrate, entities decide rules
- Services provide capabilities, not flow
- Use cases are easy to unit test

Clear use cases lead to predictable behavior and simpler systems.
