# Scalability and Multi-tenancy for the Complaint Management System

## Context and Problem Statement

The Complaint Management System (CMS) needs to handle thousands of simultaneous user complaints from large banking and telecom companies. The system must be available 24/7 and support a growing user base, with an expected 10% annual increase in users.

The challenge is:

> How can we architect the CMS to ensure high availability and scalability while managing distinct data and users for multiple large client organisations (multi-tenancy)?

## Considered Options

* **Monolithic Architecture:** A single, tightly-coupled application handling all concerns (UI, business logic, data).
* **Three-Tiered Architecture:** Separates the application into presentation, business/application, and data tiers.
* **Microservices Architecture:** Decomposes the application into a collection of independently deployable, loosely coupled services, each responsible for a specific business capability.

## Decision Outcome

Chosen option: **"Three-Tiered Architecture"**, because it provides a robust, scalable foundation that aligns with the CMS's needs for a clear separation of concerns. This approach is highly effective for the Complaint Management System's requirements as it:

* **Supports Scalability:** By separating the business logic and presentation layers, we can independently scale the application tier to handle the projected 10% annual increase in users without affecting the database. This is crucial for maintaining 24/7 availability under growing load.
* **Simplifies Multi-Tenancy:** The clear isolation of the data tier allows for an efficient and secure multi-tenant design. We can partition the database to logically separate each client's data, ensuring that user complaints from one company (e.g. A banking company) are kept distinct and secure from another (e.g. A telecom company).
* **Improves Maintainability:** The distinct layers make it easier for separate teams to work on the UI, business logic, and data management independently. This improves development velocity and simplifies the process of making updates or bug fixes.
* **Facilitates Evolution:** This architecture provides a solid base for future growth. While it's not as granular as a microservices architecture, it allows for a gradual transition to a more fine-grained approach if specific functionalities of the CMS require more specialised scaling in the future (e.g. Adding a chatbot or a specific analytics service).

### Consequences

#### Good

* The separation of tiers makes the system more maintainable and easier to debug, as issues can be isolated to a specific layer.
* The application layer can be scaled out independently (e.g. By adding more web servers) to handle concurrent user requests without affecting the data layer.
* It facilitates the development of a multi-tenant solution where each client's data is logically separated within the database tier, ensuring data integrity and security.

#### Bad

* This architecture can become a performance bottleneck if communication between tiers is not optimised, especially with a high volume of requests.
* While more flexible than a monolith, it is less granular than microservices, making it harder to scale specific features without scaling the entire application tier.
