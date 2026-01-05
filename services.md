# Services

Services represent **pluggable capabilities** that can be integrated into the system.

They are used whenever the application needs to interact with something **external or replaceable**, such as third-party tools, infrastructure, or technical providers.

This document explains how services are understood, designed, and used in this architecture.

---

## What Is a Service

A **Service** represents a capability provided to the system.

Typical examples include:
- External APIs
- Payment providers
- Messaging systems
- Email delivery
- File storage
- Authentication providers
- Any tool that can be swapped or replaced

A service answers the question:

> “What capability is available to the system?”

---

## Responsibilities

A service is responsible for:

- Executing a specific capability
- Encapsulating integration details
- Hiding third-party or infrastructure complexity
- Providing a clear and stable interface

A service is **not responsible** for:

- Application flow
- Business rules
- Coordinating entities
- Deciding *when* or *why* it is used

---

## Services and Interfaces

Services are always defined by **interfaces**.

The interface:
- Describes what the service can do
- Is depended on by use cases
- Is stable over time

The implementation:
- Can change
- Can be replaced
- Can vary by environment

---

### Example: Service Interface and Implementation

    interface PaymentGateway
    {
        public function charge(Money $amount): void;
    }

    final class StripePaymentGateway implements PaymentGateway
    {
        public function charge(Money $amount): void
        {
            // Call external Stripe API
        }
    }

The service:
- Does one thing
- Does not know about orders
- Does not know about application flow

---

## Services Do Not Orchestrate

Services **must not contain application orchestration**.

This means a service must not:
- Call multiple entities
- Decide execution order
- Coordinate multiple steps
- Represent a business workflow

Services execute actions.  
Use cases decide **when and how** those actions are used.

---

### Bad Example: Service Doing Orchestration

    final class OrderPaymentService
    {
        public function pay(Order $order): void
        {
            $order->validate();
            $this->gateway->charge($order->total());
            $order->markAsPaid();
            $this->mailer->sendConfirmation($order);
        }
    }

This service:
- Coordinates multiple steps
- Knows business flow
- Acts like a use case

---

### Good Example: Service as a Tool

    interface EmailSender
    {
        public function send(string $to, string $message): void;
    }

    final class SmtpEmailSender implements EmailSender
    {
        public function send(string $to, string $message): void
        {
            // Send email via SMTP
        }
    }

The service only provides a capability.  
The flow belongs elsewhere.

---

## Services and Use Cases

Use cases:
- Decide *when* a service is called
- Decide *why* it is called
- Combine services with entities and repositories

Services:
- Execute their task
- Remain unaware of application intent

This separation keeps the system flexible and easy to change.

---

## Services and Other Layers

Services:

- Are used by use cases
- May be used by entities in rare, well-justified cases
- Do not depend on use cases
- Do not depend on repositories
- Do not depend on framework controllers

This keeps services isolated and reusable.

---

## Testing Services

Services are usually tested in isolation.

Tests should:
- Focus on the service contract
- Mock external dependencies
- Avoid real external systems

In unit tests for use cases, services are typically mocked or faked.

---

## Summary

- Services represent pluggable capabilities
- Services are defined by interfaces
- Services execute actions, not workflows
- Services do not orchestrate application flow
- Use cases decide how services are used

Clear service boundaries prevent hidden coupling and keep application logic predictable.
