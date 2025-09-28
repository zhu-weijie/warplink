#### **Custom Alias Creation**

**Problem:**
To satisfy **FR3 (Custom URL Alias)**, registered users require the ability to create predictable, branded, and memorable short links (e.g., `warplink.com/my-event`) instead of relying on randomly generated keys. The system must prevent alias collisions (two users claiming the same alias) and associate the created link with the authenticated user for future management.

**Solution:**
The architecture will be extended to handle a new authenticated "write path" for creating links.

1.  A new, protected API endpoint (e.g., `POST /links/custom`) will be created. It will only be accessible to requests bearing a valid JWT, as enforced by the API Gateway Authorizer from Issue #7.
2.  The authenticated user will submit their `long_url`, the desired `custom_alias`, and their JWT.
3.  The `WarpLink Service` will receive the request. The user's unique ID (`user_id`) will be available from the validated JWT payload.
4.  The service will perform an **atomic, conditional write** operation to the `URL Database`. It will attempt to insert the new record (`short_key` = `custom_alias`) with an associated `user_id`, but only on the condition that an item with that `short_key` does not already exist.
5.  **Success:** If the write operation succeeds, it means the alias was available. The service returns an `HTTP 201 Created` response.
6.  **Failure:** If the write operation fails due to the condition check (the alias is already taken), the service returns an `HTTP 409 Conflict` error to inform the user that the alias is unavailable.

This approach is highly reliable and prevents race conditions where multiple users might attempt to claim the same alias simultaneously.

**Trade-offs:**

*   **Conflict Resolution (Conditional Write vs. Read-then-Write):**
    *   **Pros (Conditional Write):** This is the architecturally correct choice. Using the database's native conditional operation (e.g., DynamoDB's `ConditionExpression`) makes the check-and-set operation atomic. It is completely immune to race conditions and guarantees data integrity.
    *   **Cons (Read-then-Write):** A naive "check if exists, then write" approach is fundamentally flawed. It creates a time window between the read and the write where another process could claim the alias, leading to data corruption or silent failures. This approach is not acceptable for a reliable system.

*   **Schema Modification for Ownership:**
    *   **Pros:** Adding a `user_id` attribute to the `links` table is a necessary schema evolution. It establishes ownership, which is the foundation for all future user-specific features like "list my links" or "delete a link."
    *   **Cons:** This requires careful data modeling. Anonymous links (from Issue #1) will not have this attribute. This is easily handled by our NoSQL data model, which does not require all items to have the same attributes.

---

#### **Design the Architecture-as-Code (AaC)**

Here are the updated artifacts for this new feature.

**Artifact 1: Logical View (C4 Component Diagram)**

This diagram illustrates the new authenticated write path.

```mermaid
---
title: "Milestone #5 - Logical View (Custom Alias Creation)"
---
graph LR
    %% Define Styles
    style User fill:#EFEFEF,stroke:#333,stroke-width:2px
    style WS fill:#DCEBFF,stroke:#007BFF,stroke-width:2px
    style DB fill:#DCEBFF,stroke:#007BFF,stroke-width:2px,shape:cylinder

    %% Actor
    User("Authenticated User")

    %% System Boundary
    subgraph WarpLink System
        WS(WarpLink Service)
        DB[(URL Database)]
    end

    %% Interactions
    User -- "[1] POST /links/custom <br> (JWT, long_url, custom_alias)" --> WS
    WS -- "[2] Conditional Put <br> (fail if short_key exists)" --> DB
    DB -- "[3a] On Success: OK" --> WS
    WS -- "[4a] On Success: HTTP 201 Created" --> User
    DB -- "[3b] On Failure: Condition Failed" --> WS
    WS -- "[4b] On Failure: HTTP 409 Conflict" --> User
```

**Artifact 2: Physical View (Deployment Diagram)**

This diagram shows the new API endpoint protected by the Cognito Authorizer.

```mermaid
---
title: "Milestone #5 - Physical View (Custom Alias Creation)"
---
graph LR
    %% Styles
    style User fill:#EFEFEF,stroke:#333,stroke-width:2px
    style APIGW fill:#FF9900,stroke:#333,stroke-width:1px
    style ALB fill:#FF9900,stroke:#333,stroke-width:1px
    style WS fill:#232F3E,stroke:#FF9900,stroke-width:2px,color:#fff
    style DB fill:#4B2994,stroke:#fff,stroke-width:2px,color:#fff

    %% Definitions
    User(Client Browser)

    subgraph AWS Cloud
        subgraph API Stack
            APIGW(API Gateway <br> + JWT Authorizer)
            ALB(Application Load Balancer)
            WS(WarpLink Service)
            DB[(fa:fa-database DynamoDB Table 'links' <br> + user_id attribute)]
        end
    end

    %% Interactions
    User -- "[1] POST Request with JWT" --> APIGW
    APIGW -- "[2] Authenticates Request" --> ALB
    ALB -- "[3] Forwards to Task" --> WS
    WS -- "[4] PutItem with ConditionExpression" --> DB
    DB -- "[5] Returns Success or Failure" --> WS
    WS -- "[6] HTTP 201 or 409" --> User
```

**Artifact 3: Component-to-Resource Mapping Table**

The `WarpLink Service` and `URL Database` have expanded responsibilities.

| Logical Component | Physical Resource | Rationale (Why this technology?) |
| :--- | :--- | :--- |
| **WarpLink Service** | AWS Fargate | (Expanded Role) The service logic now includes handling authenticated requests, parsing JWTs for `user_id`, and executing conditional database writes to handle alias conflict resolution. |
| **URL Database** | Amazon DynamoDB | (Expanded Role) The `links` table schema is evolved to include an optional `user_id` attribute to denote ownership. Its `PutItem` API with `ConditionExpression` is the critical technology that enables the atomic check-and-set operation, preventing race conditions. |
| **API Entrypoint** | Amazon API Gateway | (Expanded Role) A new route (`/links/custom`) is added and, crucially, protected by the existing JWT Authorizer, ensuring only authenticated users can access this feature. |
