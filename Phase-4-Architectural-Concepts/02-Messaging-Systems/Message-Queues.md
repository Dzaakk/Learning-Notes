Message Queues
===

## What is a Message Queue?
A **message queue** is a form of asynchronous service to service communication. Messages are stored in the queuee until they are processed and deleted. Each message is processed only once, by a single consumer.

> Producer → [Queue: msg1, msg2, msg3] → Consumer

It decouples the sender (producer) from the receiver (consumer). They don't need to interact with each other at the same time.

## Core Concepts

- **Producer:** The service that creates and sends messages to the queue.
- **Consumer:** The service that reads and processes messages from the queue.
- **Message:** A unit of data sent through the queue. Can be plain text, JSON, binary, etc.
- **Queue:** A buffer that stores messages until the consumer is ready to processes them.

## Why Use Message Queues?
|Problem|Solution via Queue|
|-|-|
|Service B is slow|Producer doesn't have to wait, just enqueue|
|Traffic spikes|Queue absorbs burst, consumer processes at its own pace|
|Service B is down|Messages wait in queue, no data loss|
|Scaling|Add more consumers to drain the queue faster|

## Message Queue Patterns

### 1. Point to Point (P2P)
One producer, one consumer. Each message is delivered to **exactly one** consumer.

> Producer → [Queue] → Consumer A\
> Producer → [Queue] → Consumer B (won't receive the same message)

### 2. Competing Consumers
Multiple consumers reading from the same queue. Messages are distributed across consumers (load balanced).

![competing consumers](../../images/Phase-4-Architectural-Concepts/competing-consumers.png)

## Message Lifecycle
1. Producer sends message → queue stores it
2. Consumer polls / subscribes → receives message
3. Consumer processes message
4. Consumer sends ACK (acknowledgement) → queue deletes message
5. If no ACK (crash/failure) → message becomes visible again (retry)

**ACK is critical** it prevents message loss if a consumer crashes mid processing.

## Key Properties

**Durability**\
Messages are persisted to disk so they survive broker restarts.

**Ordering**
- **FIFO (First in, First out):** Messages are processed in order.
- **Best effort ordering:** No string guarantee, often faster but unordered.

**Delivery Guarantees**
|Type|Meaning|
|-|-|
|**At most once**|Message delivered 0 or 1 time. May lose messages.|
|**At least once**|Message delivered 1 or more times. May have duplicates.|
|**Exactly once**|Message delivered exactly once. Hardest to implement.|

**Visibility Timeout**\
When a consumer reads a message, it becomes "invisible" to others for a period. If not ACKed in time, it reappears for retry.

**Dead Letter Queue (DLQ)**\
Messages that fail repeatedly (exceed retry limit) are moved to a DLQ for inspection/debugging. Essential for production systems.
>[Main Queue] → Consumer fails 3x → [Dead Letter Queue] → Alert/Debug

## Example
### Simple Queue with Goroutines
This simulates a basic in memory message queue pattern in Go.
```go
package main

import (
	"fmt"
	"sync"
	"time"
)

type Message struct {
	ID      int
	Payload string
}

func producer(queue chan<- Message, wg *sync.WaitGroup) {
	defer wg.Done()
	for i := 1; i <= 5; i++ {
		msg := Message{ID: i, Payload: fmt.Sprintf("task-%d", i)}
		fmt.Printf("[Producer] Sending: %+v\n", msg)
		queue <- msg
		time.Sleep(100 * time.Millisecond)
	}
	close(queue)
}

func consumer(id int, queue <-chan Message, wg *sync.WaitGroup) {
	defer wg.Done()
	for msg := range queue {
		fmt.Printf("[Consumer-%d] Processing: %+v\n", id, msg)
		time.Sleep(200 * time.Millisecond) // simulate work
	}
}

func main() {
	queue := make(chan Message, 10) // buffered = acts like a queue
	var wg sync.WaitGroup

	// Start 1 producer
	wg.Add(1)
	go producer(queue, &wg)

	// Start 2 competing consumers
	wg.Add(2)
	go consumer(1, queue, &wg)
	go consumer(2, queue, &wg)

	wg.Wait()
	fmt.Println("All messages processed.")
}
```

**Output (approximate):**
> [Producer] Sending: {ID:1 Payload:task-1}\
> [Consumer-1] Processing: {ID:1 Payload:task-1}\
> [Producer] Sending: {ID:2 Payload:task-2}\
> [Consumer-2] Processing: {ID:2 Payload:task-2}\
> ...

### Dead Letter Queue Simulation
```go
package main

import "fmt"

type Message struct {
	ID      int
	Payload string
	Retries int
}

const maxRetries = 3

func process(msg Message) error {
	if msg.ID%2 == 0 {
		return fmt.Errorf("simulated failure for msg %d", msg.ID)
	}
	fmt.Printf("[OK] Processed: %s\n", msg.Payload)
	return nil
}

func main() {
	mainQueue := []Message{
		{ID: 1, Payload: "task-1"},
		{ID: 2, Payload: "task-2"},
		{ID: 3, Payload: "task-3"},
	}
	var dlq []Message

	for _, msg := range mainQueue {
		for msg.Retries < maxRetries {
			err := process(msg)
			if err == nil {
				break
			}
			msg.Retries++
			fmt.Printf("[RETRY %d] %v\n", msg.Retries, err)
		}
		if msg.Retries == maxRetries {
			fmt.Printf("[DLQ] Moving message %d to dead letter queue\n", msg.ID)
			dlq = append(dlq, msg)
		}
	}

	fmt.Printf("\nDead Letter Queue: %+v\n", dlq)
}
```

## Common Pitfalls
1. **Not handling idempotency:** With at-least-once delivery, duplicates can occur. Make your consumer idempotent. Processing the same message twice should have the same result as once.
2. **Ignoring DLQ:** DLQs are often set up but never monitored. Silent failures pile up.
3. **Queue depth growing unbounded:** If consumers are too slow, the queue grows. Set alerts on queue depth and scale consumers accordingly.
4. **Forgetting message TTL (Time To Live):** Old messages can become irrelevant. Set a TTL so stale messages are auto-expired.

## When to Use a Message Queue

### Use When:
- Tasks are time-consuming and can run in the background (e.g., sending emails, resizing images)
- You need to decouple services
- You expect bursty traffic
- You need retry/failure handling

### Don't use when:
- You need an immediate response (use direct HTTP/RPC instead)
- Message ordering is critical and hard to guarantee
- System complexity isn't justified (overkill for simple use cases)

## Popular Message Queue Sytems
|Tool|Best For|
|-|-|
|**RabbitMQ**|Task queues, complex routing, low latency|
|**Amazon SQS**|Simple managed cloud queue|
|**Redis (List/Streams)**|Lightweight, fast, in memory|
|**Kafka**|High throughput event streaming (more than just a queue)|
|**NATS**|Cloud-native, very low latency|