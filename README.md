# Uber-like Ride Hailing --- Microservices Side Project

> A backend-focused ride-hailing system inspired by Uber/Grab, built as
> a learning project to practice **Java Backend, Microservices, Redis,
> Kafka, Distributed Systems, and System Design**.

## 1. Project Overview

This project simulates the core backend of a ride-hailing platform:

-   Rider requests a ride.
-   Drivers continuously send their GPS location.
-   The system finds nearby available drivers.
-   A matching service selects a suitable driver.
-   Ride state is persisted and managed.
-   Kafka is used for asynchronous event-driven communication.
-   Redis is used for real-time location and geo-spatial queries.
-   API Gateway provides a single entry point for clients.

The project is intentionally designed to evolve from a simple backend
into a distributed microservices system.

------------------------------------------------------------------------

## 2. Architecture

``` mermaid
flowchart TB
    C[Client<br/>Mobile / Web<br/>User / Driver]

    G[API Gateway<br/>Routing • Auth • Rate Limit]

    L[Location Service<br/>GPS • Location Management]
    M[Matching Service<br/>Driver-Rider Matching]
    R[Ride Service<br/>Booking • Status • Trip]

    REDIS[(Redis<br/>Geo Data)]
    DB[(PostgreSQL / MySQL)]

    K[(Kafka<br/>Event Streaming / Message Broker)]

    C -->|HTTP / HTTPS| G

    G -->|GPS real-time location| L
    G -->|Find driver / match request| M
    G -->|Ride APIs| R

    L <--> REDIS
    R <--> DB

    L -->|Location / Driver events| K
    M -->|Matching events| K
    R -->|Ride events| K

    K -->|Events| M
    K -->|Events| R
```

### Main components

  -----------------------------------------------------------------------
  Component                           Responsibility
  ----------------------------------- -----------------------------------
  **Client**                          Rider/driver application

  **API Gateway**                     Routing, authentication, rate
                                      limiting and centralized entry
                                      point

  **Location Service**                Receive GPS updates, store/query
                                      driver locations

  **Matching Service**                Find and select a suitable driver

  **Ride Service**                    Ride lifecycle, booking, status and
                                      trip management

  **Redis**                           Real-time location and GEO queries

  **PostgreSQL/MySQL**                Persistent transactional data

  **Kafka**                           Event streaming and asynchronous
                                      communication
  -----------------------------------------------------------------------

------------------------------------------------------------------------

## 3. High-Level Request Flow

### 3.1 Driver location update

``` text
Driver App
    │
    │ GPS coordinates
    ▼
API Gateway
    │
    ▼
Location Service
    │
    ├──► Redis GEO
    │
    └──► Kafka
             │
             └──► Matching Service
```

Example:

``` json
{
  "driverId": 1001,
  "latitude": 20.994,
  "longitude": 105.812,
  "timestamp": "2026-09-24T15:00:00Z"
}
```

------------------------------------------------------------------------

### 3.2 Rider requests a ride

``` text
Rider
  │
  │ POST /rides
  ▼
API Gateway
  │
  ▼
Ride Service
  │
  │ RideRequested
  ▼
Kafka
  │
  ▼
Matching Service
  │
  │ Search nearby available drivers
  ▼
Redis GEO
  │
  ▼
Driver selected
```

------------------------------------------------------------------------

### 3.3 Driver matching

The initial matching algorithm can be simple:

``` text
1. Find nearby drivers
2. Filter AVAILABLE drivers
3. Calculate distance
4. Select the nearest driver
5. Reserve the driver
6. Publish DriverMatched event
```

The algorithm can later be extended with:

-   Distance
-   Driver rating
-   Driver waiting time
-   Vehicle type
-   Estimated arrival time
-   Driver availability
-   Surge/priority rules

------------------------------------------------------------------------

## 4. Ride Lifecycle

A ride can follow this state machine:

``` text
REQUESTED
    │
    ▼
SEARCHING_DRIVER
    │
    ▼
DRIVER_ASSIGNED
    │
    ▼
DRIVER_ACCEPTED
    │
    ▼
DRIVER_ARRIVING
    │
    ▼
DRIVER_ARRIVED
    │
    ▼
TRIP_STARTED
    │
    ▼
TRIP_COMPLETED
```

Alternative terminal states:

``` text
REQUESTED ─────► CANCELLED
SEARCHING ─────► NO_DRIVER_FOUND
DRIVER_ASSIGNED ► CANCELLED
```

The service should reject invalid state transitions.

------------------------------------------------------------------------

## 5. Kafka Events

Kafka is used as the event backbone between services.

Example events:

  Event                     Producer           Consumer
  ------------------------- ------------------ --------------------------
  `DriverLocationUpdated`   Location Service   Matching Service
  `RideRequested`           Ride Service       Matching Service
  `DriverMatched`           Matching Service   Ride Service
  `DriverAccepted`          Ride Service       Notification Service
  `RideStarted`             Ride Service       Notification / Analytics
  `RideCompleted`           Ride Service       Notification / Analytics
  `RideCancelled`           Ride Service       Notification / Analytics

Example event:

``` json
{
  "eventId": "7f8c2d10-...",
  "eventType": "RideRequested",
  "timestamp": "2026-09-24T15:10:00Z",
  "payload": {
    "rideId": 10001,
    "riderId": 20001,
    "pickup": {
      "latitude": 20.994,
      "longitude": 105.812
    },
    "destination": {
      "latitude": 21.028,
      "longitude": 105.834
    }
  }
}
```

------------------------------------------------------------------------

## 6. Redis GEO

Redis is used for real-time driver location.

Conceptually:

``` text
driver:1001 → (longitude, latitude)
driver:1002 → (longitude, latitude)
driver:1003 → (longitude, latitude)
```

Example Redis command:

``` text
GEOADD drivers 105.812 20.994 driver:1001
```

Search nearby drivers:

``` text
GEOSEARCH drivers
  FROMLONLAT 105.812 20.994
  BYRADIUS 5 km
  ASC
```

Redis is suitable here because location data is:

-   Frequently updated
-   Frequently queried
-   Time-sensitive
-   Suitable for geo-spatial indexing

Persistent business data should remain in the relational database.

------------------------------------------------------------------------

## 7. Microservice Responsibilities

### Location Service

Responsibilities:

-   Receive driver GPS updates
-   Validate coordinates
-   Store current location in Redis
-   Find nearby drivers
-   Publish location-related events

Possible APIs:

``` http
POST /api/v1/locations/drivers/{driverId}
GET  /api/v1/locations/drivers/nearby
GET  /api/v1/locations/drivers/{driverId}
```

------------------------------------------------------------------------

### Matching Service

Responsibilities:

-   Consume ride requests
-   Find nearby available drivers
-   Apply matching algorithm
-   Reserve a driver
-   Publish matching result

Possible APIs:

``` http
POST /api/v1/matching/rides/{rideId}
GET  /api/v1/matching/rides/{rideId}
```

The matching service should eventually be mostly event-driven rather
than requiring synchronous communication with every other service.

------------------------------------------------------------------------

### Ride Service

Responsibilities:

-   Create rides
-   Manage ride state
-   Store ride information
-   Validate state transitions
-   Publish ride events
-   Consume matching results

Possible APIs:

``` http
POST /api/v1/rides
GET  /api/v1/rides/{rideId}
POST /api/v1/rides/{rideId}/cancel
POST /api/v1/rides/{rideId}/accept
POST /api/v1/rides/{rideId}/start
POST /api/v1/rides/{rideId}/complete
```

------------------------------------------------------------------------

## 8. Database Ownership

Each service should own its data.

Recommended direction:

``` text
User Service
    └── user_db

Ride Service
    └── ride_db

Location Service
    └── Redis

Matching Service
    └── Redis / its own persistence when required
```

Avoid tightly coupling all microservices to one shared database.

The goal is to practice:

> **Service owns its data and exposes business capabilities through
> APIs/events.**

------------------------------------------------------------------------

## 9. Technology Stack

### Backend

-   Java 21+
-   Spring Boot
-   Spring Web
-   Spring Data JPA
-   Spring Security
-   Spring Validation
-   Spring Cloud Gateway

### Data

-   PostgreSQL
-   Redis
-   Redis GEO

### Messaging

-   Apache Kafka

### Infrastructure

-   Docker
-   Docker Compose

### Testing

-   JUnit 5
-   Mockito
-   Spring Boot Test
-   Testcontainers

### Quality / Operations

-   Maven
-   Git
-   GitHub/GitLab
-   CI/CD
-   Prometheus
-   Grafana
-   Structured logging

------------------------------------------------------------------------

## 10. Development Roadmap

Do **not** implement the complete architecture immediately.

### Phase 1 --- Backend Foundation

``` text
Spring Boot
    │
    └── PostgreSQL
```

Implement:

-   User
-   Driver
-   Ride
-   REST API
-   DTO
-   Validation
-   Global Exception Handler
-   JPA/Hibernate
-   Transaction
-   Pagination
-   Database indexes

------------------------------------------------------------------------

### Phase 2 --- Security

Implement:

-   JWT authentication
-   Spring Security
-   Role-based authorization
-   Rider/Driver roles
-   Refresh token
-   Authentication error handling

------------------------------------------------------------------------

### Phase 3 --- Redis

Add:

``` text
Spring Boot
    ├── PostgreSQL
    └── Redis
```

Implement:

-   Cache
-   Driver online/offline state
-   Driver location
-   Redis GEO
-   Nearby driver search

------------------------------------------------------------------------

### Phase 4 --- Microservices

Split the monolith:

``` text
API Gateway
    │
    ├── User Service
    ├── Location Service
    ├── Ride Service
    └── Matching Service
```

Focus on:

-   Service boundaries
-   API contracts
-   Database ownership
-   Configuration
-   Inter-service communication
-   Failure handling

------------------------------------------------------------------------

### Phase 5 --- Kafka

Introduce event-driven communication:

``` text
Location Service ──┐
                   │
Ride Service ──────┼──► Kafka
                   │
Matching Service ──┘
```

Learn:

-   Producer
-   Consumer
-   Consumer Group
-   Partition
-   Offset
-   Key
-   Ordering
-   Retry
-   Dead Letter Topic
-   At-least-once delivery
-   Idempotent consumer

------------------------------------------------------------------------

### Phase 6 --- Real-time

Add WebSocket:

``` text
Driver
  │
  │ GPS
  ▼
WebSocket
  │
  ▼
Location Service
  │
  ├──► Redis
  └──► Kafka
```

Use it for:

-   Driver location
-   Ride status
-   Driver assigned notification
-   Driver arrival notification

------------------------------------------------------------------------

### Phase 7 --- Distributed System Patterns

Implement:

-   Idempotency
-   Retry
-   Timeout
-   Circuit Breaker
-   Dead Letter Queue
-   Outbox Pattern
-   Optimistic Locking
-   Distributed transaction analysis
-   Eventual consistency

------------------------------------------------------------------------

### Phase 8 --- Observability

Add:

``` text
Application
    │
    ├── Logs
    ├── Metrics
    └── Traces
```

Technologies:

-   Prometheus
-   Grafana
-   OpenTelemetry
-   Micrometer

Monitor:

-   API latency
-   Error rate
-   Kafka consumer lag
-   Redis performance
-   Database connection pool
-   JVM metrics

------------------------------------------------------------------------

## 11. Recommended Project Structure

For a Maven multi-module project:

``` text
uber/
├── pom.xml
├── README.md
├── .gitignore
│
├── api-gateway/
│   ├── pom.xml
│   └── src/
│
├── user-service/
│   ├── pom.xml
│   └── src/
│
├── location-service/
│   ├── pom.xml
│   └── src/
│
├── matching-service/
│   ├── pom.xml
│   └── src/
│
├── ride-service/
│   ├── pom.xml
│   └── src/
│
├── notification-service/
│   ├── pom.xml
│   └── src/
│
├── common/
│   ├── pom.xml
│   └── src/
│
└── infrastructure/
    ├── docker-compose.yml
    ├── kafka/
    ├── postgres/
    └── redis/
```

`common` should contain only genuinely shared infrastructure/contracts.
Avoid turning it into a dumping ground for business logic.

------------------------------------------------------------------------

## 12. Non-Functional Requirements

The project should eventually consider:

### Scalability

-   Stateless services
-   Horizontal scaling
-   Kafka partitioning
-   Redis scalability
-   Database indexing
-   Connection pooling

### Reliability

-   Retry
-   Timeout
-   Circuit breaker
-   Idempotency
-   DLQ
-   Graceful degradation

### Consistency

Understand the difference between:

``` text
Strong consistency
        vs
Eventual consistency
```

and decide which model is appropriate for each business operation.

### Security

-   JWT
-   Password hashing
-   RBAC
-   Input validation
-   Rate limiting
-   HTTPS
-   Secret management

------------------------------------------------------------------------

## 13. Example Business Scenario

### Request a ride

``` text
1. Rider sends POST /rides
2. API Gateway authenticates the request
3. Ride Service creates a REQUESTED ride
4. Ride Service publishes RideRequested
5. Matching Service consumes RideRequested
6. Matching Service searches Redis GEO
7. Matching Service filters available drivers
8. Matching algorithm selects a driver
9. Matching Service publishes DriverMatched
10. Ride Service updates ride → DRIVER_ASSIGNED
11. Driver receives the request
12. Driver accepts
13. Ride Service publishes DriverAccepted
14. Driver arrives
15. Trip starts
16. Trip completes
17. Ride Service publishes RideCompleted
```

------------------------------------------------------------------------

## 14. Learning Objectives

This project is not only intended to produce a working application.

The main learning objectives are:

### Java / Spring Boot

-   Clean code
-   SOLID
-   Design patterns
-   Dependency Injection
-   REST API design
-   Exception handling
-   Transaction management
-   Testing

### Database

-   SQL
-   Indexing
-   Query optimization
-   Transaction isolation
-   Locking
-   Data modeling

### Redis

-   Cache
-   TTL
-   GEO
-   Distributed data structures

### Kafka

-   Event-driven architecture
-   Partition
-   Consumer Group
-   Offset
-   Ordering
-   Retry
-   Idempotency

### Microservices

-   Service boundaries
-   Data ownership
-   API Gateway
-   Inter-service communication
-   Failure handling
-   Eventual consistency

### System Design

-   Scalability
-   Availability
-   Reliability
-   Bottlenecks
-   Caching
-   Message queues
-   Observability

------------------------------------------------------------------------

## 15. Definition of Done

A feature should not be considered complete just because its API works.

For important features, aim to have:

``` text
Code
 ├── Unit Test
 ├── Integration Test
 ├── Validation
 ├── Error Handling
 ├── Logging
 ├── Metrics
 └── Documentation
```

For distributed features:

``` text
Distributed Feature
 ├── Timeout
 ├── Retry
 ├── Idempotency
 ├── Failure Scenario
 ├── Observability
 └── Recovery Strategy
```

------------------------------------------------------------------------

## 16. Project Goal

The final goal is to build a realistic backend system while
progressively learning:

``` text
Junior Backend
      │
      ▼
Spring Boot
      │
      ▼
Database + Redis
      │
      ▼
Microservices
      │
      ▼
Kafka / Event Driven
      │
      ▼
Real-time System
      │
      ▼
Distributed System
      │
      ▼
Middle-level Backend Skills
```

> **Principle:** build simple first, understand the problem, then
> introduce complexity only when the system has a reason to need it.
