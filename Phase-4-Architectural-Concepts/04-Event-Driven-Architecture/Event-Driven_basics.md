Event-Driven Architecture Basics
==

## What is Event-Driven Architecture?
**Event-Driven Architecture (EDA)** is a software design pattern where components communicate by **producing and consuming events**, rather than calling each other directly.
> Instead of Service A calling Service B → Service A emits an event → Service B reacts to it

An **event** is a record that something happened, "Order placed", "User signed up", "Payment failed". Events are facts, they describe the **past**.

### Analogy
Think of a **newspaper:**
- The journalist (producer) writes and **publishes** an article
- The journalist doesn't know or care who reads it
- Thousands of readers (consumers) can read the same article independently
- If a new reader subscribes tomorrow, the can still get past editions

In EDA: **producers don't know consumers exist, and consumers don't know each other**.


