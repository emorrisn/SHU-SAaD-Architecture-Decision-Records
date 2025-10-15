# Complaint Service Write Path Transactional Flow (Hybrid Persistence)  

## Context and Problem Statement  

The Complaint Service (CMS) must manage the lifecycle of a complaint with strong transactional integrity. The initial design favored a pure Event-Driven Architecture, where state changes would be pushed directly to the Kafka message bus (Transactional Outbox pattern). However, two critical requirements conflict with this approach:  

1. Immediate Status Consistency: Consumers and Help Desk Agents require the status of a newly logged complaint to be immediately accurate upon submission.
2. Multi-Tenancy and Security: Core business rules and multi-tenancy checks must be enforced synchronously during the write operation to prevent data leakage and ensure security.

Relying solely on an asynchronous outbox introduces complexity and potential delay in read operations via the Query Service, violating the need for immediate, reliable status updates.  

## Decision Outcome

The Complaint Service will adopt a Hybrid Persistence Model:

1. Synchronous Write: The Command Handlers will commit the state change (the complaint record) synchronously to the PostgreSQL Database via the Complaint Repository (JPA). The database remains the primary source of truth for the complaint's current state.
2. Asynchronous Notification: Immediately following the successful database transaction commit, the Complaint Event Producer (Spring Kafka Template) will publish the corresponding domain event (e.g., ComplaintLogged).  

## Consequences

### Good

* Immediate Read Consistency: The Complaint Query Service can immediately read the updated state from the database, satisfying the Consumer's need for instant status confirmation.
* Simplified Transaction Management: Leverages the database's native ACID properties, simplifying error handling and rollback in the core write path.
* Maintained Loose Coupling: Downstream services like the Notification Service and Reporting Service remain decoupled and are still updated asynchronously via Kafka.

### Bad

* Increased Write Latency: The synchronous commit adds the database transaction time to the overall command execution time.
* Tight Database Coupling: The availability of the Complaint Service's write path is now directly dependent on the availability of the PostgreSQL database.