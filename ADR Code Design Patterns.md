# Repository and Factory Patterns for Complaint Microservice

## Context and Problem Statement

The internal Complaint Service write path is complex, involving: (1) multi-tenant authorization, (2) synchronous database commit, and (3) asynchronous event publishing (Hybrid Persistence). The service must maintain 24/7 availability and guarantee data integrity throughout the complaint lifecycle (logging, allocation, resolving, closure).

The core challenges within the code are:

1. **Persistence Decoupling:** How to separate the business logic (LogComplaintHandler) from the specific data access technology (PostgreSQL via EF Core) to ensure testability and compliance with the Dependency Inversion Principle (DIP).
2. **Domain Integrity:** How to ensure that the core domain entity (Complaint) is always created and validated correctly, regardless of where the data originates.

## Considered Options

* **Direct Data Access:** Inject the Entity Framework Core DbContext directly into the LogComplaintHandler.
* **Static Factory/Constructor:** Use a static Complaint.Create() method or a simple public constructor for entity creation.
* **Repository Pattern and Handler as Implicit Factory (Chosen):** Implement the structural Repository Pattern and delegate complex object creation to the Command Handler (acting as a Factory Method).

## Decision Outcome

Repository Pattern and Handler as Implicit Factory (Chosen): Implement the structural Repository Pattern and delegate complex object creation to the Command Handler (acting as a Factory Method).

This is implemented as seen in the C4 Level 4 Structural and Behavioural Diagrams:

1. **Repository Pattern:** The LogComplaintHandler depends on the IComplaintRepository interface for all persistence operations.
2. **Factory Method (Handler):** The LogComplaintHandler is responsible for receiving the LogComplaintCommand DTO and explicitly calling the constructor/factory method for the Complaint entity, ensuring all necessary business rules are applied at creation.

### Consequences

#### Good (Case Study Specific)

* **Enhanced Reliability for 24/7 Operations (Repository Pattern):** The use of the Repository Pattern is vital because it allows the LogComplaintHandler to be unit tested independently. This rigorous testing minimizes defects in the core business logic, directly supporting the non-functional requirement for 24/7 system availability.
* **Centralized Complaint Lifecycle Integrity (Factory Method):** The Factory Method guarantees that a new complaint is always initialized correctly (e.g. with a unique ID and the initial status='Logged'). This is crucial for managing the full complaint lifecycle (logging, allocation, resolving, closure) mentioned in the case study brief.
* **Decoupling (Repository Pattern):** The business logic is independent of the database stack, ensuring long-term flexibility and maintainability for the platform supporting large enterprise clients.

#### Good (General)

* **Enhanced Testability (Repository Pattern):** The LogComplaintHandler can be unit tested by mocking the IComplaintRepository interface (e.g. using Moq in .NET). This eliminates the need for expensive database setups during local development and CI/CD pipelines.

* **Centralized Object Integrity (Factory Method):** By making the LogComplaintHandler (or an internal factory) responsible for entity creation, we guarantee that the Complaint entity is created in a valid state, enforcing all invariants (like automatically setting status='Logged'). This is a key principle of Domain-Driven Design (DDD).

* **Adherence to DIP:** The reliance on the IComplaintRepository interface over the concrete implementation strictly follows the Dependency Inversion Principle of SOLID.

#### Bad

* **Increased Amount of Code:** Implementing the Repository Pattern introduces two extra files (an interface and a concrete class) for data access.
* **Initial Complexity:** Developers new to the project must understand the strict separation between the Handler, Repository, and Entity, which requires a slight initial learning curve.
