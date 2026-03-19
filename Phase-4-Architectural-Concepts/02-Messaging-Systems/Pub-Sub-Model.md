Pub/Sub Model
===

## What is Pub/Sub?
**Publish/Subscribe** is a messaging pattern where:
- **Publishers** send messages to a **topic/channel**, they don't know who will receive it.
- **Subscribers** express interest in a topic and receive all messages published to it, they don't know who sent it.

The key trait: **publishers and subscribers are fully decoupled.** Neither knows about each other.

![pub sub](../../images/Phase-4-Architectural-Concepts/pub-sub.png)

Every subscriber gets its own copy of the message (broadcast), unlike a task queue where only one consumer gets each message.

## Pub/Sub vs Message Queue

||Pub/Sub|Message Queue|
|-|-|-|
|**Delivery**|Broadcast, all subscribers receive the message|Point to point, one consumer gets the message|
|**Coupling**|Publishers doesn't know subscribers|Producer targets a specific queue|
|**Use case**|Event notifications, fan-out|Task distribution, job workers|
|**Message retention**|Usually not retained after delivery|Retained until ACKed|

> In practice, systems like Kafka and RabbitMQ (fanout exchange) **implement** pub/sub. Pub/Sub is a pattern, not a specific tool.

## Core Concepts
- **Topic:** A named channel that publishers write to and subscribers listen on. E.g., `user.registered`, `order.complete`.
- **Subscription:** A subscriber's registered interest in a topic. Each subscription typically gets its own copy of every message.
- **Fan out:** One message → delivered to N subscribers simultaneously. THis is the defining behavior of pub/sub.
- **Filtering:** Some pub/sub systems let subscribers filter messages by attributes, so they only receive relevant events.

>Topic: payments\
  Subscriber A filter: amount > 1000  → only gets high-value transactions\
  Subscriber B filter: status = failed → only gets failed payments\
  Subscriber C (no filter)             → gets everything