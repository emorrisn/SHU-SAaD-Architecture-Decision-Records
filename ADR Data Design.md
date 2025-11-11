# Data Model for Multi-Tenancy and Auditability

## Context and Problem Statement

The Complaint Management System (CMS) must store and manage sensitive complaint data for multiple, large, and distinct enterprise clients (e.g. banks and telecom companies). The system architecture must strictly enforce data isolation; data belonging to one tenant (e.g. NatWest) must never be accessible or visible to another tenant (e.g. Barclays), as specified in the case study.

Furthermore, the system must support the full lifecycle of a complaint, from logging to resolution. This requires a robust, auditable trail of all actions taken on a complaint to meet compliance standards and to enable managers (like Mark Bennett) to monitor performance.

The challenge is:

How do we design the data model to enforce strict multi-tenant data isolation while simultaneously providing strong data integrity and a clear, auditable activity for all complaint-related actions?

## Considered Options

1. **Database per Tenant:** Provision a completely separate, physically isolated database for each tenant. This offers the strongest possible isolation but has high operational and cost overhead.
2. **Shared Database, Shared Schema (Chosen):** Use a single database where all tenants share the same tables. Data is isolated by adding a TenantID "discriminator" column to key tables (Complaint, User).
3. **NoSQL Document Model:** Use a document database (e.g. MongoDB) and store each complaint as a rich JSON document. Tenant isolation would be managed by a TenantID field within each document.

## Decision Outcome

Chosen option: "Shared Database, Shared Schema (Relational Model)". This decision is visualized in the Entity-Relationship Diagram (Figure 6).

This model was chosen because it provides the best balance of multi-tenant security, data integrity, and operational efficiency for the system's requirements:

* **Enforced Logical Isolation:** The Tenant entity is the root of this model. Every User and every Complaint record contains a mandatory foreign key (TenantID). All application-layer data queries must be filtered by this TenantID, thereby ensuring that users from one tenant can only ever see their own organisation's data.
* **Data Integrity and Structure:** A relational model (like PostgreSQL, as identified in the stack) is chosen over NoSQL. The system's data is highly structured and relational (Users handle Complaints, Complaints have Activity). Using an ACID-compliant database is critical for ensuring transactional integrity, especially for the types of sensitive financial complaints ("Report false Transaction") identified in the case study.
* **Auditability by Design:** Instead of overwriting data, the model uses a dedicated ComplaintActivity table. This table creates an immutable, append-only log of every Action (e.g. 'Created', 'Assigned', 'StatusChange') performed on a complaint, linking who did it (UserID) and when (Timestamp). This directly supports the managerial and compliance requirements for auditing the complaint lifecycle.
* **Normalisation and Efficiency:** The model is normalised to reduce data redundancy. Attachment and ComplaintMessage are separated from the core Complaint entity, which keeps the primary complaint table lean and improves query performance for dashboards and task queues.

## Consequences

### Advantages (Case Study Specific)

* **Meets Core Multi-Tenancy Requirement:** The TenantID discriminator column directly implements the case study's requirement to isolate data between clients like NatWest and Barclays.
* **Full Lifecycle Audit:** The ComplaintActivity entity provides a complete, auditable log, which is essential for managers monitoring resolution times and for meeting strict banking/telecom compliance.
* **Transactional Integrity:** Using a relational/ACID database ensures that critical operations, like logging a "stollen debit card" report, are handled with guaranteed consistency.
* **Supports Reporting:** The structured SQL model allows for powerful and flexible queries, which is necessary for the Help Desk Manager's analytics dashboard and comprehensive reports.

### Advantages (General)

* **Cost-Effective:** A single database is significantly cheaper to host, manage, and back up compared to a database-per-tenant model.
* **Simplified Maintenance:** A single, shared schema is easier to update, migrate, and maintain over time.
* **Clear, Proven Model:** The ERD is a well-understood, standard approach, making it easier for new developers to understand the system.

### Disadvantages (Case Study Specific)

* **Security Reliant on Application:** The entire data isolation model depends on the application code never failing to filter by TenantID. A single bug in a query could lead to a catastrophic data breach across tenants.
* **"Noisy Neighbour"** Risk: In a shared database, one tenant with extremely high traffic (e.g., a massive bank) could consume a disproportionate amount of database resources, potentially slowing down performance for other tenants.

### Disadvantages (General)

* **Schema Rigidity:** Changes to the data model (e.g., adding new fields to the Complaint table) require a formal schema migration, which is less flexible than a NoSQL model.
* **Complex Sharding:** If the single database grows too large, scaling it becomes more complex (e.g., manual sharding) than the horizontal scaling often native to NoSQL.
