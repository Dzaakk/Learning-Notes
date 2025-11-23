Load Balancer Intro
====

## Load Balancer — The Internet's Traffic Cop
Now imagine a website like YouTube.\
One server can't handle millions of users — so they use **many servers** behind the scenes.

The **load balancer** sits in front and distributes incoming requests accross those servers to:
- Prevent any single server from overloading.
- Improve reliability (if one server fails, others handle the traffic). 
- Increase performance (serve users from the fastest or nearest machine).

### How it works:
1. The client sends as requet to the **load balancer's IP**.
2. The load balancer forwards that request to one of many **backend servers**.
3. When one server gets busy, the load balancer ashifts new requests to others.

There are two mian types:
- **Layer 4 (Transport level)**: balances based on IP & port (fast, simple).
- **Layer 7 (Application level)**: understands requests (e.g. HTTP path, headers) and can make smarter decisions.

### Analogy:
Imagine a busy resturant with multiple chefs. The load balancer is the head waiter who decides which chef gets which order — keeping the kitchen efficient.
