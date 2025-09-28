# WarpLink

## Logical View

### Milestone 01: Core Link Shortening (Write Path)

```mermaid
---
title: "Milestone #1 - Logical View (Write Path)"
---
graph LR
    %% Define Styles
    style User fill:#EFEFEF,stroke:#333,stroke-width:2px
    style WS fill:#DCEBFF,stroke:#007BFF,stroke-width:2px
    style KGS fill:#DCEBFF,stroke:#007BFF,stroke-width:2px
    style DB fill:#DCEBFF,stroke:#007BFF,stroke-width:2px,shape:cylinder

    %% Actor
    User(User)

    %% System Boundary
    subgraph WarpLink System
        direction LR
        WS(WarpLink Service)
        KGS(Key Generation Service)
        DB[(URL Database)]
    end

    %% Interactions
    User -- "[1] POST /links (long_url)" --> WS
    WS -- "[2] Request Unique Key" --> KGS
    KGS -- "[3] Provide Unique Key" --> WS
    WS -- "[4] Store (short_key, long_url)" --> DB
    WS -- "[5] Return Short URL" --> User
```

### Milestone 02: Core Link Redirection (Read Path)

```mermaid
---
title: "Milestone #2 - Logical View (Redirection Path)"
---
graph LR
    %% Define Styles
    style User fill:#EFEFEF,stroke:#333,stroke-width:2px
    style WS fill:#DCEBFF,stroke:#007BFF,stroke-width:2px
    style DB fill:#DCEBFF,stroke:#007BFF,stroke-width:2px,shape:cylinder

    %% Actor
    User(User)

    %% System Boundary
    subgraph WarpLink System
        direction LR
        WS(WarpLink Service)
        DB[(URL Database)]
    end

    %% Interactions
    User -- "[1] GET /short_key" --> WS
    WS -- "[2] Fetch long_url by short_key" --> DB
    DB -- "[3] Return long_url" --> WS
    WS -- "[4] HTTP 302 Redirect" --> User
```

### Milestone 03: High-Performance Read Cache

```mermaid
---
title: "Milestone #3 - Logical View (Cached Redirection Path)"
---
graph LR
    %% Define Styles
    style User fill:#EFEFEF,stroke:#333,stroke-width:2px
    style WS fill:#DCEBFF,stroke:#007BFF,stroke-width:2px
    style Cache fill:#D4EDDA,stroke:#155724,stroke-width:2px
    style DB fill:#DCEBFF,stroke:#007BFF,stroke-width:2px,shape:cylinder

    %% Actor
    User(User)

    %% System Boundary
    subgraph WarpLink System
        direction TB
        WS(WarpLink Service)
        
        subgraph Data Stores
            direction LR
            Cache(Read Cache)
            DB[(URL Database)]
        end
    end

    %% Interactions
    User -- "[1] GET /short_key" --> WS
    WS -- "[2] Check Cache" --> Cache
    Cache -- "[3a] On Hit: Return long_url" --> WS
    Cache -- "[3b] On Miss: Not Found" --> WS
    WS -- "[4a] On Miss: Fetch from DB" --> DB
    DB -- "[4b] Return long_url" --> WS
    WS -- "[4c] On Miss: Populate Cache" --> Cache
    WS -- "[5] HTTP 302 Redirect" --> User
```

### Milestone 04: User Authentication System

```mermaid
---
title: "Milestone #4 - Logical View (Authentication)"
---
graph LR
    %% Define Styles
    style User fill:#EFEFEF,stroke:#333,stroke-width:2px
    style Frontend fill:#C3E6CB,stroke:#155724,stroke-width:2px
    style AuthService fill:#F8D7DA,stroke:#721C24,stroke-width:2px
    style WS fill:#DCEBFF,stroke:#007BFF,stroke-width:2px
    style UsersDB fill:#DCEBFF,stroke:#007BFF,stroke-width:2px,shape:cylinder

    %% Actor
    User(User)

    %% System Boundary
    subgraph WarpLink System
        direction TB
        Frontend(Web Frontend)
        AuthService(Authentication Service)
        WS(WarpLink Service)
        UsersDB[(Users Database)]
    end

    %% Interactions
    User -- "[1] Clicks Login" --> Frontend
    Frontend -- "[2] Redirects to Auth" --> AuthService
    AuthService -- "[3] Issues JWT" --> User
    
    User -- "[4] API Request with JWT" --> WS
    WS -- "[5] (Optional) Read/Write" --> UsersDB
    WS -- "[6] Returns Data" --> User
```

### Milestone 05: Custom Alias Creation

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

### Milestone 06: Public API & Key Authentication

```mermaid
---
title: "Milestone #6 - Logical View (Public API & Key Auth)"
---
graph LR
    %% Define Styles
    style User fill:#EFEFEF,stroke:#333,stroke-width:2px
    style DevClient fill:#EFEFEF,stroke:#333,stroke-width:2px
    style WS fill:#DCEBFF,stroke:#007BFF,stroke-width:2px
    style UsersDB fill:#DCEBFF,stroke:#007BFF,stroke-width:2px,shape:cylinder

    %% Actors
    User("Registered User")
    DevClient("Developer Client")

    %% System Boundary
    subgraph WarpLink System
        WS(WarpLink Service)
        UsersDB[(Users Database)]
    end

    %% Interactions
    User -- "[1] Manage API Keys <br> (via JWT Auth)" --> WS
    WS -- "[2] Store/Retrieve API Key" --> UsersDB
    
    DevClient -- "[3] Shorten URL Request <br> (via API Key Auth)" --> WS
    WS -- "[4] Validate API Key" --> UsersDB
    WS -- "[5] Return Short URL" --> DevClient
```

### Milestone 07: Asynchronous Analytics Event Capture

```mermaid
---
title: "Milestone #7 - Logical View (Analytics Capture)"
---
graph TD
    %% Define Styles
    style User fill:#EFEFEF,stroke:#333,stroke-width:2px
    style WS fill:#DCEBFF,stroke:#007BFF,stroke-width:2px
    style MQ fill:#FFF3CD,stroke:#856404,stroke-width:2px
    style AS fill:#D1ECF1,stroke:#0C5460,stroke-width:2px
    style AnalyticsDB fill:#D1ECF1,stroke:#0C5460,stroke-width:2px,shape:cylinder

    User(User)

    subgraph "Redirection Path (Real-time)"
        User -- "[1] GET /short_key" --> WS(WarpLink Service)
        WS -- "[2] Publish Click Event" ---> MQ(Message Queue)
        WS -- "[3] HTTP 302 Redirect" --> User
    end

    subgraph "Analytics Path (Asynchronous)"
        MQ -- "[4] Events" --> AS(Analytics Service)
        AS -- "[5] Aggregate & Store Data" --> AnalyticsDB[(Analytics Database)]
    end
```

### Overall Logical View

```mermaid
---
title: "WarpLink - Overall Logical View"
---
graph TD
    %% Define Styles
    style User fill:#EFEFEF,stroke:#333,stroke-width:2px
    style DevClient fill:#EFEFEF,stroke:#333,stroke-width:2px
    style Frontend fill:#C3E6CB,stroke:#155724,stroke-width:2px
    style AuthService fill:#F8D7DA,stroke:#721C24,stroke-width:2px
    style WS fill:#DCEBFF,stroke:#007BFF,stroke-width:2px
    style KGS fill:#DCEBFF,stroke:#007BFF,stroke-width:2px
    style AS fill:#D1ECF1,stroke:#0C5460,stroke-width:2px
    style MQ fill:#FFF3CD,stroke:#856404,stroke-width:2px

    %% Actors
    User("Registered User")
    DevClient("Developer Client")

    subgraph WarpLink System
        direction LR
        
        subgraph User Interface
            direction TB
            Frontend(Web Frontend)
            AuthService(Authentication Service)
        end

        subgraph "Backend"
            direction TB
            WS(WarpLink Service)
            subgraph "Async Processing"
                MQ(Message Queue) --> AS(Analytics Service)
            end
            KGS(Key Generation Service)
        end

        subgraph Data Stores
            direction TB
            LinksDB[(URL Database)]
            Cache(Read Cache)
            UsersDB[(Users Database)]
            AnalyticsDB[(Analytics Database)]
        end
    end

    %% Interactions
    User --> Frontend
    Frontend --> AuthService
    Frontend -- "API Calls (JWT)" --> WS
    DevClient -- "API Calls (API Key)" --> WS
    
    WS --> KGS
    WS --> LinksDB
    WS --> Cache
    WS --> UsersDB
    WS --> MQ
    AS --> AnalyticsDB
```

## Physical View

### Milestone 01: Core Link Shortening (Write Path)

```mermaid
---
title: "Milestone #1 - Physical View (Write Path)"
---
graph LR
    %% Styles
    style User fill:#EFEFEF,stroke:#333,stroke-width:2px
    style APIGW fill:#FF9900,stroke:#333,stroke-width:1px
    style ALB fill:#FF9900,stroke:#333,stroke-width:1px
    style WS fill:#232F3E,stroke:#FF9900,stroke-width:2px,color:#fff
    style KGS fill:#232F3E,stroke:#FF9900,stroke-width:2px,color:#fff
    style DB fill:#4B2994,stroke:#fff,stroke-width:2px,color:#fff

    %% Definitions
    User(Client)

    subgraph AWS Cloud
        APIGW(API Gateway)

        subgraph VPC
            ALB(Application Load Balancer)

            subgraph ECS Fargate Tasks
                WS(WarpLink Service)
                KGS(Key Generation Service)
            end

            DB[(fa:fa-database DynamoDB Table 'links')]
        end
    end

    %% Interactions
    User -- "HTTPS Request" --> APIGW
    APIGW -- "Proxy" --> ALB
    ALB -- "Route" --> WS
    WS -- "SDK Call" --> DB
    WS -- "gRPC/HTTP" --> KGS
```

### Milestone 02: Core Link Redirection (Read Path)

```mermaid
---
title: "Milestone #2 - Physical View (Redirection Path)"
---
graph LR
    %% Styles
    style User fill:#EFEFEF,stroke:#333,stroke-width:2px
    style ALB fill:#FF9900,stroke:#333,stroke-width:1px
    style WS fill:#232F3E,stroke:#FF9900,stroke-width:2px,color:#fff
    style DB fill:#4B2994,stroke:#fff,stroke-width:2px,color:#fff

    %% Definitions
    User(Client Browser)

    subgraph AWS Cloud
        subgraph DNS
            R53(Route 53)
        end
        
        subgraph VPC
            ALB(Application Load Balancer)
            
            subgraph ECS Fargate Tasks
                WS(WarpLink Service)
            end

            DB[(fa:fa-database DynamoDB Table 'links')]
        end
    end

    %% Interactions
    User -- "[1] GET warplink.com/short_key" --> R53
    R53 -- "[2] Resolves to ALB IP" --> User
    User -- "[3] HTTPS Request" --> ALB
    ALB -- "[4] Forward to Task" --> WS
    WS -- "[5] GetItem(short_key)" --> DB
    DB -- "[6] Returns long_url" --> WS
    WS -- "[7] HTTP 302 Redirect" --> User
```

### Milestone 03: High-Performance Read Cache

```mermaid
---
title: "Milestone #3 - Physical View (Cached Redirection Path)"
---
graph LR
    %% Styles
    style User fill:#EFEFEF,stroke:#333,stroke-width:2px
    style ALB fill:#FF9900,stroke:#333,stroke-width:1px
    style WS fill:#232F3E,stroke:#FF9900,stroke-width:2px,color:#fff
    style Cache fill:#D11C37,stroke:#fff,stroke-width:2px,color:#fff
    style DB fill:#4B2994,stroke:#fff,stroke-width:2px,color:#fff

    %% Definitions
    User(Client Browser)

    subgraph AWS Cloud
        subgraph VPC
            ALB(Application Load Balancer)
            
            subgraph Data Stores
                Cache[(fa:fa-bolt ElastiCache for Redis)]
                DB[(fa:fa-database DynamoDB Table 'links')]
            end

            subgraph ECS Fargate Tasks
                WS(WarpLink Service)
            end

        end
    end

    %% Interactions
    User -- "HTTPS Request" --> ALB
    ALB -- "Forward" --> WS
    WS -- "GET from Cache" --> Cache
    WS -- "On Miss: GET from DB" --> DB
    WS -- "On Miss: SET to Cache" --> Cache
    WS -- "HTTP 302 Redirect" --> User
```

### Milestone 04: User Authentication System

```mermaid
---
title: "Milestone #4 - Physical View (Authentication)"
---
graph LR
    %% Styles
    style User fill:#EFEFEF,stroke:#333,stroke-width:2px
    style CDN fill:#FF9900,stroke:#333,stroke-width:1px
    style S3 fill:#FF9900,stroke:#333,stroke-width:1px
    style Cognito fill:#F8D7DA,stroke:#721C24,stroke-width:2px
    style APIGW fill:#FF9900,stroke:#333,stroke-width:1px
    style ALB fill:#FF9900,stroke:#333,stroke-width:1px
    style WS fill:#232F3E,stroke:#FF9900,stroke-width:2px,color:#fff
    style DB fill:#4B2994,stroke:#fff,stroke-width:2px,color:#fff

    %% Definitions
    User(Client Browser)

    subgraph AWS Cloud
        Cognito(fa:fa-users AWS Cognito <br> User Pool)
        
        subgraph Frontend Hosting
            CDN(fa:fa-globe Amazon CloudFront)
            S3(fa:fa-archive Amazon S3 Bucket)
        end
        
        subgraph API Stack
            subgraph " "
                direction LR
                APIGW(API Gateway <br> + JWT Authorizer)
                ALB(Application Load Balancer)
            end
            
            subgraph " "
                direction LR
                 WS(WarpLink Service <br> Fargate Task)
                 DB[(fa:fa-database DynamoDB Table 'users')]
            end
        end
    end

    %% Interactions
    User -- "[1] Loads site" --> CDN -- "Origin" --> S3
    User -- "[2] Login Redirect" --> Cognito
    Cognito -- "[3] Returns JWT" --> User
    User -- "[4] API Call with JWT" --> APIGW
    APIGW -- "[5] Validates JWT w/ Cognito" --> Cognito
    APIGW -- "[6] Forward on Success" --> ALB --> WS
    WS -- "[7] CRUD Operations" --> DB
```

### Milestone 05: Custom Alias Creation

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

### Milestone 06: Public API & Key Authentication

```mermaid
---
title: "Milestone #6 - Physical View (Public API & Key Auth)"
---
graph TD
    %% Define Styles
    style User fill:#EFEFEF,stroke:#333,stroke-width:2px
    style DevClient fill:#EFEFEF,stroke:#333,stroke-width:2px
    style APIGW fill:#FF9900,stroke:#333,stroke-width:1px
    style ALB fill:#FF9900,stroke:#333,stroke-width:1px
    style WS fill:#232F3E,stroke:#FF9900,stroke-width:2px,color:#fff
    style UsersDB fill:#4B2994,stroke:#fff,stroke-width:2px,color:#fff
    style LinksDB fill:#4B2994,stroke:#fff,stroke-width:2px,color:#fff
    style Cognito fill:#F8D7DA,stroke:#721C24,stroke-width:2px

    %% Actors
    User("Registered User <br> (via Browser)")
    DevClient("Developer Client")

    subgraph AWS Cloud
        Cognito(fa:fa-users AWS Cognito)
        subgraph VPC
            APIGW(fa:fa-sitemap API Gateway <br> JWT & API Key Auth)
            ALB(Application Load Balancer)
            WS(WarpLink Service)
            UsersDB[(fa:fa-database 'users' Table)]
            LinksDB[(fa:fa-database 'links' Table)]
        end
    end

    %% Interactions
    User -- "1a. Manage API Keys (JWT)" --> APIGW
    DevClient -- "1b. POST /v1/shorten (API Key)" --> APIGW
    
    APIGW -- "Validates JWT with Cognito" --> Cognito
    APIGW -- "Forwards authenticated requests" --> ALB
    ALB --> WS
    
    WS -- "CRUD API Keys" --> UsersDB
    WS -- "Create Link" --> LinksDB
```

### Milestone 07: Asynchronous Analytics Event Capture

```mermaid
---
title: "Milestone #7 - Physical View (Analytics Capture)"
---
graph LR
    %% Styles
    style User fill:#EFEFEF,stroke:#333,stroke-width:2px
    style ALB fill:#FF9900,stroke:#333,stroke-width:1px
    style WS fill:#232F3E,stroke:#FF9900,stroke-width:2px,color:#fff
    style SQS fill:#FF4F8B,stroke:#fff,stroke-width:2px,color:#fff
    style Lambda fill:#FF9900,stroke:#333,stroke-width:2px
    style AnalyticsDB fill:#4B2994,stroke:#fff,stroke-width:2px,color:#fff

    %% Definitions
    User(Client Browser)

    subgraph AWS Cloud
        subgraph VPC
            ALB(Application Load Balancer)
            WS(WarpLink Service <br> Fargate Task)
        end
        
        subgraph "Async Processing"
            SQS(fa:fa-comments Amazon SQS Queue)
            Lambda(fa:fa-microchip AWS Lambda Function)
            AnalyticsDB[(fa:fa-database DynamoDB Table 'analytics')]
        end
    end

    %% Interactions
    User -- "[1] HTTPS Request" --> ALB
    ALB -- "[2] Forward" --> WS
    WS -- "[3] SendMessage (Fire-and-Forget)" --> SQS
    WS -- "[4] HTTP 302 Redirect" --> User
    
    SQS -- "[5] Triggers Lambda (Event)" --> Lambda
    Lambda -- "[6] Batch Write/Update" --> AnalyticsDB
```

### Overall Physical View

```mermaid
---
title: "WarpLink - Overall Physical View"
---
graph TD
    %% Class Definitions for consistent styling
    classDef actors fill:#EFEFEF,stroke:#333,stroke-width:2px;
    classDef web_edge fill:#FF9900,stroke:#333,stroke-width:1px;
    classDef identity fill:#F8D7DA,stroke:#721C24,stroke-width:2px;
    classDef compute fill:#232F3E,stroke:#FF9900,stroke-width:2px,color:#fff;
    classDef messaging fill:#FF4F8B,stroke:#fff,stroke-width:2px,color:#fff;
    classDef db fill:#4B2994,stroke:#fff,stroke-width:2px,color:#fff;

    %% Actors
    Browser("Client Browser"):::actors
    DevClient("Developer Client"):::actors

    subgraph AWS Cloud
        %% Tier 1: Identity and Frontend Hosting
        subgraph " "
            direction LR
            Cognito(fa:fa-users AWS Cognito):::identity
            subgraph Frontend Hosting
                CDN(fa:fa-globe Amazon CloudFront):::web_edge --> S3(fa:fa-archive S3 Bucket):::web_edge
            end
        end

        %% Tier 2: Synchronous API and Data
        subgraph VPC
            direction TB
            APIGW(fa:fa-sitemap API Gateway):::web_edge
            ALB(Application Load Balancer):::web_edge
            Fargate(WarpLink Services <br> on Fargate):::compute
            
            subgraph Primary Data Stores
                direction LR
                Cache[(fa:fa-bolt ElastiCache)]:::db
                LinksDB[(fa:fa-database 'links' Table)]:::db
                UsersDB[(fa:fa-database 'users' Table)]:::db
            end
        end

        %% Tier 3: Asynchronous Pipeline
        subgraph Async Analytics Pipeline
            direction LR
            SQS(fa:fa-comments SQS Queue):::messaging --> Lambda(fa:fa-microchip Lambda):::web_edge --> AnalyticsDB[(fa:fa-database 'analytics' Table)]:::db
        end
    end

    %% Interactions
    Browser --> CDN
    Browser -- "Login" --> Cognito
    Browser -- "API Call (JWT)" --> APIGW
    DevClient -- "API Call (API Key)" --> APIGW
    
    APIGW -- "Validates with" --> Cognito
    APIGW --> ALB --> Fargate
    
    Fargate --> Cache
    Fargate --> LinksDB
    Fargate --> UsersDB
    Fargate -- "Fire-and-forget Event" --> SQS
```
