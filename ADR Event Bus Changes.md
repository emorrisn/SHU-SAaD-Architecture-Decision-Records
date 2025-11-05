# Complaint Service Write Path Transactional Flow (Hybrid Persistence)  

## Context and Problem Statement  

The Complaint Service (CMS) must manage the lifecycle of a complaint with strong transactional integrity. The initial design favored a pure Event-Driven Architecture, where state changes would be pushed directly to the Kafka message bus (Transactional Outbox pattern). However, two critical requirements conflict with this approach:

1. **Immediate Status Consistency:** Consumers and Help Desk Agents require the status of a newly logged complaint to be immediately accurate upon submission.
2. **Multi-Tenancy and Security:** Core business rules and multi-tenancy checks must be enforced synchronously during the write operation to prevent data leakage and ensure security.

Relying solely on an asynchronous outbox introduces complexity and potential delay in read operations via the Query Service, violating the need for immediate, reliable status updates.

## Decision Outcome

The Complaint Service will adopt a Hybrid Persistence Model:

1. **Synchronous Write:** The Command Handlers will commit the state change (the complaint record) synchronously to the PostgreSQL Database via the Complaint Repository (JPA). The database remains the primary source of truth for the complaint's current state.
2. **Asynchronous Notification:** Immediately following the successful database transaction commit, the Complaint Event Producer (Spring Kafka Template) will publish the corresponding domain event (e.g. ComplaintLogged).  

### Consequences

#### Advantages (Case Study Specific)

* **Immediate Complaint Status Updates:** Consumers and Help Desk Agents receive real-time confirmation when a complaint is logged, aligning with the case study’s requirement for responsiveness and accuracy.
* **Enforced Multi-Tenant Security:** Tenant and authorization checks occur before the database commit, ensuring that no cross-tenant data exposure occurs between clients (e.g. NatWest and Vodafone).
* **Improved User Confidence:** The immediate feedback loop supports user trust in large enterprise contexts, where customers expect reliable, real-time updates.
* **Simplified SLA Compliance:** Synchronous persistence helps meet service-level commitments for logging and confirming complaints within strict response times.

#### Advantages (General)

* **Immediate Read Consistency:** The Query Service can instantly reflect the latest complaint state without waiting for asynchronous event propagation.
* **Simplified Transaction Management:** Relies on database ACID properties for rollback and consistency, reducing complexity in error handling.
* **Balanced Scalability:** Downstream services (e.g. Notifications, Reporting) remain loosely coupled through Kafka, preserving microservice scalability and resilience.
* **Clear System of Record:** Maintains a single authoritative source of truth, simplifying auditing and debugging.

#### Disadvantages (Case Study Specific)

* **Database Dependency for Availability:** The Complaint Service’s write path is now tied directly to PostgreSQL uptime — any outage affects complaint logging, which may challenge the 24/7 availability target.
* **Potential Performance Bottleneck for Large Clients:** As tenants like HSBC or Vodafone scale up, database contention may increase, requiring optimisations such as sharding or caching.

#### Disadvantages (General)

* **Increased Write Latency:** Synchronous commits extend overall command execution time before acknowledgment.
* **Reduced Architectural Purity:** Mixing synchronous and asynchronous flows introduces added coordination complexity.
* **Operational Overhead:** Monitoring both synchronous DB transactions and asynchronous event publishing requires more sophisticated observability setups.
* **Tighter Coupling to Data Layer:** The write path becomes more dependent on database health and connection stability.
