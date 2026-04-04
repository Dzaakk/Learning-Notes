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

## How It Works: Step by Step

### Writing (Command → Event)
```
1. User sends command:  DepositMoney { accountID: "acc-1", amount: 100 }
2. Load aggregate:      Replay all past events for "acc-1" to get current state
3. Validate:            Is the account active? Is the amount valid?
4. Emit event:          Append "money.deposited { amount: 100 }" to event store
5. Done.                No UPDATE statements — only INSERT into event log
```

### Reading (Replay → Current State)
```
1. Fetch all events for "acc-1" from event store (ordered by sequence)
2. Start with empty BankAccount{}
3. Apply each event in order:
     account.opened     → set owner, status = active
     money.deposited    → balance += 100
     money.deposited    → balance += 50
     money.withdrawn    → balance -= 30
4. Result: BankAccount{ Balance: 120, Status: "active" }
```

### Go Implementation
```go
// Event store — append-only
type EventStore struct {
    db *sql.DB
}

func (es *EventStore) Append(aggregateID string, event Event) error {
    _, err := es.db.Exec(`
        INSERT INTO events (aggregate_id, event_type, data, occurred_at)
        VALUES ($1, $2, $3, $4)
    `, aggregateID, event.Type, event.Data, event.OccurredAt)
    return err
}

func (es *EventStore) Load(aggregateID string) ([]Event, error) {
    rows, err := es.db.Query(`
        SELECT event_type, data, occurred_at, sequence
        FROM events
        WHERE aggregate_id = $1
        ORDER BY sequence ASC
    `, aggregateID)
    // ... scan rows into []Event
}

// Account service — uses event store
type AccountService struct {
    store *EventStore
}

func (s *AccountService) Deposit(accountID string, amount float64) error {
    // Load current state by replaying
    events, _ := s.store.Load(accountID)
    account   := Rebuild(events)

    // Validate business rules
    if account.Status != "active" {
        return errors.New("account is not active")
    }

    // Append new event
    return s.store.Append(accountID, Event{
        Type:        "money.deposited",
        OccurredAt:  time.Now(),
        Data:        map[string]any{"amount": amount},
    })
}

func (s *AccountService) GetBalance(accountID string) (float64, error) {
    events, err := s.store.Load(accountID)
    if err != nil {
        return 0, err
    }
    account := Rebuild(events)
    return account.Balance, nil
}
```

## Snapshots (Performance Optimization)
Replaying thousands of events every time gets slow. **Snapshots** save the current state periodically so you only replay events *after* the last snapshot.
```
Without snapshot:  replay all 10,000 events on every read (slow)
 
With snapshot:
  Snapshot at event #9000: { balance: 500, status: "active" }
  Only replay events #9001 to #10000 (1000 events — much faster)
```
```go
type Snapshot struct {
    AggregateID string
    Sequence    int
    State       []byte    // serialized aggregate state
    TakenAt     time.Time
}

func (s *AccountService) GetAccount(accountID string) (*BankAccount, error) {
    // Try to load latest snapshot first
    snapshot, _ := s.snapshots.Load(accountID)

    var events []Event
    if snapshot != nil {
        // Only load events AFTER the snapshot
        events, _ = s.store.LoadAfter(accountID, snapshot.Sequence)
        account   := snapshot.ToAccount()  // restore from snapshot
        for _, e := range events {
            account.Apply(e)
        }
        return account, nil
    }

    // No snapshot — full replay
    events, _ = s.store.Load(accountID)
    return Rebuild(events), nil
}

// Take a snapshot every 100 events
func (s *AccountService) maybeTakeSnapshot(accountID string, seq int, account *BankAccount) {
    if seq % 100 == 0 {
        s.snapshots.Save(Snapshot{
            AggregateID: accountID,
            Sequence:    seq,
            State:       serialize(account),
        })
    }
}
```

## Key Benefits
### 1. Compelete Audit Log, Built In
Every change is stored as an event. You never lose history. This is free, not a feature you bolt on.
```go
// "What happened to account acc-1 last Tuesday?"
events, _ := store.LoadBetween("acc-1", tuesday9am, tuesday6pm)
for _, e := range events {
    fmt.Printf("[%s] %s: %v\n", e.OccurredAt, e.Type, e.Data)
}
// [10:05] money.deposited: {amount: 100}
// [11:30] money.withdrawn: {amount: 30}
// [14:00] money.transferred: {amount: 20, to: "bob"}
```

### 2. Time Travel, Query Past State
Replay events up to any point in time.
```go
func GetBalanceAt(accountID string, pointInTime time.Time) float64 {
    // Load only events that happened BEFORE the target time
    events, _ := store.LoadUntil(accountID, pointInTime)
    account   := Rebuild(events)
    return account.Balance
}

balanceLastMonth := GetBalanceAt("acc-1", time.Now().AddDate(0, -1, 0))
```

### 3. Event Replay, Fix Bugs, Add Features
Made a bug in your business logic? Fix the code, then repaly all historical events to rebuild correct state.
```go
// New feature: track total deposits count (didn't exist before)
// Just replay ALL events with the new handler — instant backfill
func backfill(store *EventStore) {
    allEvents, _ := store.LoadAll()
    for _, event := range allEvents {
        if event.Type == "money.deposited" {
            analyticsDB.IncrementDepositCount(event.AggregateID)
        }
    }
}
```

### 4. Natural Integration with EDA
Every event you append is also published to other service, zero extra work.
```go
func (es *EventStore) Append(aggregateID string, event Event) error {
    // 1. Save to event store
    es.db.Exec(`INSERT INTO events ...`)

    // 2. Publish to event bus — free, no extra code
    es.eventBus.Publish(event.Type, event)

    return nil
}
// Email Service, Analytics, Inventory — all react automatically
```

## Trade offs
||✅ Pros|❌ Cons|
|:------|:------|
|Complete audit log for free|Querying current state requires replay|
|Time travel / point in time queries|Event schema changes are hard to manage|
|Replay to fix bugs or add features|Learning curve, different mental model|
|Natural fit with EDA|More storage than traditional DBs|
|No data loss, ever|Eventual consistency on read side|
|Debugging is easy, full history|Snapshots needed at scale|

## When to Use Event Sourcing
### ✅ Use When:
- **Audit trail is a requirement** (finance, healthcare, compliance, legal)
- You need **point in time queries** ("what was the state on Jan 15?")
- Domain has **complex business rules** that evolve over time
- You want **natural integration** with event driven systems
- Debugging correctness matters more than raw simplicity

### ❌ Avoid When:
- **Simpe CRUD** apps, it's massive overkill for a blog or a todo list
- **Read heavy systems** where you need fast, direct queries (use CQRS to add a read model on top)
- **Team unfamiliar with the pattern, steep initial learning curve**
- Schema changes are frequent and not planned for (evolving events is hard)

## Traditional DB vs Event Sourcing

### Updating a User's Address

### Traditional approach
```sql
-- Only current state survives
UPDATE users SET address = '456 New St' WHERE id = 123;
-- Old address is gone forever
```

### Event Sourcing approach
```go
// Append an event — old address still exists in history
store.Append("user-123", Event{
    Type: "address.updated",
    Data: map[string]any{
        "old_address": "123 Old St",
        "new_address": "456 New St",
        "reason":      "moved",
    },
})
// Old address? Still there. Who changed it? In the event.
// When? Timestamp on the event. Why? Also in the event.
```

### Engineering Takeaways
1. **The event log is the truth**, the current state is derived not stored
2. **Events are immutable**, never update or delete them. Ever
3. **Name events in past tense**, `address.updated`, not `update.address`
4. **Include enough data** in each event, consumers shouldn't need to call back
5. **Plan for schema evolution**, events live forever, so think before you change their shape
6. **Add snapshots early**, don't wait until replays get slow. Design for it from the start
7. **Combine with CQRS**, Event sourcing handles writes, CQRS adds optimized read models

### Notes
- **Event sourcing != Event Driven Architecture:** EDA is about how services communicate, ES is about how you store state. They work great together but are independent concepts
- **Outbox pattern:** In production, append the event to the DB and publish to the bus in the same transaction to guarantee consistency
- **Idempotency:** Because events can be replayed, every Apply() function must be idempotent, applying the same event twice should produce the same result
