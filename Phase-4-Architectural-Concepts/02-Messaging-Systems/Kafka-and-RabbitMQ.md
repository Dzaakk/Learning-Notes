Kafka and RabbitMQ
===
Two of the most widely used messaging systems, but built for different purposes. Understanding when to use which is the key.

## High Level Comparison
||Kafka|RabbitMQ|
|-|-|-|
|**Type**|Distributed event log / streaming|Message broker (queue based)|
|**Model**|Pull based (consumers pull)|Push based (broker pushes)|
|**Message retention**|Retained for a configurable time (days/weeks)|Deleted after ACK|
|**Ordering**|Guaranteed within a partition|Guaranteed within a queue|
|**Throughput**|Very high(millions msg/sec)|High (tense of thousands msg/sec)|
|**Use case**|Event streaming, audit logs, data pipelines|Task queues, RPC, job workers|
|**Consumers**|Multiple independent consumers, each replay from offset|Competing consumers share the queue|
|**Complexity**|Higher (needs ZooKeeper or KRaft)|Lower, easier to set up|


## Apache Kafka

### Core Idea
Kafka is a **distributed, append only log**. Producers write events to topics. Consumers read from topics independently at their own pace by tracking an **offset** (position in the log).

![kafka core idea](../../images/Phase-4-Architectural-Concepts/kafka-core-idea.png)

Consumers don't delete messages, they just advance their offset. This means **multiple independent consumers can read the same data**.

### Key Concepts
- **Topic:** A named stream of records. Like a category/channel. e.g., `orders`, `user-events`.
- **Partition:** Topics are split into partitions for parallelism. More partitions = more thorughput. Messages within a partition are strictly ordered.
- **Offset:** An interger index pointing to a message's position in a partition. Consumers track their own offset. Kafka doesn't track it for them (stored in `__consumer_offset`) topic or externally.
- **Consumer Group:** A group of consumers that together consume a topic. Kafka assigns each partition to exactly one consumer in the group, enabling parallel processing without duplication.

> Topic (3 partitions) + Consumer Group (3 consumers):
> 
> Partition 0 → Consumer A\
> Partition 1 → Consumer B\
> Partition 2 → Consumer C

if you have more consumers than partitions, some consumers sit idle. Scale partitions, not just consumers.

- **Broker:** A single Kafka server. A Kafka cluster has multiple brokers for fault tolerance and load distribution.
- **Replication Factor:** Each partition is replicated across N brokers. If one broker dies, another takes over. Typically set to 3 in production.
- **Retention:** Messages are kept for a configured time (e.g., 7 days) or size regardless of whether they've been consumed this enables **replay**, reprocessing historical data.

### Kafka Message Flow
1. Producer sends message to Topic → routed to a Partition (by key hash or round robin)
2. Broker writes to the partition log (append only)
3. Replicated to follower brokers
4. Consumer polls for new messages using its offset
5. After processing, consumer commits the offset

**Key decision: when to commit offset?**
- Commit before processing → **at-most-once** (risk losing messages on crash)
- Commit after processing → **at-least-once** (risk duplicates on crash)
- Use idempotent processing + transactions → **exactly-once**

### Go Example: Kafka Producer & Consumer
```go
// go get github.com/segmentio/kafka-go

package main

import (
	"context"
	"fmt"
	"log"
	"time"

	"github.com/segmentio/kafka-go"
)

// Producer: write messages to Kafka
func produce() {
	writer := kafka.NewWriter(kafka.WriterConfig{
		Brokers:  []string{"localhost:9092"},
		Topic:    "orders",
		Balancer: &kafka.LeastBytes{},
	})
	defer writer.Close()

	for i := 1; i <= 5; i++ {
		err := writer.WriteMessages(context.Background(),
			kafka.Message{
				Key:   []byte(fmt.Sprintf("order-%d", i)),
				Value: []byte(fmt.Sprintf(`{"order_id": %d, "item": "widget"}`, i)),
			},
		)
		if err != nil {
			log.Fatal("failed to write:", err)
		}
		fmt.Printf("[Producer] Sent order-%d\n", i)
		time.Sleep(200 * time.Millisecond)
	}
}

// Consumer: read messages from Kafka using a consumer group
func consume() {
	reader := kafka.NewReader(kafka.ReaderConfig{
		Brokers:  []string{"localhost:9092"},
		Topic:    "orders",
		GroupID:  "order-processors", // consumer group
		MinBytes: 1,
		MaxBytes: 10e6,
	})
	defer reader.Close()

	for {
		msg, err := reader.ReadMessage(context.Background())
		if err != nil {
			log.Fatal("failed to read:", err)
		}
		fmt.Printf("[Consumer] offset=%d key=%s value=%s\n",
			msg.Offset, string(msg.Key), string(msg.Value))
		// ReadMessage auto-commits offset after successful read
	}
}

func main() {
	go produce()
	time.Sleep(500 * time.Millisecond)
	consume()
}
```

### When to Use Kafka

**Use when:**
- Event streaming / event sourcing
- Audit logs (need message replay)
- Data pipelines (feeding multiple downstream systems from one stream)
- High throughput ingestion (IoT, clickstreams, metrics)
- Multiple independent consumers need to read the same events

**Don't use when:**
- Not ideal for simple task queues
- Overkill for low volume systems
- Harder to implement request/reply (RPC) patterns

## RabbitMQ

### Core Idea
RabbitMQ is a **message broker** built around the AMQP protocol. Producers send messages to an **Exchange**, which routes them to **Queues** based on routing rules. Consumers subscribe to queues.

![rabitmq core idea](../../images/Phase-4-Architectural-Concepts/rabbitmq-core-idea.png)

Unlike Kafka, once a message is ACKed by a consumer, it's **gone**. RabbitMQ is designed for task distribution, not event streaming.

### Key Concepts
- **Exchange:** Receives messages from producers and routes them to queues. There are 4 types:

|Exchange Type|Routing Logic|Use Case|
|-|-|-|
|**Direct**|Route by exact routing key|Task queues, point to point|
|**Fanout**|Broadcast to all bound queues|Notifications, pub/sub|
|**Topic**|Route by pattern matching (`*.error`,`order.#`)|Flexible routing|
|**Headers**|Route by message headers|Rarely used|

- **Queue:** Stores messages until a consumer processes them. Supports durability (survives restart) and exclusivity.
- **Binding:** The link between an Exchange and a Queue, with an optional routing key.
- **Channel:** A lightweight virtual connection inside a TCP connection. Use `on channel per goroutine, don't share channels across goroutines.

**ACK / NACK:**
- `ACK`: Message processed successfully, remove from queue.
- `NACK`/`Reject`: Processing failed. Can requeue or discard (→ DLQ).

- **Prefetch Count:** Limits how many unACKed messages a consumer holds at once. Prevents a slow consumer from hoarding messages.
```go
channel.Qos(prefetchCount, 0, false) // send max N messages at a time
```

### RabbitMQ Message Flow
1. Producer connects → creates Channel → publishes to Exchange with routing Key
2. Exchange matches routing key → forward to bound Queue(s)
3. Consumer subscribes to Queue → receives message (push)
4. Consumer processes → sends ACK
5. RabbitMQ removes message from queue

### Go Example: RabbitMQ Producer & Consumer
```go
// go get github.com/rabbitmq/amqp091-go

package main

import (
	"fmt"
	"log"

	amqp "github.com/rabbitmq/amqp091-go"
)

func failOnError(err error, msg string) {
	if err != nil {
		log.Fatalf("%s: %s", msg, err)
	}
}

// Producer
func produce(ch *amqp.Channel) {
	q, err := ch.QueueDeclare(
		"task_queue", // name
		true,         // durable
		false,        // auto-delete
		false,        // exclusive
		false,        // no-wait
		nil,
	)
	failOnError(err, "Failed to declare queue")

	body := `{"task": "send_email", "to": "user@example.com"}`
	err = ch.Publish(
		"",     // default exchange (direct to queue by name)
		q.Name, // routing key = queue name
		false,
		false,
		amqp.Publishing{
			DeliveryMode: amqp.Persistent, // survive broker restart
			ContentType:  "application/json",
			Body:         []byte(body),
		},
	)
	failOnError(err, "Failed to publish")
	fmt.Printf("[Producer] Sent: %s\n", body)
}

// Consumer
func consume(ch *amqp.Channel) {
	q, err := ch.QueueDeclare("task_queue", true, false, false, false, nil)
	failOnError(err, "Failed to declare queue")

	// Prefetch: only 1 unACKed message at a time per consumer
	ch.Qos(1, 0, false)

	msgs, err := ch.Consume(
		q.Name,
		"",    // consumer tag (auto-generated)
		false, // auto-ack = false (manual ACK)
		false, false, false, nil,
	)
	failOnError(err, "Failed to register consumer")

	forever := make(chan struct{})
	go func() {
		for d := range msgs {
			fmt.Printf("[Consumer] Received: %s\n", d.Body)
			// ... do work ...
			d.Ack(false) // ACK after successful processing
		}
	}()
	fmt.Println("[Consumer] Waiting for messages...")
	<-forever
}

func main() {
	conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
	failOnError(err, "Failed to connect")
	defer conn.Close()

	ch, err := conn.Channel()
	failOnError(err, "Failed to open channel")
	defer ch.Close()

	produce(ch)
	consume(ch)
}
```

### Go Example: Topic Exchange (Pattern Routing)
```go
// Declare a topic exchange
ch.ExchangeDeclare("logs", "topic", true, false, false, false, nil)

// Producer: publish with routing key
ch.Publish("logs", "order.created", false, false,
    amqp.Publishing{Body: []byte(`{"order_id": 42}`)})

ch.Publish("logs", "order.failed", false, false,
    amqp.Publishing{Body: []byte(`{"order_id": 43}`)})

// Consumer A: subscribe to all order events
q, _ := ch.QueueDeclare("", false, true, true, false, nil)
ch.QueueBind(q.Name, "order.*", "logs", false, nil) // matches order.created, order.failed

// Consumer B: subscribe only to failures
q2, _ := ch.QueueDeclare("", false, true, true, false, nil)
ch.QueueBind(q2.Name, "*.failed", "logs", false, nil) // matches order.failed, payment.failed
```

### When to Use RabbitMQ

**Use when:**
- Task queues / background job workers
- Request/reply (RPC) patterns
- Complex routing logic (topic, fanout, direct)
- When you need push-based delivery with low latency
- Simpler operational setup than Kafka

**Don't use when:**
- Not ideal for event replay / audit logs (messages are deleted after ACK)
- Lower throughput ceiling compared to Kafka
- Multiple independent consumers reading the same data = need separate queues per consumer

## Side by Side: Which to Choose?
> Need to process the same vent in multiple independent services?\
> → Kafka (consumer groups can all read independently)
>
> need to distribute tasks across worker instances?\
> RabbitMQ (competing consumers on one queue)
>
> Need to replay historical events?\
> → Kafka (log retention)
>
> Need complex routing (topic patterns, fanout)?\
> → RabbitMQ (Exchange types)
>
> Expecting millions of events/sec?\
> → Kafka
>
> Simple background jobs email sendubg, etch.?\
> → RabbitMQ

## Common Pitfalls

### Kafka
- Too few partitions → bottleneck, can't scale consumers
- Not handling offset commits carefully → duplicate processing or message los
- Ignoring consumer lag → queue builds up silently, onsumer fall behind

### RabbitMQ
- Sharing a channel across goroutines → race conditions (use one channel per goroutine)
- Not setting `Durable: true` + `Persistent` delivery mode → messages lost on restart
- No perfetch limit → slow consumers hoard messages, other starve
- Not setting up a DLQ → failed messages silently disappear