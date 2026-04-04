CQRS Pattern
==

**CQRS (Command Query Responsibility Segregation)** is a pattern that **separates read operations from write operations** into two distinct models.
> Instead of one model that handles both reading and writing → you have a **Command side** (writes) and a **Query side** (reads), each optimized for its job.

The name says it all:
- **Command** = change something (create, update, delete), answers nothing, just acts
- **Query** = read something, answers a question, changes nothing.

### Analogy
Think of a **restaurant kitchen vs the menu board**:
- The **kitchen** (command side) handles all the work, chopping, cooking, plating. It's optimized for doing things correctly
- The **menu board** (Query side) shows customers what's available and what's ready. It's optimized for being read fast and looked at from anywhere
- The kitchen doesn't update the menu board directly, a waiter carries the information over (the event). Both do their job independently

## The Problem CQRS Solves
In a traditonal architecture, one model does everything:
```go
// One model, one DB — handles reads AND writes
type OrderRepository struct{ db *sql.DB }

// Write: complex business logic, validation, transactions
func (r *OrderRepository) PlaceOrder(order Order) error {
    // validate, check inventory, charge payment, save...
}

// Read: but now you need a complex JOIN for the dashboard
func (r *OrderRepository) GetOrderDashboard(userID string) ([]OrderSummary, error) {
    return r.db.Query(`
        SELECT o.id, o.status, u.name, p.title, SUM(oi.price)
        FROM orders o
        JOIN users u ON o.user_id = u.id
        JOIN order_items oi ON oi.order_id = o.id
        JOIN products p ON oi.product_id = p.id
        WHERE o.user_id = $1
        GROUP BY o.id, u.name, p.title
        ORDER BY o.created_at DESC
    `, userID)
}
```

**The problems with this:**
- Write model is optimized for **consistency**, heavy validation, transactions
- Read model needs **denormalized, fast queries**, but it's stuck in the same normalized schema
- You can't scale reads and writes independently
- Adding a new view (e.g. analytics dashboard) requires changing the core model