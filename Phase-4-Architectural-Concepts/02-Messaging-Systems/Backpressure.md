Backpressure
===

## What is Backpressure?
**Backpressure** is the mechanism that lets a slow consumer signal to a fast producer to slow down, preventing the system from being overwhelmed.
![backpressure](../../images/Phase-4-Architectural-Concepts/backpressure.png)

Without backpressure, the gap between producer and consumer speed causes unbounded queue growth, memory exhaustion, or message loss.

The analogy: imagine a garden hose (producer) filling a small cup (cosnumer). Backpressure is what tells you to turn down the tap before the cup overflows.

## Why It Matters
In distributed systems, services rarely run at the same speed. Traffic spikes, slow database queries, or downstream failures can all cause a consumer to fall behind. Without backpressure:
- **Queues gro unbounded** → memory exhaustion
- **Message are dropped** → data loss
- **Cascading failures** → the whole system goes down
- **Producer wastes resources** → processing work that will never be consumed

## Backpressure Strategies

### 1. Blocking (Syncrhonous Backpressure)
Producer is blocked, it cannot send until the consumer is ready.
> Producer → tries to send → buffer full → BLOCKS → waits → consumer drains → unblocks

Simple and safe. Works well when producer and consumer are in the same process.
```go
package main

import "fmt"

func main() {
	// Buffered channel = bounded queue
	queue := make(chan int, 5) // buffer of 5

	// Producer
	go func() {
		for i := 1; i <= 10; i++ {
			queue <- i // BLOCKS when buffer is full (backpressure!)
			fmt.Printf("[Producer] Sent: %d\n", i)
		}
		close(queue)
	}()

	// Slow consumer
	for msg := range queue {
		fmt.Printf("[Consumer] Processing: %d\n", msg)
		// time.Sleep(100 * time.Millisecond) // simulate slow consumer
	}
}
```
When the buffer is full `queue <- i blocks automatically. The producer slows to the consumer's pace. This is Go's built in backpressure via buffered channels.

### 2. Dropping (Lossy Backpressure)
When the buffer is full, new messages are dropped. Acceptable when losing some data is tolerable (e.g., metrics, logs, live game state).
```go
package main

import "fmt"

func tryPublish(queue chan<- int, msg int) bool {
	select {
	case queue <- msg:
		return true // accepted
	default:
		return false // dropped — queue full
	}
}

func main() {
	queue := make(chan int, 3)

	dropped := 0
	for i := 1; i <= 10; i++ {
		if !tryPublish(queue, i) {
			dropped++
			fmt.Printf("[Producer] Dropped: %d\n", i)
		} else {
			fmt.Printf("[Producer] Sent: %d\n", i)
		}
	}
	fmt.Printf("Total dropped: %d\n", dropped)
}
```
Non-blocking. Producer never waits, it just discards if the consumer can't keep up.

### 3. Rate Limiting (Proactive Backpressure)
Control the producer's speed before it even hits the buffer. The producer is limited to a fixed rate regardless of consumer speed.
```go
// go get golang.org/x/time/rate

package main

import (
	"context"
	"fmt"
	"time"

	"golang.org/x/time/rate"
)

func main() {
	// Allow 5 events per second, burst up to 10
	limiter := rate.NewLimiter(5, 10)

	for i := 1; i <= 20; i++ {
		// Wait until rate limiter allows
		if err := limiter.Wait(context.Background()); err != nil {
			fmt.Println("Error:", err)
			return
		}
		fmt.Printf("[Producer] Sent message %d at %s\n", i, time.Now().Format("15:04:05.000"))
	}
}
```
The producer is throttled to 5 messages/sec. Even if the consumer is slow, the producer won't flood the system.

### 4. Consumer Controlled (Pull-Based)
Instead of the producer pushing messages, the consumer **pulls** when it's ready. This is how Kafka works.
> Consumer ready → pulls N messages → processes → pulls N more

The consumer naturally controls the pace, it only asks for more work when it's done.

```go
// Kafka example: consumer controls the pace via polling
reader := kafka.NewReader(kafka.ReaderConfig{
	Brokers:  []string{"localhost:9092"},
	Topic:    "orders",
	GroupID:  "order-processors",
	MinBytes: 1,
	MaxBytes: 1e6, // max 1MB per fetch — limits how much work per pull
})

for {
	msg, err := reader.ReadMessage(context.Background())
	if err != nil {
		break
	}
	process(msg)
	// Next ReadMessage only called after processing — natural backpressure
}
```

### 5. Buffer with Monitoring + Alerts
Accept messages into a bounded buffer. If the buffer exceeds a threshold, alert ops and scale consumers, don't silently let it grow.
```go
package main

import (
	"fmt"
	"time"
)

const maxQueueSize = 100
const alertThreshold = 80 // 80% full

func monitorQueue(queue chan int) {
	ticker := time.NewTicker(1 * time.Second)
	for range ticker.C {
		usage := len(queue)
		pct := usage * 100 / maxQueueSize
		if pct >= alertThreshold {
			fmt.Printf("[ALERT] Queue at %d%% capacity (%d/%d) — scale consumers!\n",
				pct, usage, maxQueueSize)
		}
	}
}

func main() {
	queue := make(chan int, maxQueueSize)
	go monitorQueue(queue)
	// ... producer and consumer logic
}
```

## Backpressure in Real Systems

### Kafka
Kafka handles baackpressure naturally via the pull model. Consumer lag (the gap between latest offset and consumer's current offset) is the primary metric to monitor.

> Consumer lag = latest offset - consumer offset
>
> Lag growing → consumer failing behind → scale up consumers or reduce producer rate

Monitor with: `Kafka-consumer-groups.sh --describe` or Prometheus + Grafana

### RabbitMQ
RabbitMQ supports backpressure through:
- **Prefetch count** (`Qos`): limits unACKed messages per consumer
- **Queue lenth limit**: set `x-max-length` to cap the queue, extras are dropped or dead lettered
- **Publsiher confirms**: producer waits for broker to confirm receipt before sending more

```go
// Limit queue size — excess messages go to DLQ
args := amqp.Table{
	"x-max-length":           1000,
	"x-overflow":             "reject-publish", // reject new messages when full
	"x-dead-letter-exchange": "dlx",
}
ch.QueueDeclare("task_queue", true, false, false, false, args)

// Prefetch: consumer handles max 10 messages at a time
ch.Qos(10, 0, false)
```

## Comparison of Strategies
|Strategy|Data Loss?|Complexity|Best For|
|-|-|-|-|
|**Blocking**|No|Low|In process, batch jobs|
|**Dropping**|Yes|Low|Metrics, logs, live data|
|**Rate Limiting**|No|Medium|APIs, external producers|
|**Pull based**|No|Medium|Kafka consumers|
|**Buffer + Monitor**|No (until full)|Medium|Production queues|

## Common Pitfalls
- **Unbounded queues:** Never use an unbounded queue in production. Always set a max size, otherwise you're just delaying the crash.
```go
// Bad
queue := make(chan int) // unbounded goroutine leak risk

// Good
queue := make(chan int, 1000) // bounded — backpressure kicks in at 1000
```
- **Ignoring consumer lag:** A queue that's always growing is a ticking time bomb. Set up alerts on queue depth and consumer lag, don't find out you have a problem when the system crashes.
- **Backpressure that causes cascading timeouts:** If service A blocks waiting for service B (which is backed up), and service C is waiting for A, one slow service can cascade into a full system stall. Use timeouts + circuit breakers alongside backpressure.
```go
// Always pair blocking backpressure with a timeout
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()

select {
case queue <- msg:
    fmt.Println("Sent")
case <-ctx.Done():
    fmt.Println("Timeout — consumer too slow, dropping or retrying")
}
```
- **Treating all data the same:** Not all messages need the same backpressure strategy. High value transactional data (payments, orders) should block or retry. Low value telemetry (metrics, logs) can safely drop.

## Key Takeaways
- Backpressure is **not optional** in production, it's what keeps a system from falling over under load.
- The right strategy depends on whether you can afford data loss and how tightly coupled your services are.
- **Monitor queue depth and consumer lag**, these are the early warning signals.
- Always pair backpressure with **timeouts to avoid cascading stalls.**
- Pull-based systems (Kafka) handle backpressure more naturally than push-based ones (RabbitMQ, webhooks).