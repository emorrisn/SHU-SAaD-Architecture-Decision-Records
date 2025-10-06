# Extensibility for Chatbot Integration

## Context and Problem Statement

The Complaint Management System (CMS) must be designed to be extensible to accommodate the future integration of new functionalities, specifically a chatbot. The current architecture needs to be flexible enough to allow for the seamless addition of this feature without a major system overhaul.

The challenge is:

> How can we architect the CMS to support the integration of a future chatbot, ensuring minimal disruption to the existing system while providing the necessary data and services for the chatbot to function effectively?

## Considered Options

* **Monolithic Architecture with Feature Toggles:** Integrate the chatbot code directly into the main application, but keep it dormant using a feature toggle until it's ready for release.
* **Service-Oriented Architecture (SOA):** Develop the chatbot as a new, independent service that communicates with the core CMS via a set of well-defined APIs.
* **Serverless Architecture:** Implement the chatbot's logic using serverless functions (e.g. AWS Lambda, Google Cloud Functions) that are triggered by user input and interact with the main system via APIs.

## Decision Outcome

Chosen option: **"Service-Oriented Architecture (SOA)"**, because it directly addresses the need for extensibility by treating the chatbot as an independent service. This approach:

* **Minimises Disruption:** The chatbot can be developed and deployed separately from the main CMS application, preventing the need for a full system re-deployment or complex code merges.
* **Encapsulates Logic:** The chatbot service will encapsulate its own business logic for handling user queries, but it will rely on the core CMS services for data access and complaint management actions, such as logging a new ticket or fetching an existing one.
* **Promotes Reusability:** The APIs created for the chatbot can also be reused by other new features or third-party integrations in the future, promoting a more flexible and adaptable system.

### Consequences

#### Good

* It allows for a clear separation of concerns, making the chatbot feature easier to develop, test, and maintain independently.
* The chatbot's performance can be monitored and scaled separately from the main CMS application, ensuring that its usage doesn't negatively impact core system performance.
* The API-driven approach simplifies future integrations, such as adding a new mobile app or a voice assistant.

#### Bad

* Adds a layer of complexity due to the need for inter-service communication and management of a distributed system.
* Performance can be impacted by network latency between the chatbot service and the core CMS APIs, especially for data-intensive operations.
