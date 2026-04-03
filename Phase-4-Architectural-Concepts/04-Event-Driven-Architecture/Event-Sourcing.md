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

## Core Concepts

### Event Store
The central database, an **append only log** of all events, ordered by time.
```
Event Store (append-only, never delete/update):
┌─────┬─────────────────────┬──────────────┬────────────────────────────┐
│  #  │     event_type      │  timestamp   │            data            │
├─────┼─────────────────────┼──────────────┼────────────────────────────┤
│  1  │ account.opened      │ 09:00:00     │ { user: "alice", bal: 0 }  │
│  2  │ money.deposited     │ 09:05:00     │ { amount: 100 }            │
│  3  │ money.deposited     │ 10:00:00     │ { amount: 50  }            │
│  4  │ money.withdrawn     │ 11:30:00     │ { amount: 30  }            │
│  5  │ money.transferred   │ 14:00:00     │ { amount: 20, to: "bob" }  │
└─────┴─────────────────────┴──────────────┴────────────────────────────┘
Current balance = 100 + 50 - 30 - 20 = $100
```

Key rules:
- Events are **never deleted or updated**, only appended
- Each event has a **sequence number** (for ordering)
- Each event is **immutable**, it's a historical fact


### Aggregate
The domain object whose state is rebuilt by replaying events.
```go
type BankAccount struct {
    ID      string
    Owner   string
    Balance float64
    Status  string
}

// Rebuild state by replaying events — no DB row, just history
func Rebuild(events []Event) *BankAccount {
    account := &BankAccount{}
    for _, event := range events {
        account.Apply(event)  // each event mutates state
    }
    return account
}

func (a *BankAccount) Apply(event Event) {
    switch event.Type {
    case "account.opened":
        a.ID     = event.Data["id"].(string)
        a.Owner  = event.Data["owner"].(string)
        a.Status = "active"

    case "money.deposited":
        a.Balance += event.Data["amount"].(float64)

    case "money.withdrawn":
        a.Balance -= event.Data["amount"].(float64)

    case "account.closed":
        a.Status = "closed"
    }
}
```

### Command vs Event

A critical distinction:
||Command|Event|
|**What**|A request to do something|A record that something happened|
|**Tense**|Imperative ("do this")|Past tense ("this happened")|
|**Can fail?**|Yes, can be rejected|No, already happened, immutable|
|**Example**|`DepositMoney(amount:100)`|`money.deposited { amount: 100}`|

```go
// Command — an intent, can be rejected
type DepositMoneyCommand struct {
    AccountID string
    Amount    float64
}

// Handler validates and converts to an event
func (h *Handler) Handle(cmd DepositMoneyCommand) error {
    if cmd.Amount <= 0 {
        return errors.New("amount must be positive") // command rejected
    }

    // Command accepted → emit an immutable event
    event := Event{
        Type:      "money.deposited",
        Timestamp: time.Now(),
        Data:      map[string]any{"amount": cmd.Amount},
    }
    return h.store.Append(cmd.AccountID, event)
}
```