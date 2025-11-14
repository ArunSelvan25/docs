
# Technical Documentation: A Workshop on System Architecture

This document provides a comprehensive, engineering-level guide to the principles, patterns, and components of modern system architecture. It is designed for use as a workshop handout, training manual, or technical reference.

---

## 1. Introduction to System Architecture

### Definition
System Architecture is the high-level structure of a software system. It defines the components, their relationships, and the principles and guidelines governing their design and evolution over time. Think of it as the **blueprint for a city**, not the detailed design of a single house. It dictates how the system will behave under load, how it can be modified, and how resilient it will be to failure.

### Why It Matters
A system's architecture is the most critical technical decision. It's the hardest to change and has the largest impact on:
* **Performance:** Determines latency and throughput.
* **Scalability:** Defines how the system grows to handle more load.
* **Reliability:** Governs fault tolerance and availability.
* **Maintainability:** Impacts how easy it is to add features, fix bugs, and onboard new developers.
* **Cost:** Directly influences infrastructure and development expenses.

### Use Cases
* Web Applications (e.g., E-commerce, Social Media)
* Mobile Application Backends
* Real-time Data Processing Pipelines (e.g., Financial Tickers)
* Internet of Things (IoT) Platforms
* Distributed Computing Grids

### Key Characteristics

| Characteristic | Description | Real-World Analogy |
| :--- | :--- | :--- |
| **Scalability** | The ability of the system to handle increasing load. | A restaurant adding more tables (horizontal) or a chef cooking faster (vertical). |
| **Reliability** | The system remains operational and correct despite component failures. | A car with a spare tire and a backup engine. |
| **Availability** | The percentage of time the system is operational (e.g., "five nines" = 99.999%). | A utility (like electricity) that is almost never down. |
| **Maintainability** | The ease with which the system can be modified, debugged, or updated. | A house with clearly labeled pipes and electrical panels. |
| **Cost Efficiency** | Delivering the required performance and features at the lowest possible cost. | Using a fuel-efficient car for a daily commute instead of a race car. |

### Examples from Real Systems
* **Amazon:** A classic example of **microservices**. The website is composed of hundreds of independent services (e.in g., cart, recommendations, payment, user-profile) that communicate via APIs. This allows independent scaling (e.g., scale the "deals" service on Prime Day) and independent deployment.
* **Netflix:** A cloud-native, microservice-based architecture. It heavily uses **event-driven patterns** and an extensive **CDN (Open Connect)** to stream video globally with low latency. Its "Chaos Monkey" tool is famous for testing reliability by randomly terminating production instances.
* **UPI (India):** A massive, real-time, **event-driven** system. It's not a single application but an interoperable protocol connecting dozens of banks, payment service providers (PSPs), and the central NPCI switch. It must be extremely high-availability and strongly consistent (for financial transactions).

---

## 2. Architectural Styles

### Monolithic Architecture

**Definition:** The entire application is built and deployed as a single, indivisible unit. All components (UI, business logic, data access) are tightly coupled and run in the same process.

**Diagram:**
```ascii
+---------------------------------------------+
|          Monolithic Application             |
|                                             |
| +-----------+  +-----------+  +-----------+ |
| |   Web UI  |  | Business  |  |   Data    | | 
| | (View)    |==|   Logic   |==|  Access   | |
| +-----------+  +-----------+  +-----------+ |
|                                             |
|                  +----------+               |
|                  | Database |               |
|                  +----------+               |
+---------------------------------------------+
````

**Use Cases:**

  * Startups and MVPs (Minimum Viable Products)
  * Simple applications with limited scope
  * Projects where speed of initial development is key

| Advantages | Disadvantages |
| :--- | :--- |
| **Simplicity:** Easy to develop, test, and deploy (at first). | **Hard to Scale:** You must scale the *entire* application, not just the parts that need it. |
| **Performance:** Components communicate in-process, avoiding network latency. | **Low Reliability:** A bug in one module (e.g., memory leak) can crash the entire system. |
| **Easy Debugging:** All code is in one place; tracing a request is straightforward. | **Brittle:** High coupling means a change in one place can break another. |
| **Single Codebase:** Easier to manage (initially). | **Technology Lock-in:** Difficult to adopt new technologies or frameworks. |

**When to Choose:** Your team is small, the problem is simple, and time-to-market is the \#1 priority.
**When to Avoid:** You are building a large, complex system that needs to scale to millions of users and be maintained by multiple teams.

-----

### Microservices Architecture

**Definition:** The application is structured as a collection of small, autonomous, and independently deployable services. Each service is self-contained, owns its data, and communicates with others over a network (typically HTTP APIs or message queues).

**Diagram:**

```ascii
+-------------+      +-------------+      +-------------+
| Service A   |      | Service B   |      | Service C   |
|(e.g., User) |      |(e.g., Order)|      |(e.g, Notify)|
| +---------+ |      | +---------+ |      | +---------+ |
| | DB (A)  | |<---->| | DB (B)  | |<---->| | (Queue) | |
| +---------+ |      | +---------+ |      | +---------+ |
+-------------+      +-------------+      +-------------+
       ^                    ^                    ^
       |                    |                    |
+-----------------------------------------------------+
|                      API Gateway                    |
+-----------------------------------------------------+
       ^
       |
+-------------+
|   Client    |
+-------------+
```

**Use Cases:**

  * Large, complex applications (e.g., Netflix, Amazon, Uber)
  * Systems with multiple teams working independently
  * Applications that require high scalability and reliability for specific features

| Advantages | Disadvantages |
| :--- | :--- |
| **Independent Scaling:** Scale only the services that need it. | **High Complexity:** You are now managing a distributed system. |
| **Independent Deployment:** Push updates to one service without redeploying the whole app. | **Network Latency:** In-process calls become network calls. |
| **Resilience:** Failure in one service doesn't (or shouldn't) bring down the entire system. | **Data Consistency:** Managing transactions across services is extremely difficult. |
| **Technology Freedom:** Use the best language/database for each service. | **Operational Overhead:** Requires complex DevOps (service discovery, orchestration, monitoring). |

**When to Choose:** You have a complex domain, a large team (or teams), and high-scalability requirements.
**When to Avoid:** You are a small team or startup. Don't start here; you can refactor a monolith later (e.g., into a Modular Monolith first).

-----

### Event-Driven Architecture (EDA)

**Definition:** An architecture centered around the production, detection, consumption of, and reaction to *events*. Components are loosely coupled; they don't call each other directly. Instead, a *Producer* emits an event, and one or more *Consumers* subscribe and react to it.

**Diagram:**

```ascii
+-----------+                    +-------------+
|  Service A| -- (Event 1) -->   |             |
| (Producer)|                    | Event Broker| -- (Event 1) --> +-----------+
+-----------+                    | (e.g, Kafka)|                  | Service B |
                                 |             |                  | (Consumer)|
+-----------+                    |             |                  +-----------+
|  Service C| -- (Event 2) -->   |             |
| (Producer)|                    |             | -- (Event 2) --> +-----------+
+-----------+                    +-------------+                  | Service D |
                                                                  | (Consumer)|
                                                                  +-----------+
```

**Use Cases:**

  * Asynchronous workflows (e.g., e-commerce order processing)
  * Real-time data processing and analytics
  * Systems that need to be highly decoupled and resilient
  * Integrating disparate systems

| Advantages | Disadvantages |
| :--- | :--- |
| **Loose Coupling:** Producers don't know or care about consumers. | **Complex Debugging:** Tracing a "flow" is hard; it's no longer a linear request. |
| **High Scalability:** The event broker acts as a massive buffer, allowing consumers to process at their own pace. | **Eventual Consistency:** Data is not updated instantly across the system. |
| **Resilience:** If a consumer service is down, events queue up in the broker and are processed when it comes back online. | **Broker is a SPOF:** The event broker (e.g., Kafka) is a critical component that must be made highly available. |

**When to Choose:** Your system has complex, asynchronous workflows, or you need to decouple components for resilience.
**When to Avoid:** You need simple, synchronous, request/response interactions or strong, immediate consistency.

-----

### Serverless Architecture

**Definition:** An architecture where the cloud provider (e.g., AWS, Google Cloud) manages the server infrastructure. Developers write and deploy code in the form of functions (e.g., AWS Lambda) that are triggered by events (like an HTTP request, file upload, or message queue).

**Diagram:**

```ascii
                   +----------------+
(HTTP Request) --> | API Gateway    | -- (Triggers) --> +-------------+
                   +----------------+                   | Function 1  |
                                                        | (e.g., Auth)|
                                                        +-------------+
                                                            |
                   +----------------+                       v
(File Upload)  --> |  Cloud Storage | -- (Triggers) --> +-------------+
                   +----------------+                   | Function 2  |
                                                        |(e.g, Resize)|
                                                        +-------------+
                                                            |
                                                            v
                                                      +-------------+
                                                      |  Database   |
                                                      |e.g: DynamoDB|
                                                      +-------------+
```

**Use Cases:**

  * APIs and backends with variable or spiky traffic
  * Asynchronous background tasks (e.g., image processing)
  * Scheduled jobs (Cron)
  * "Glue" code for integrating different services

| Advantages | Disadvantages |
| :--- | :--- |
| **No Server Management:** Zero infrastructure overhead. | **Vendor Lock-in:** Code is often tied to a specific cloud provider's services. |
| **Pay-per-Use:** You only pay for the compute time you *actually* use, down to the millisecond. | **Cold Starts:** Functions that haven't been used recently can have initial latency (a "cold start"). |
| **Automatic Scaling:** Scales from zero to thousands of requests instantly. | **Complexity:** Can become a complex web of "Functions-as-a-Service" (FaaS). |
| **Forced Modularity:** Encourages small, single-purpose functions. | **Limited Execution Time:** Functions often have a maximum execution limit (e.g., 15 minutes). |

**When to Choose:** You have event-based, spiky workloads and want to minimize infrastructure cost and management.
**When to Avoid:** You have long-running, compute-heavy tasks or require deep control over the underlying environment.

-----

### Modular Architecture (Modular Monolith)

**Definition:** A compromise between a monolith and microservices. The application is built as a single deployable unit (a monolith), but its internal code is organized into
highly-decoupled, independent modules with explicit, well-defined boundaries (APIs). Modules *cannot* directly access each other's internal code or databases.

**Diagram:**

```ascii
+---------------------------------------------------+
|               Monolithic Application              |
|                                                   |
| +-------------+   (Public API)   +--------------+ |
| |   Module A  | <------------>   |   Module B   | |
| | (e.g., User)|                  | (e.g., Order)| |
| | [Internal]  |                  | [Internal]   | |
| |[DB Tables A]|                  | [DB Tables B]| |
| +-------------+                  +--------------+ |
|        |                                 |        |
|        +--------------|------------------+        |
|                       v                           |
|                  +----------+                     |
|                  | Database |  Shared, but        |
|                  |          |logically partitioned|
|                  +----------+                     |
+---------------------------------------------------+
```

**Use Cases:**

  * Most "standard" business applications
  * Startups that want to build quickly but avoid monolithic spaghetti code
  * An intermediary step when migrating from a monolith to microservices

| Advantages | Disadvantages |
| :--- | :--- |
| **Manages Complexity:** Enforces strong boundaries *in the code*, preventing spaghetti. | **Requires Discipline:** Easy for developers to "cheat" and bypass the public APIs. |
| **Easier Refactoring:** Well-defined modules are much easier to extract into microservices later. | **Single Deployment:** Still a single unit. A bad deploy takes down the whole app. |
| **Good Performance:** All calls are in-process. | **Single Tech Stack:** All modules must share the same language/framework. |
| **Simple Operations:** It's still just one thing to deploy and monitor. | **Shared Database:** Risk of one module's queries impacting another. |

**When to Choose:** This is arguably the **best default starting point** for many new applications.
**When to Avoid:** You have extreme, independent scaling needs from day one (a rare case).

-----

## 3\. Core Components of Modern Architecture

### Load Balancer

  * **Purpose:** To distribute incoming network traffic across multiple backend servers.
  * **How it works:** It sits in front of your servers and routes client requests using an algorithm (e.g., Round Robin, Least Connections, IP Hash). It also performs **health checks** to stop sending traffic to unhealthy servers.
  * **Where it fits:** At the very front of your system, between the public internet (or client) and your web servers/API gateway.
  * **Advantages:** High Availability (HA), fault tolerance, horizontal scalability.
  * **Examples:** Nginx, HAProxy, AWS ELB, Google Cloud Load Balancer.

### API Gateway

  * **Purpose:** A single entry point ("front door") for all clients to access your backend services (especially in a microservices architecture).
  * **How it works:** It receives a client request, routes it to the correct downstream service, and may perform other "cross-cutting" concerns like:
      * **Authentication & Authorization:** Validating tokens.
      * **Rate Limiting:** Protecting services from abuse.
      * **Request/Response Transformation:** Converting data formats.
      * **Caching:** Caching common responses.
  * **Where it fits:** Behind the Load Balancer, but in front of all your application services.
  * **Advantages:** Simplifies the client (it only knows one endpoint), secures the backend (services aren't public), reduces code duplication.
  * **Examples:** AWS API Gateway, Kong, Apigee, Tyk.

### Application Services

  * **Purpose:** The "brains" of the operation. This is your custom business logic.
  * **How it works:** A service (e.g., `OrderService`, `UserService`) written in a language (e.g., Java, Python, Node.js) that exposes an API (e.g., REST, gRPC) to perform a specific business function.
  * **Where it fits:** The core of your backend, sitting behind the API Gateway.
  * **Advantages:** This is the value you are building. It's the implementation of your features.
  * **Examples:** A Spring Boot app, a Django project, a Node.js Express server.

### Databases (SQL and NoSQL)

  * **Purpose:** To provide persistent, long-term storage for your application data.
  * **How it works:**
      * **SQL (Relational):** Stores data in predefined, structured tables with rows and columns. Enforces a rigid schema and uses SQL (Structured Query Language). Ideal for data that is highly related and requires strong consistency (ACID compliance).
      * **NoSQL (Non-relational):** A broad category. Stores data in flexible formats like documents (MongoDB), key-value pairs (Redis), wide-column (Cassandra), or graphs (Neo4j). Ideal for unstructured data, high write throughput, and horizontal scalability.
  * **Where it fits:** The "stateful" layer at the back of your system, accessed by your Application Services.
  * **Advantages:** SQL (Consistency, data integrity, powerful queries). NoSQL (Scalability, flexibility, high performance for specific use cases).
  * **Examples:** **SQL** (PostgreSQL, MySQL, MS SQL Server). **NoSQL** (MongoDB, Redis, Cassandra, DynamoDB).

### Cache (Redis/Memcached)

  * **Purpose:** To store frequently accessed data in a very fast, in-memory database. This reduces load on your main database and speeds up response times.
  * **How it works:** An application first checks the cache for data. If found (**cache hit**), it returns it immediately. If not found (**cache miss**), it queries the main database, stores the result in the cache, and then returns it.
  * **Where it fits:** Between your Application Services and your Database.
  * **Advantages:** Drastically reduces latency, decreases load on databases.
  * **Examples:** Redis, Memcached.

### Message Queue (Kafka/RabbitMQ/SQS)

  * **Purpose:** A component that allows asynchronous communication between services. It acts as a buffer or "mailbox."
  * **How it works:** A *Producer* service writes a message (e.S.g., `order_created` event) to a *Queue* or *Topic*. A *Consumer* service subscribes to that queue and processes the message at its own pace. The producer and consumer are decoupled.
  * **Where it fits:** As the "connective tissue" between services in an event-driven architecture.
  * **Advantages:** Decoupling, resilience (if consumer fails, messages wait), load leveling (absorbs spikes in traffic), asynchronous processing.
  * **Examples:** Kafka (for high-throughput streaming), RabbitMQ (for complex routing), AWS SQS (simple, managed queue).

### CDN (Content Delivery Network)

  * **Purpose:** To deliver static assets (images, videos, CSS, JS files) to users from a server that is *geographically close* to them.
  * **How it works:** It's a globally distributed network of "edge" servers. When a user in Japan requests an image, they get it from a server in Tokyo, not from your main server in Virginia.
  * **Where it fits:** At the very edge, "in front" of your entire application for static content.
  * **Advantages:** Massively reduces latency for users, reduces load on your application servers.
  * **Examples:** Cloudflare, Akamai, AWS CloudFront, Fastly.

### Monitoring & Logging Stack

  * **Purpose:** To understand the health and behavior of your system.
  * **How it works:**
      * **Monitoring (Metrics):** Collects time-series data (numbers) about your system (e.g., CPU %, request count, error rate). (e.g., **Prometheus**, Datadog).
      * **Logging:** Collects text-based event logs from all services (e.g., "User 123 logged in"). (e.g., **ELK Stack** - Elasticsearch, Logstash, Kibana; or Splunk).
      * **Tracing:** Tracks a single request as it flows through multiple microservices. (e.g., Jaeger, Zipkin).
  * **Where it fits:** It's an external system that *observes* every other component.
  * **Advantages:** Essential for debugging, alerting on failures, and understanding performance bottlenecks.
  * **Examples:** Prometheus + Grafana (Monitoring), ELK Stack (Logging), Datadog (All-in-one).

### Service Discovery

  * **Purpose:** In a dynamic microservices world, how does Service A find the IP address of Service B (when new instances of B are constantly being created and destroyed)?
  * **How it works:** Services register themselves with a central "registry" (like a phone book) when they start up. When another service wants to call them, it queries the registry to get a current list of healthy instances.
  * **Where it fits:** A central, critical component in a microservices architecture.
  * **Advantages:** Enables dynamic scaling and resilience.
  * **Examples:** Consul, Eureka. (Often handled automatically by orchestrators like Kubernetes).

### Reverse Proxy

  * **Purpose:** A server that sits in front of one or more web servers, forwarding client requests to them.
  * **How it works:** From the client's perspective, it looks like the main server. Internally, it can handle tasks like SSL termination (HTTPS), caching static content, compression, and load balancing.
  * **Where it fits:** Between the Load Balancer and your Application Servers. *Note: A Load Balancer is a type of Reverse Proxy, but a Reverse Proxy can do more.*
  * **Advantages:** Security, performance, simplifying server configuration.
  * **Examples:** Nginx, Apache httpd.

### Authentication/Authorization Layer

  * **Purpose:**
      * **Authentication (AuthN):** Who are you? (Verifying identity, e.g., login/password, JWT).
      * **Authorization (AuthZ):** What are you allowed to do? (Checking permissions, e.g., admin vs. user).
  * **How it works:** Often implemented as a dedicated service (e.g., `AuthService`) or as a layer in the API Gateway. It issues and/or validates a token (like a JWT) on every "protected" request.
  * **Where it fits:** At the entry point (API Gateway) or as a central service.
  * **Advantages:** Centralizes security logic, securing your application.
  * **Examples:** OAuth 2.0 / OIDC (protocols), Okta, Auth0 (services), Keycloak (self-hosted).

-----

## 4\. Detailed Example: E-commerce Architecture

We will design a microservices-based, event-driven e-commerce platform.

**Services:**

  * **User Service:** Manages user accounts, profiles, auth.
  * **Catalog Service:** Manages products, categories, search.
  * **Cart Service:** Manages temporary user carts.
  * **Order Service:** Manages order creation, history, status.
  * **Payment Service:** Integrates with external gateways (e.g., Stripe).
  * **Inventory Service:** Manages stock levels for products.
  * **Notification Service:** Sends emails, SMS, push notifications.

-----

### Flow 1: Product Listing & Cart

1.  **Client** (Browser) requests `/products?category=electronics`.
2.  **CDN** serves static assets (CSS, JS).
3.  Request hits **Load Balancer (LB)**.
4.  LB forwards to **API Gateway**.
5.  API Gateway (optional: auth check) routes to **Catalog Service**.
6.  **Catalog Service** checks **Cache (Redis)** for this query.
      * **Cache Hit:** Returns cached JSON to client. (Fast)
      * **Cache Miss:** Queries **Product DB (MongoDB)**, gets results, stores them in Cache, and returns to client. (Slower)
7.  User clicks "Add to Cart". Request hits API GW $\rightarrow$ **Cart Service**.
8.  **Cart Service** writes/updates the user's cart in its **Cart DB (Redis)**, which is perfect for fast-changing, non-permanent data.

-----

### Flow 2: Order Placement (The Core Flow)

This is a complex, asynchronous flow.

**Sequence Diagram (Text-based):**

```
Client        API-GW        OrderSvc      PaymentSvc    MsgQueue      InventorySvc  NotifySvc
  |             |             |             |             |             |             |
  | POST /order |             |             |             |             |             |
  |------------>|             |             |             |             |             |
  |             | POST /order |             |             |             |             |
  |             |------------>|             |             |             |             |
  |             |             | 1. Create Order (status=PENDING)      |             |             |
  |             |             | 2. Call PaymentSvc |             |             |             |
  |             |             |------------------->|             |             |             |
  |             |             |             | 3. Call Stripe (External) |             |             |
  |             |             |             |------------>|             |             |
  |             |             |             |             |             |             |
  |             |             |             | 4. [Later] Stripe Webhook |             |             |
  |             |             |             |<--------------------------|             |             |
  |             |             |             | 5. Mark Order PAID        |             |             |
  |             |             |             | 6. Publish 'order_paid'   |             |             |
  |             |             |             |-------------------------->|             |             |
  |             |             |             |             |             |             |
  |             |             |             |             | 'order_paid'|             |             |
  |             |             |             |             |------------>|             |
  |             |             |             |             |             | 7. Decrement Stock |
  |             |             |             |             |             |             |
  |             |             |             |             | 'order_paid'|             |
  |             |             |             |             |-------------------------->|
  |             |             |             |             |             |             | 8. Send Email
  |             |             | 2a. Return 'PENDING' |             |             |             |
  |<--------------------------|             |             |             |             |
  | (Show user "Processing")  |             |             |             |             |
  |             |             |             |             |             |             |
```

**Breakdown:**

1.  **Order Creation:** Client submits cart. API Gateway forwards to **Order Service**. Order Service creates an order in its **Order DB (PostgreSQL)** with `status: PENDING` and returns the `orderId` to the client *immediately*.
2.  **Payment:** Order Service *synchronously* calls **Payment Service**. Payment Service calls an external gateway (e.g., Stripe) and returns a payment intent to the Order Service.
3.  **Client Confirmation:** The client is told the order is `PENDING`. The client UI uses the Stripe intent to prompt the user for payment (e.g., credit card form).
4.  **Payment Webhook:** The user pays. Stripe *asynchronously* calls a webhook endpoint on our **Payment Service** (e.g., `/webhook/stripe`).
5.  **Event Publishing:** The Payment Service validates the webhook, marks the payment as `PAID`, and then publishes an `order_paid` event to a **Message Queue (Kafka)**. This event contains the `orderId` and items.
6.  **Event Consumption (Decoupled):**
      * **Inventory Service:** Subscribes to `order_paid` events. When it receives one, it decrements the stock for the items in its **Inventory DB**.
      * **Notification Service:** Also subscribes to `order_paid`. When it receives one, it sends a confirmation email/SMS to the user.

-----

### Data Models (Simplified JSON)

```json
// Catalog Service (MongoDB)
{
  "_id": "product_123",
  "name": "Laptop",
  "description": "...",
  "price": 1200,
  "category": "electronics",
  "specs": { "cpu": "M3", "ram": "16GB" }
}

// Order Service (PostgreSQL)
{
  "order_id": "uuid-abc-123",
  "user_id": "user_456",
  "status": "PAID", // PENDING, PAID, SHIPPED
  "total_amount": 1250.50,
  "created_at": "timestamp",
  "items": [
    { "product_id": "product_123", "quantity": 1, "price": 1200 }
  ]
}

// Inventory Service (SQL or NoSQL)
{
  "product_id": "product_123",
  "stock_level": 150
}
```

### Scaling & Caching Strategy

  * **Scaling:** All services (Catalog, Order, etc.) are stateless and can be **horizontally scaled** (e.g., by adding more Docker containers in Kubernetes). The Databases (PostgreSQL, MongoDB) will use **Read Replicas** to scale read operations. Kafka and Redis are inherently scalable.
  * **Caching:**
      * **CDN:** For all product images, CSS, JS.
      * **Redis (Catalog):** Cache results of common queries (e.g., homepage products, popular categories).
      * **Redis (Cart):** The entire cart database *is* a cache.

### Failure Handling & Idempotency

  * **Failure:**
      * **Circuit Breaker:** If the Inventory Service is down, the API Gateway or calling service (e.g., Order Service) should "trip a circuit" and stop calling it for a short time to let it recover.
      * **Retries:** Asynchronous event consumers (Inventory, Notification) should retry processing a message with *exponential backoff* if they fail.
      * **Dead Letter Queue (DLQ):** If a message fails processing (e.g., 5 times), it is moved to a DLQ for manual inspection.
  * **Idempotency:**
      * **Problem:** What if the Payment Service's `order_paid` event is sent *twice*? The Inventory Service might decrement stock twice.
      * **Solution:** The `order_paid` event must contain a unique ID (e.g., `event_id` or just use the `order_id`). The Inventory Service must keep track of `order_id`s it has already processed. If it sees the same `order_id` again, it skips it.
      * This is **CRITICAL** for payment and order flows.

-----

## 5\. Detailed Flow Diagram Explanation

This is the canonical flow for a modern, scalable web application.

`Client → Load Balancer → API Gateway → Services → DB/Cache → Response`

### 1\. Client

  * **What:** The user's browser, mobile app, or another system.
  * **Action:** Initiates an HTTP request (e.g., `GET /api/users/123`).
  * **Latency:** The user's network connection (e.g., 20ms - 500ms).

### 2\. $\rightarrow$ Load Balancer (LB)

  * **What:** A high-availability network device (e.g., Nginx, AWS ELB).
  * **Why:**
    1.  **Distributes Load:** Spreads requests across multiple (e.g., 3) API Gateway instances.
    2.  **Fault Tolerance:** Performs health checks. If Gateway \#2 fails, the LB stops sending it traffic, and the user never notices.
  * **Fault-Tolerance:** Health checks, automatic removal of unhealthy nodes.
  * **Latency:** Adds minimal latency (e.g., \< 1ms).

### 3\. $\rightarrow$ API Gateway (GW)

  * **What:** A managed service (e.g., Kong, AWS API Gateway) or a self-hosted proxy.
  * **Why:**
    1.  **Authentication:** Checks the `Authorization` header (e.g., validates a JWT). Rejects unauthenticated requests (401).
    2.  **Rate Limiting:** Checks the client's IP or API key. Rejects if they exceed their quota (429 Too Many Requests).
    3.  **Routing:** Reads the path (`/api/users/123`) and forwards the request to the correct internal service (e.g., `http://user-service:8080/users/123`).
  * **Fault-Tolerance:** Can cache responses for common `GET` requests. Can implement circuit breakers if a downstream service is failing.
  * **Latency:** Adds some latency (e.g., 5ms - 50ms) due to the logic it performs.

### 4\. $\rightarrow$ Application Service(s)

  * **What:** Your backend code (e.g., `UserService`).
  * **Why:** This is the business logic.
    1.  Parses the request.
    2.  Performs the business operation (e.g., "get user data").
    3.  Fetches/writes data from the data layer.
  * **Fault-Tolerance:** The code itself should handle errors (e.g., `try-catch` blocks). The LB/Orchestrator handles *instance* failures.
  * **Latency:** This is your application's processing time (e.g., 10ms - 200ms).

### 5\. $\rightarrow$ DB / Cache

  * **What:** Your stateful persistence layers.
  * **Why:**
    1.  **Cache (Redis):** The Service *first* asks the cache: "Do you have `user:123`?"
          * **Cache Hit:** Redis returns the data in \< 5ms. The Service skips the DB.
    2.  **Database (PostgreSQL):**
          * **Cache Miss:** The Service queries the DB: `SELECT * FROM users WHERE id=123`.
          * The DB executes the query, finds the data on disk, and returns it.
          * The Service then *writes* this data to the cache for next time.
  * **Fault-Tolerance:** Caching reduces load on the DB, preventing it from failing. DBs use replicas and backups for data safety.
  * **Latency:** Cache (e.g., \< 5ms). Database (e.g., 10ms - 100ms+).

### 6\. $\rightarrow$ Response (The path back)

  * The data (e.g., User JSON) flows all the way back up the chain:
    `DB → Service → API GW → LB → Client`
  * The client's browser receives the JSON and renders the UI.
  * **Total Latency:** The sum of all steps (e.g., 50ms (network) + 1ms (LB) + 20ms (GW) + 30ms (Service) + 10ms (Cache) = **111ms**).

-----

## 6\. Trade-offs in System Architecture

There is **no perfect architecture**. Every decision is a trade-off.

### Consistency vs. Availability (CAP Theorem)

  * **Definition:** In a distributed system, you can only choose two of the following three:
      * **C**onsistency: Every read receives the most recent write or an error.
      * **A**vailability: Every request receives a (non-error) response, without guaranteeing it contains the most recent write.
      * **P**artition Tolerance: The system continues to operate despite network failures (partitions) between nodes.
  * **The Trade-off:** Because network partitions (P) are a fact of life, you must choose between Consistency (C) and Availability (A).
      * **CP (Choose C over A):** A bank. If the network splits, the "minority" partition *must* shut down to prevent inconsistent data (e.g., two people withdrawing the same money). **Example: RDBMS in strong consistency mode.**
      * **AP (Choose A over C):** A social media 'like' count. If the network splits, it's better to let users *both* 'like' the post and be available. The counts will be merged later (**eventual consistency**). **Example: Cassandra, DynamoDB.**

### Latency vs. Cost

  * **The Trade-off:** Low latency costs more.
  * **Example 1: Caching.** A large in-memory (RAM) cache (like Redis) is very fast (low latency) but RAM is expensive. Storing data on SSDs is cheaper but slower.
  * **Example 2: CDN.** A global CDN with 300 edge locations gives very low latency worldwide but is expensive. A CDN with 10 locations is cheaper but users far from those locations will have higher latency.

### Complexity vs. Flexibility

  * **The Trade-off:** More flexible systems are almost always more complex to manage.
  * **Example: Microservices.** They are highly *flexible* (each team can deploy, scale, and write in any language). They are also highly *complex* (you now have a distributed system, network latency, data consistency problems, and need a DevOps team).
  * **Example: Monolith.** It is *inflexible* (one tech stack, one deployment) but *simple* (one codebase, one server to run).

### Eventual Consistency

  * **Definition:** A consistency model where, given no new updates, all replicas of a piece of data will *eventually* converge to the same value. It's the "AP" choice from CAP.
  * **Example:** You upload a video to YouTube. It may take 10 minutes for it to be available to all users globally. The system is highly available (you can upload, your friends can watch *other* videos), but not strongly consistent. This is acceptable.
  * **Counter-Example:** A bank transfer. You *cannot* have eventual consistency. It must be **strongly consistent**.

### Strong vs. Weak (Loose) Coupling

  * **Coupling:** The degree to which components depend on each other.
  * **Strong Coupling:** Service A calls Service B's API *directly*.
      * **Problem:** If Service B is slow, Service A becomes slow. If Service B is down, Service A fails.
  * **Loose Coupling:** Service A publishes an *event* to a Message Queue. Service B subscribes to that event.
      * **Advantage:** Service A and B don't even know about each other. If Service B is down, the message just waits in the queue. This is **much more resilient**.

### Monolith vs. Microservices Trade-off (Summary)

| Factor | Monolith | Microservices |
| :--- | :--- | :--- |
| **Initial Development** | **Fast** | Slow |
| **Operational Complexity** | **Low** | High |
| **Scalability** | Coarse-grained (all or nothing) | **Fine-grained (per-service)** |
| **Resilience** | Low (one bug crashes all) | **High (if built right)** |
| **Team Organization** | Works for one team | **Works for many teams** |
| **Tech Stack** | Single stack | **Polyglot (many stacks)** |
| **Data Consistency** | **Strong (ACID)** | Eventual (Hard) |

-----

## 7\. Mini System Design Exercises

### A. URL Shortener (e.g., bit.ly)

**1. Requirements:**

  * **Functional:**
    1.  User provides a long URL.
    2.  System returns a short, unique URL.
    3.  User hits the short URL.
    4.  System redirects them to the original long URL.
  * **Non-Functional:**
    1.  **Low Latency:** Redirects must be *extremely* fast (read-heavy).
    2.  **High Availability:** The redirect service must always be up.
    3.  **Scalability:** Must handle millions of redirects per day.

**2. Architecture Diagram:**

```ascii
+--------+   (Write)
| Client | POST /shorten
+--------+     |
               v
+-----------------------+
|    Load Balancer      |
+-----------------------+
              |
              v
+-----------------------+
| URL Shortener Service | (App Logic)
|(Generates 6-char hash)|
+-----------------------+
              |
              v
+-----------------------+
|  Database (e.g., SQL) | (Stores hash <-> long_url)
|  (Indexed on 'hash')  |
+-----------------------+

---------------------------------------------------

+--------+   (Read)
| Client | GET /{short_hash}
+--------+      |
                v
+-----------------------+
|    Load Balancer      |
+-----------------------+
              |
              v
+-----------------------+  (Cache Check)   +-------------------+
|  Redirect Service     | ---------------> |   Cache (Redis)   |
|(Read-Only, Scaled 10x)| <--------------- |(Stores hash:long_url)|
+-----------------------+  (Cache Miss)    +-------------------+
              |
              v
+-----------------------+
| Database(Read Replica)|
+-----------------------+
```

**3. DB Schema:**

  * A single SQL table is fine. We need a strong index on the `short_hash`.
    ```sql
    CREATE TABLE urls (
      short_hash VARCHAR(8) PRIMARY KEY,
      long_url TEXT NOT NULL,
      created_at TIMESTAMPO
    );
    CREATE UNIQUE INDEX idx_short_hash ON urls(short_hash);
    ```

**4. Scaling Approach:**

  * This is a **read-heavy** system.
  * Scale the `Redirect Service` horizontally (e.g., 100 instances).
  * The `URL Shortener Service` (the write part) doesn't need as many instances.
  * Use Database **Read Replicas** for the read path.

**5. Caching:**

  * **This is the most important part.**
  * Cache *aggressively*.
  * When a user hits `GET /{short_hash}`:
    1.  Check Redis: `GET short_hash`.
    2.  **Hit:** Return the cached `long_url` immediately. (Fastest)
    3.  **Miss:** Query the DB Read Replica, get the `long_url`, store it in Redis `SET short_hash long_url`, and then return.

**6. Consistency Model:**

  * **Eventual Consistency** is perfectly fine.
  * When a user creates a new short URL, it's okay if it takes a few seconds (or even a minute) to propagate to all edge caches/DB replicas. The *write* operation doesn't need to be instant, but the *read* does.

**7. API Design:**

  * `POST /api/v1/shorten`
      * Body: `{ "url": "http://very-long-url.com/..." }`
      * Response (201): `{ "short_url": "http://sho.rt/aB3xZ1" }`
  * `GET /{short_hash}` (e.g., `GET /aB3xZ1`)
      * Response: `HTTP 301 Moved Permanently`
      * Header: `Location: http://very-long-url.com/...`

**8. Edge Cases:**

  * **Hash Collisions:** What if two URLs generate the same hash? The service must re-try generation with a new salt or check the DB for existence.
  * **Vanity URLs:** `POST /api/v1/shorten` body: `{ "url": "...", "custom": "my-cool-link" }`. The service must check if "my-cool-link" is taken.

**9. Trade-offs:**

  * **Hash Length:** A 6-character (Base62) hash gives $\approx 56$ billion combos. A 7-character hash gives $\approx 3.5$ trillion. We trade a slightly longer URL for a *much* lower collision probability.
  * **301 vs 302 Redirect:** `301 (Permanent)` is better. It tells browsers/crawlers to cache the redirect *themselves*, reducing load on our system.

-----

### B. Real-Time Chat System (e.g., WhatsApp/Slack)

**1. Requirements:**

  * **Functional:**
    1.  Users can send/receive 1-on-1 messages in real-time.
    2.  Users can see message history.
    3.  Users can see "online/offline" presence status.
  * **Non-Functional:**
    1.  **Low Latency:** Messages must appear instantly.
    2.  **Scalable:** Handle millions of concurrent connections.
    3.  **Reliable:** Don't lose messages.

**2. Architecture Diagram:**

```ascii
+----------+                               +----------------------+
| Client 1 | <---- (WebSocket Conn) ---->  |                      |
+----------+                               | WebSocket Gateway    |
                                           | (Manages connections)|
+----------+                               |                      |
| Client 2 | <---- (WebSocket Conn) ---->  |                      |
+----------+                               +----------+-----------+
                                                      |
                                     (New Message)    | (Fan-out)
                                                      v
+-------------------+                           +----------------------+
| Chat Service      | <------ (HTTP API) ------ | API Gateway / LB     |
| (Business Logic)  |                           +----------------------+
| (Stores Message)  |                                      ^
+---------+---------+                                      |
          |                                       (HTTP API calls)
          v                                                |
+-------------------+                           +----------------------+
|  Database         |                           |   Presence Service   |
| (e.g., Cassandra) |                           |   (Uses Redis)       |
| (Stores messages) |                           +----------------------+
+-------------------+
```

**3. DB Schema:**

  * This is a high-write system. A NoSQL (wide-column) database is ideal.
  * **`messages` table (in Cassandra):**
      * `chat_id` (Partition Key) - (e.g., `user1:user2`)
      * `message_id` (Clustering Key, TimeUUID)
      * `sender_id`
      * `body` (text)
      * `created_at`

**4. Scaling Approach:**

  * The hardest part is **connection state**.
  * The **WebSocket Gateway** is a stateful service. We need many instances of it.
  * We need a way to map `user_id` to *which Gateway instance* they are connected to. (e.g., using Redis: `SET user:123 gateway:server_5`).
  * **Fan-out:** When User 1 sends a message:
    1.  Message goes to `Chat Service` via API GW.
    2.  `Chat Service` saves to DB.
    3.  `Chat Service` looks up User 2's connection: `GET user:456` $\rightarrow$ `gateway:server_8`.
    4.  `Chat Service` pushes the message to `gateway:server_8`, which sends it down User 2's WebSocket.

**5. Caching:**

  * Use Redis for **Presence Status** (online/offline/typing). This data changes constantly and doesn't need long-term storage.
  * Use Redis for mapping `user_id` $\rightarrow$ `websocket_server_id`.

**6. Consistency Model:**

  * **Eventual Consistency** for presence. It's okay if a user appears "online" for a few seconds after they disconnect.
  * **Strong-ish Consistency** for messages. We cannot lose messages. The write to the database must be durable.

**7. API Design:**

  * **WebSocket (Main Channel):**
      * `ws://chat.example.com/connect?token=...`
      * Client sends: `{ "type": "message", "to": "user_456", "body": "Hello!" }`
      * Server pushes: `{ "type": "message", "from": "user_123", "body": "Hello!" }`
  * **HTTP API (for history):**
      * `GET /api/v1/chats/{user_id}/messages?page=...`

**8. Edge Cases:**

  * **User Offline:** If User 2 is offline, the message is saved to the DB. When User 2 reconnects, the client must *pull* all messages from the API (`GET /messages`) that it missed.
  * **"Typing" Indicator:** This is a high-volume, low-importance event. Send it directly over WebSockets *without* saving to the DB.

**9. Trade-offs:**

  * **WebSocket vs. Long Polling:** WebSockets are more efficient (one long-lived connection) but harder to scale (stateful). Long polling is stateless (HTTP) but less efficient (many requests). We choose WebSockets for a true real-time feel.

-----

## 8\. Additional Example Architectures

### A. Full Real-time Chat Architecture (Recap & Enhancement)

  * **Components:**
    1.  **Client (Mobile/Web):** Maintains a persistent WebSocket connection.
    2.  **Load Balancer:** Standard LB, but must support "sticky sessions" if the gateways are not globally aware.
    3.  **WebSocket Gateway Service:** A cluster of stateful servers (e.g., built with Go, Elixir, or Node.js) that manage persistent client connections.
    4.  **Presence Service:** Uses a fast K/V store (Redis) with short TTLs (Time-to-Live) to track online status. Clients send a "heartbeat" every 30 seconds.
    5.  **Chat API Service:** Stateless HTTP service for sending messages, getting history, creating groups.
    6.  **Message Queue (Kafka):** Used to *decouple* message sending from message delivery.
    7.  **Message DB:** A high-write NoSQL DB (Cassandra/DynamoDB) to store all chat history.
  * **Flow (Fan-out on Write):**
    1.  Client A sends message $\rightarrow$ Chat API Service.
    2.  API Service validates, saves to Message DB.
    3.  API Service publishes message to Kafka (Topic: `group_123_messages`).
    4.  A set of "Fan-out" workers subscribe to Kafka.
    5.  The worker looks up all *online* members of `group_123` (from Presence Service).
    6.  For each online member, it finds their connected WebSocket Gateway (from Redis).
    7.  It pushes the message to the *correct* Gateway, which delivers it to the *correct* client.
  * **Failure Handling:** If a client is offline, the message is still in Kafka (and the DB). When they connect, they pull history from the Chat API. This ensures *guaranteed delivery*.

### B. UPI Payment Flow

  * **Components:**
    1.  **PSP App:** (e.g., Google Pay, PhonePe) - The client app.
    2.  **PSP Server:** The backend for the Google Pay/PhonePe app.
    3.  **NPCI Switch:** The central router/regulator for all UPI transactions.
    4.  **Payer Bank:** The user's bank (where money is debited).
    5.  **Payee Bank:** The merchant's/receiver's bank (where money is credited).
  * **Flow Diagram (Simplified):**
    ```ascii
    +----------+   +----------+   +---------+   +------------+   +----------+
    | PSP App  | ->| PSP Serv | ->|   NPCI  | ->| Payer Bank | ->|   NPCI   |
    |(Initiate)|   | (Sign)   |   | (Route) |   | (Debit OK?)|   | (Confirm)|
    +----------+   +----------+   +---------+   +------------+   +----------+
                                                                   |
                                                                   v
    +--------+   +----------+   +---------+   +-----------------+   +----------+
    | PSP App  | <-| PSP Serv | <-|   NPCI    |   | Payee Bank  | <-|   NPCI     |
    | (Success)|   | (Notify) |   | (Route)   |   | (Credit OK) |   | (Forward)  |
    +--------+   +----------+   +---------+   +-----------------+   +----------+
    ```
  * **Data Consistency:** **MUST be strongly consistent** (ACID). This is handled using a 2-Phase Commit-like protocol between the banks and NPCI. A transaction is not "done" until all parties (debit and credit) have confirmed.
  * **Event-driven?** Yes, but in a *synchronous, orchestrated* way. Each step is an "event" that triggers the next, but the entire flow must complete (or fail cleanly) in real-time (under 10 seconds).
  * **Failure Handling:** Extremely robust. If any step fails (e.g., Payee Bank is down), the entire transaction is rolled back. The Payer Bank must *reverse* the debit. This leads to "Amount debited, transaction failed" states, which are reconciled later.

### C. Notification System Architecture

  * **Components:**
    1.  **Notification API:** A simple, stateless HTTP service that receives requests: `POST /notify` with body `{ "user_id": "123", "channel": "email", "template_id": "welcome_email", ... }`.
    2.  **Message Queue(s):** The most critical part. The API *only* drops the request into a queue and returns `202 Accepted`.
    3.  **Channel Workers:** Separate worker fleets for each channel.
          * `Email_Worker` (consumes from `email_queue`)
          * `SMS_Worker` (consumes from `sms_queue`)
          * `Push_Worker` (consumes from `push_queue`)
    4.  **External Gateways:** (e.g., SendGrid for email, Twilio for SMS, APNS/FCM for push).
    5.  **Templates DB:** Stores the content of the notifications.
  * **Flow Diagram:**
    ```ascii
    +--------+   +---------------+   +---------------+
    | Client | ->| Notification  | ->| Kafka/SQS     |
    | (e.g., |   | API           |   | (email_queue) |
    | OrderSvc)| +---------------+   +-------+-------+
    +--------+                             |
                                           v
    +---------------+   +--------------+   +---------------+
    | Email Worker  | ->| Template DB  | ->| SendGrid      |
    | (Consumes msg)|   | (Get template) | |(External API) |
    +---------------+   +--------------+   +---------------+
    ```
  * **Performance:** The API is ultra-fast because it's *asynchronous*. It just accepts the request and moves on. The user never waits for the email to actually be sent.
  * **Failure Handling:**
      * **Rate Limiting:** The *workers* are rate-limited, not the API. This prevents sending 1 million emails at once and getting blocked by SendGrid.
      * **Retries:** If SendGrid's API is down, the Email Worker fails to process the message. The queue ensures it will be *retried* (e.g., 1 min, 5 min, 10 min later).
      * **Dead Letter Queue (DLQ):** If a message fails 10 times (e.g., "Email address does not exist"), it's moved to a DLQ for manual review, so it doesn't block the queue forever.
