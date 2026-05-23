# FoodChain

A cloud-native restaurant management platform built across **10 microservices** — handling ordering, kitchen operations, menu management, branch coordination, analytics, and real-time notifications. Available on web and mobile (React Native).

All source code is maintained across multiple repositories under the **[FoodchainGroup08](https://github.com/FoodchainGroup08)** GitHub organisation.

---

## Architecture Overview

- **Pattern:** Microservices behind a single API Gateway entry point
- **Service Discovery:** Netflix Eureka — all services self-register and load-balance via `lb://`
- **Configuration:** Centralised via Spring Cloud Config Server pulling from a private GitHub repository
- **Messaging:** Event-driven via Apache Kafka — order lifecycle events flow between services asynchronously
- **Real-time:** WebSocket (STOMP) for live kitchen updates and customer order notifications
- **Deployment:** Docker containers across 2 AWS EC2 accounts, automated via GitHub Actions CI/CD

---

## Tech Stack

| Layer | Technology |
|---|---|
| Language | Java 17, 21, 25 |
| Framework | Spring Boot 3, Spring Cloud |
| Gateway | Spring Cloud Gateway |
| Service Discovery | Netflix Eureka |
| Database | MySQL 8.0 (AWS RDS) |
| Cache / Queue | Redis |
| Messaging | Apache Kafka |
| Frontend | React + Vite |
| Mobile | React Native |
| Containerisation | Docker + Docker Hub |
| CI/CD | GitHub Actions |
| Infrastructure | AWS EC2, RDS, S3 |

---

## Third-Party Services

| Service | Purpose |
|---|---|
| AWS EC2 × 2 (t3.medium) | Hosts all backend Docker containers |
| AWS RDS MySQL 8.0 | Persistent storage — 6 separate schemas on one instance |
| AWS S3 | Menu item image storage |
| Confluent Cloud | Managed Kafka broker (SASL_SSL) |
| Redis Cloud | Cache, sessions, and kitchen order queue |
| Google Gemini AI | Previously used for food suggestions — replaced by preference-based recommendation engine |
| Google Maps API | Branch location search and nearby branch discovery |
| Google OAuth2 | Social login for customer accounts |
| Brevo | Transactional email delivery |
| Namecheap | Domain registrar — foodchain.live |
| Cloudflare | Frontend hosting and DNS |
| Docker Hub | Container image registry |

---

## Repository Structure

| Repository | Function |
|---|---|
| [eureka-server](https://github.com/FoodchainGroup08/eureka-server) | Service registry — all services register and discover each other here |
| [config-server](https://github.com/FoodchainGroup08/config-server) | Spring Cloud Config Server — serves centralised config to all services |
| [foodchain-config](https://github.com/FoodchainGroup08/foodchain-config) | Centralised YAML config files pulled by config-server at runtime |
| [foodchain-deployment](https://github.com/FoodchainGroup08/foodchain-deployment) | Docker Compose files and GitHub Actions CI/CD workflows |
| [api-gateway](https://github.com/FoodchainGroup08/api-gateway) | Entry point — JWT auth, routing, CORS, WebSocket proxy |
| [user-service](https://github.com/FoodchainGroup08/user-service) | Authentication, Google OAuth2, user profiles and roles |
| [branch-service](https://github.com/FoodchainGroup08/branch-service) | Branch management, operating hours, and location |
| [menu-service](https://github.com/FoodchainGroup08/menu-service) | Menu items, categories, image uploads, and preference-based combo recommendations |
| [order-service](https://github.com/FoodchainGroup08/order-service) | Order placement, lifecycle management, and outbox events |
| [kitchen-service](https://github.com/FoodchainGroup08/kitchen-service) | Kitchen order queue and real-time status updates |
| [notifications-service](https://github.com/FoodchainGroup08/notifications-service) | WebSocket push notifications and transactional email alerts |
| [analytics-report-service](https://github.com/FoodchainGroup08/analytics-report-service) | Sales analytics, daily summaries, and report generation |
| [frontend](https://github.com/FoodchainGroup08/frontend) | React/Vite UI for customers and staff |
| [foodchain-mobile](https://github.com/FoodchainGroup08/foodchain-mobile) | React Native mobile app for customers and staff |

---

## Service Map

| Service | Port | URL |
|---|---|---|
| Frontend (Cloudflare) | — | [foodchain.live](https://foodchain.live) |
| API Gateway | 8080 | [api.foodchain.live](https://api.foodchain.live) |
| API Docs (Swagger) | — | [api.foodchain.live/webjars/swagger-ui/index.html](https://api.foodchain.live/webjars/swagger-ui/index.html) |
| User Service | 8086 | Internal |
| Branch Service | 8081 | Internal |
| Menu Service | 8082 | Internal |
| Order Service | 8083 | Internal |
| Kitchen Service | 8084 | Internal |
| Analytics Service | 8085 | Internal |
| Notifications Service | 8087 | Internal |
| Config Server | 8888 | Internal |
| Eureka Server | 8761 | Internal |

---

## Architecture Diagram

```mermaid
flowchart TD
    classDef client    fill:#D6EAF8,stroke:#2E86C1,color:#1A5276,font-weight:bold
    classDef gateway   fill:#F0F0F0,stroke:#555555,color:#222222,font-weight:bold
    classDef userSvc   fill:#EBF5FB,stroke:#2E86C1,color:#1A5276
    classDef orderSvc  fill:#FDEBD0,stroke:#D35400,color:#7E5109
    classDef menuSvc   fill:#EAFAF1,stroke:#1E8449,color:#145A32
    classDef branchSvc fill:#F8F9FA,stroke:#566573,color:#2C3E50
    classDef kitSvc    fill:#FDEDEC,stroke:#CB4335,color:#78281F
    classDef notifSvc  fill:#FDEDEC,stroke:#922B21,color:#78281F
    classDef anSvc     fill:#E8DAEF,stroke:#6C3483,color:#4A235A
    classDef kafka     fill:#231F20,stroke:#000000,color:#ffffff,font-weight:bold
    classDef mysql     fill:#007DB8,stroke:#004f78,color:#ffffff,font-weight:bold
    classDef redis     fill:#DC382D,stroke:#A01E1E,color:#ffffff,font-weight:bold
    classDef s3        fill:#FF9900,stroke:#CC6600,color:#ffffff,font-weight:bold
    classDef rec       fill:#EAFAF1,stroke:#1E8449,color:#145A32,font-weight:bold
    classDef email     fill:#FAD7A0,stroke:#CA6F1E,color:#784212

    subgraph CLIENT["🖥️ Client Layer"]
        FE["Frontend\nReact + Vite + TypeScript\nfoodchain.live"]
        MOB["Mobile\nReact Native\nfoodchain-mobile"]
    end

    subgraph GATEWAY["⚙️ API Gateway Layer"]
        GW["API Gateway\nSpring Cloud Gateway · JWT Auth\napi.foodchain.live : 8080"]
        EUR["Eureka Server\nService Discovery & Registry\n:8761"]
        CFG["Config Server\nSpring Cloud Config\n:8888"]
    end

    subgraph SERVICES["🔧 Microservices Layer  —  Spring Boot 3 · Java 17 / 21 / 25"]
        US["User Service\nAuth · JWT · OAuth2\nPreferences v2 · :8086"]
        OS["Order Service\nOrders · Outbox Pattern\nKafka Producer · :8083"]
        MS["Menu Service\nItems · Categories · S3\nRecommendations v2 · :8082"]
        BS["Branch Service\nBranches · Hours\nLocation · :8081"]
        KS["Kitchen Service\nOrder Queue · SLA\nKafka Consumer · :8084"]
        NS["Notifications Service\nWebSocket · Email\nBrevo · :8087"]
        AS["Analytics Service\nReports · Metrics\nKafka Consumer · :8085"]
    end

    subgraph MESSAGING["📨 Messaging Layer"]
        KF[("Apache Kafka\nConfluentCloud")]
    end

    subgraph INFRA["🏗️ Infrastructure Layer"]
        subgraph RDS["☁️ AWS RDS MySQL 8"]
            DB_USER[("user_db")]
            DB_MENU[("menu_db")]
            DB_ORDER[("order_db")]
            DB_BRANCH[("branch_db")]
            DB_NOTIF[("notifications_db")]
            DB_AN[("analytics_report_db")]
        end
        REDIS[("Redis Cloud\nQueue · Cache · Sessions")]
        S3["AWS S3\nImage Storage"]
        REC["Recommendation Engine\nPreference + History Scoring"]
        EMAIL["Brevo / SMTP\nEmail Delivery"]
    end

    FE -->|"HTTPS / WSS"| GW
    MOB -->|"HTTPS"| GW
    GW <-->|"registers"| EUR
    GW <-->|"config pull"| CFG
    EUR -->|"lb:// routing"| US & OS & MS & BS & KS & NS & AS

    OS -->|"publishes events"| KF
    US -->|"publishes events"| KF
    KF -->|"order.received"| MS
    KF -->|"kitchen events"| KS
    KF -->|"notifications"| NS
    KF -->|"analytics events"| AS

    US -->|"persist"| DB_USER
    OS -->|"persist"| DB_ORDER
    MS -->|"persist"| DB_MENU
    BS -->|"persist"| DB_BRANCH
    NS -->|"persist"| DB_NOTIF
    AS -->|"persist"| DB_AN
    KS -->|"order queue"| REDIS
    US -->|"token blacklist"| REDIS
    MS -->|"image upload"| S3
    MS -->|"scores combos"| REC
    NS -->|"sends"| EMAIL

    class FE,MOB client
    class GW,EUR,CFG gateway
    class US userSvc
    class OS orderSvc
    class MS,REC menuSvc
    class BS branchSvc
    class KS kitSvc
    class NS notifSvc
    class AS anSvc
    class KF kafka
    class DB_USER,DB_MENU,DB_ORDER,DB_BRANCH,DB_NOTIF,DB_AN mysql
    class REDIS redis
    class S3 s3
    class EMAIL email
```

---

## Class Diagram

```mermaid
classDiagram
    classDef userSvc fill:#EBF5FB,stroke:#2E86C1,color:#1A5276
    classDef menuSvc fill:#EAFAF1,stroke:#1E8449,color:#145A32
    classDef orderSvc fill:#FDEBD0,stroke:#D35400,color:#7E5109
    classDef branchSvc fill:#F8F9FA,stroke:#566573,color:#2C3E50
    classDef kitSvc fill:#FDEDEC,stroke:#CB4335,color:#78281F
    classDef notifSvc fill:#FDEDEC,stroke:#922B21,color:#78281F
    classDef anSvc fill:#E8DAEF,stroke:#6C3483,color:#4A235A

    %% ── User Service : 8086 ──
    class User {
        +Long id
        +String email
        +String name
        +Role role
        +String googleId
    }
    class UserPreference {
        +Long id
        +Long userId
        +List cuisinePreferences
        +List allergens
        +boolean preferencesCompleted
    }
    class Role {
        <<enumeration>>
        CUSTOMER
        KITCHEN_STAFF
        BRANCH_MANAGER
        HEAD_OFFICE_ADMIN
    }

    %% ── Menu Service : 8082 ──
    class MenuItem {
        +Long id
        +String name
        +BigDecimal price
        +Long categoryId
        +boolean isActive
    }
    class Category {
        +Long id
        +String name
    }
    class UserMenuInteraction {
        +Long id
        +Long userId
        +Long menuItemId
        +int orderCount
        +LocalDateTime lastOrderedAt
    }

    %% ── Order Service : 8183 ──
    class Order {
        +Long id
        +Long userId
        +Long branchId
        +OrderStatus status
        +BigDecimal totalAmount
        +PaymentStatus paymentStatus
    }
    class OrderItem {
        +Long id
        +Long orderId
        +Long menuItemId
        +Integer quantity
        +BigDecimal unitPrice
    }
    class Payment {
        +Long id
        +Long orderId
        +String reference
        +String provider
        +PaymentStatus status
    }

    %% ── Branch Service : 8081 ──
    class Branch {
        +Long id
        +String name
        +String address
        +Double latitude
        +Double longitude
    }
    class OperatingHours {
        +Long id
        +Long branchId
        +DayOfWeek dayOfWeek
        +LocalTime openTime
        +LocalTime closeTime
    }
    class BranchTable {
        +Long id
        +Long branchId
        +Integer tableNumber
        +Integer capacity
        +TableStatus status
    }

    %% ── Kitchen Service : 8084 ──
    class KitchenOrder {
        +Long id
        +Long orderId
        +Long branchId
        +KitchenStatus status
        +LocalDateTime assignedAt
    }
    class KitchenOrderItem {
        +Long id
        +Long kitchenOrderId
        +String menuItemName
        +Integer quantity
    }

    %% ── Notifications Service : 8087 ──
    class Notification {
        +Long id
        +Long userId
        +String type
        +String message
        +boolean read
    }

    %% ── Analytics Service : 8085 ──
    class SalesReport {
        +Long id
        +Long branchId
        +LocalDate reportDate
        +BigDecimal totalRevenue
        +Integer totalOrders
    }
    class OrderItemAnalytics {
        +Long id
        +Long menuItemId
        +String menuItemName
        +Integer totalQuantity
    }

    User "1" --> "0..1" UserPreference : has
    User --> Role : assigned
    User "1" --> "0..*" Order : places
    Order "1" *-- "1..*" OrderItem : contains
    Order "1" --> "0..1" Payment : paid via
    Order "1" ..> "1" KitchenOrder : via Kafka
    Order "1" ..> "0..*" Notification : via Kafka
    MenuItem --> Category : in
    MenuItem "1" <-- "0..*" OrderItem : included as
    MenuItem "1" <-- "0..*" UserMenuInteraction : tracked by
    Branch "1" --> "0..*" MenuItem : offers
    Branch "1" --> "0..*" OperatingHours : schedules
    Branch "1" --> "0..*" BranchTable : has
    KitchenOrder "1" *-- "1..*" KitchenOrderItem : processes

  
```

---

## Entity Relationship Diagram

```mermaid
erDiagram
    users {
        char id PK
        varchar email
        varchar role
        char branch_id FK
        boolean is_active
    }
    user_preferences {
        char user_id PK "FK to users"
        json cuisine_preferences
        json dietary_restrictions
        varchar spice_level
        boolean preferences_completed
    }
    branches {
        char id PK
        varchar name
        char manager_id FK
        boolean is_active
        numeric rating
    }
    branch_hours {
        char id PK
        char branch_id FK
        int day_of_week
        time open_time
        time close_time
    }
    branch_tables {
        char id PK
        char branch_id FK
        int table_number
        int capacity
        varchar status
    }
    menu_categories {
        char id PK
        varchar name
        int display_order
        boolean is_active
    }
    menu_items {
        char id PK
        varchar name
        char category_id FK
        numeric base_price
        boolean is_active
    }
    user_menu_interactions {
        char id PK
        char user_id FK
        char menu_item_id FK
        char branch_id
        int order_count
        timestamp last_ordered_at
    }
    orders {
        char id PK
        char branch_id FK
        char customer_id FK
        varchar order_type
        varchar status
        numeric total_amount
        varchar payment_status
    }
    order_items {
        char id PK
        char order_id FK
        char menu_item_id FK
        int quantity
        numeric unit_price
        numeric subtotal
    }
    payments {
        char id PK
        char order_id FK
        char user_id FK
        numeric amount
        varchar provider
        varchar reference
        varchar status
    }
    order_status_updates {
        char id PK
        char order_id FK
        varchar old_status
        varchar new_status
        char updated_by FK
    }
    outbox_events {
        char id PK
        varchar topic
        varchar partition_key
        text payload
        boolean published
    }
    notification_logs {
        bigint id PK
        varchar notification_type
        varchar order_id FK
        varchar customer_id FK
        boolean is_read
        timestamp sent_at
    }
    order_analytics {
        bigint id PK
        char order_id FK
        char branch_id FK
        char customer_id FK
        numeric total_amount
    }
    order_item_analytics {
        bigint id PK
        char order_id FK
        char menu_item_id FK
        int quantity
        numeric line_total
    }
    branch_daily_summary {
        bigint id PK
        char branch_id FK
        date summary_date
        int total_orders
        numeric total_revenue
    }
    reports {
        bigint id PK
        varchar report_type
        char branch_id FK
        date start_date
        date end_date
    }

    users ||--o| user_preferences : "has"
    users ||--o{ orders : "places"
    users ||--o{ user_menu_interactions : "interacts"
    branches ||--|{ branch_hours : "has"
    branches ||--o{ branch_tables : "has"
    branches ||--o{ orders : "receives"
    menu_categories ||--o{ menu_items : "groups"
    menu_items ||--o{ order_items : "ordered as"
    menu_items ||--o{ user_menu_interactions : "tracked by"
    orders ||--|{ order_items : "contains"
    orders ||--o{ order_status_updates : "tracks"
    orders ||--o| payments : "paid via"
    orders ||--o{ notification_logs : "triggers"
    orders ||--o{ order_analytics : "feeds"
    branches ||--o{ branch_daily_summary : "summarised by"
    branches ||--o{ reports : "in"
    order_items ||--o{ order_item_analytics : "aggregated in"
```

---

## Key Improvements to Work On

- [ ] HTTPS termination for all internal services (gateway only for now)
- [ ] Circuit breakers with Resilience4j on critical inter-service calls
- [ ] Database schema migrations with Flyway
- [ ] Centralised structured logging (CloudWatch or ELK stack)
- [ ] Rate limiting at the API gateway level
- [ ] Expand test coverage — unit and integration tests per service
- [ ] Move from EC2 t2.micro to containerised orchestration (ECS or Kubernetes) for better resource management
- [ ] Multi-tenancy system — support multiple restaurant brands or franchise groups on one platform
