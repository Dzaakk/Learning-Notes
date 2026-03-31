Event Sourcing
==

## What is Event Sourcing?
**Event Sourcing** is a pattern where instead of storing only the **current state** of data, you store the **full history of events** that led to that state.
> Traditional DB: "User's balance is $70"\
> Event Sourcing: "User deposited $100, then withdrew"\
> $30 → current balance is $70

The **event log is the source of truth**, the current state is just something you *derive* from it, not something you store directly.

### Analogy
Think of a **bank statement** vs a balance display:
- Most apps store only the **current balance** → fast, simple, but you lose history.
- A bank stores every **transaction** (deposit, withdrawal, transfer) → the balance is just the sum of all transactions.
- Lose the balance? Recalculate it from transactions.
- Want balance from 3 month ago? Replay transactions up to that point.

**Event Sourcing treats your entire database like a bank statement**.

