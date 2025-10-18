Internet Identity Pack
==================

Topics on this file : **IP Address, DNS, Load Balancer**

## 1. IP Address (The Internet's Home Address)
Every device on the internet — your phone, laptop, or a ginat server has an **IP address**, which acts like its *home address* so data knows where to go.

There are **two main versions**:

**IPv4 (old but still dominant)**
- Looks like: `192.168.1.10` 
- 4 groups of numbers (0-255)
- About 4 billion possible addresses

**IPv6 (newer, bigger, future-proof)**
- Looks like: `2001:0db8:85a3:0000:0000:8a2e:0370:7334`
- Way more addresses (~340 undecillion)

### How it works:
When a client sends a request, it needs the **destination IP** — like sending a letter to a street address.\
Routers then pass the packet along the internet until it reaches that IP.

### You should know:
- Public IP = visible to the internet (like your house's front door).
- Private IP = used inside local networks (like rooms in your house).
- NAT (Network Address Translation) lets many devices share one public IP (used by your home Wi-Fi router).

## 2. DNS (Domain Name System) — The Internet's Phone Book
Humans remember names (like `www.google.com`) not numbers like `192.168.1.10`.\
DNS (Domain Name System) translates those names into IP address.

### Process:
1. You type `instagram.com`
2. Your computer asks a **DNS resolver**: "Hey, what's the IP for this domain?"
3. The resolver checks a hierarchy of DNS servers:
    - Root -> `.com` -> `instagram.com` -> final IP.
4. Once found, it's cached locally so you don't have to look it up again next time.

### Why it matters:
- Without DNS, you'd have to memorize IP addresses for every site.
- DNS also helps with **load balancing**, **failover**, and **security**.

### Analogy:
Think of DNS like the phone book. You look up a person's name (domain) and get their number (IP).

## 3. Load Balancer — The Internet's Traffic Cop
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
