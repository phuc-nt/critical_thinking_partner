# Architecture Proposal: "Project Speedboat"
**Target Architecture**: Event-Driven Microservices
**Timeline**: 12 Months

## 1. Vision
Decompose the `LogiTrack` monolith into domain-driven microservices communicated via asynchronous events.

## 2. Key Architecture Components

### 2.1 Domain Services (The Core)
We will split by functionality:
1.  **Order Service (Go)**: High throughput, manages order lifecycle.
2.  **Dispatch Service (Python)**: Uses OR-Tools for routing algorithms. CPU intensive.
3.  **Driver Service (Node.js)**: Real-time websocket connection for app updates.
4.  **Billing Service (Java)**: Strong typing, reliable library support for PDF/Math.

### 2.2 Event Bus and Broker
- **Technology**: Apache Kafka.
- **Pattern**: Choreography (mostly) + Orchestration (for Saga transactions).
- **Topics**:
  - `order.created`
  - `driver.assigned`
  - `delivery.completed`

### 2.3 Database Strategy (Database-per-Service)
- No more shared Postgres.
- **Order Service** -> PostgreSQL.
- **Driver Service** -> MongoDB (GeoJSON support).
- **Dispatch Service** -> Redis (Ephemeral state).

### 2.4 API Gateway
- **Kong** or **AWS API Gateway**.
- Unified entry point for all clients.
- Handles AuthN/AuthZ (JWT).

## 3. Migration Strategy (The "Strangler Fig" Pattern)

### Phase 1 (Months 1-3)
- Build the **Event Bus**.
- Implement "CDC" (Change Data Capture) on the Legacy DB using Debezium.
- This publishes legacy DB changes to Kafka so new services can consume without existing services knowing.

### Phase 2 (Months 4-6) - "The Driver Module"
- Carve out **Driver Service** first.
- Redirect mobile app traffic to new API Gateway -> Driver Service.
- Sync data back to Monolith for read-compatibility.

### Phase 3 (Months 7-9) - " The Dispatch Brain"
- Move routing logic to Python microservice.
- This is the riskiest move as it touches the core IP.

### Phase 4 (Months 10-12)
- Retire the Monolith.
- Celebrate?

## 4. Expected Benefits
- **Independent Scaling**: Scale "Dispatch" service on GPU instances without paying for the rest.
- **Technology Freedom**: Use Python for Math, Go for Speed.
- **Resilience**: If Billing Service goes down, Drivers can still deliver packages (Eventual Consistency).

## 5. New Complexity (The Trade-offs)
- **Distributed Tracing**: Need to implement Jaeger/OpenTelemetry.
- **Data Consistency**: No more ACID transactions across domains. We must handle SAGA rollbacks.
- **DevOps**: Need a dedicated Platform Team to manage Kubernetes/Helm charts.
