Monolith vs Microservices
===

## What is a Monolith?
A **monolith** is a single, unified application where all components (UI, business logic, data acess) are tightly coupled and deployed as one unit.

### Characteristics
- **Single codebase:** All code in one repository
- **Single deployment:** Deploy entire application together
- **Shared database:** One database for all functionality
- **Tight coupling:** Components depend on each other directly
- **Single process:** Runs as one process or application

### Example Structure
![monolith app structure](../../images/Phase-4-Architectural-Concepts/monolith-app-structure.png)

All deployed together as one application.

## What is Microservices?
**Microservices** architecture breaks application into small, independent services that communicate over network. Each service handles one specific business capability.

### Characteristics
- **Multiple codebases:** Separate repository per service
- **Independent deployment:** Deploy services individually
- **Distributed data:** Each service has its own database
- **Loose coupling:** Services communicate via APIs
- **Multiple processes:** Each service runs independently

### Example Structure
![microservices app structure](../../images/Phase-4-Architectural-Concepts/microservices-app-structure.png)

Each service deployed independently with its own database.

## Visual Comparison

### Monolith Architecture
![monolith structure](../../images/Phase-4-Architectural-Concepts/microservices-structure.png)

### Microservices Architecture
![microservices structure](../../images/Phase-4-Architectural-Concepts/microservices-structure.png)