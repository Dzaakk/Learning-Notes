Distributed Systems Basics
===

## What is a Distributed System?
A distributed system is a collection of independent computers that work together by communicating over a network to achieve a common goal. Each computer (node) has its own memory and processor, and they coordinate only by passing messages.

**Example:** A web application with separate database servers, application servers, and cache servers working together.

## Why Use Distributed Systems?

### Scalability
Single machines have limits. Distributed systems let you scale horizontally by adding more machines, which is often cheaper than buying one massive machine.

**Example:** Handle 1 million requests/second by using 100 servers instead of one impossibly expensive super-computer.

### Reliability and Fault Tolerance
If one machine fails, others continue working. Redundancy across multiple nodes prevents total system failure.

**Example:** Store data on 3 servers so if 1 fails, you still have 2 copies.

### Geographic Distribution
Place servers closer to user worldwide to reduce latency.

**Example:** Netflix has servers in multiple continents to serve local users quickly.

### Performance Through Parallelism
Process tasks simultaneously across many machines.

**Example:** Analyze 1 billion images by distribting work across 1000 machines.