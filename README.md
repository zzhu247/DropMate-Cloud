# DropMate – Cloud-Native Delivery Management System

**Live App:** https://dropmate-frontend-ovkf3.ondigitalocean.app/

DropMate is a cloud-native delivery management platform built with a microservices architecture on Kubernetes. It provides real-time GPS tracking, WebSocket-based notifications, and scalable infrastructure designed for small-to-medium delivery businesses.

**Quick Stats:**
- **4 Microservices**: Core API (8080), Location Service (8081), Notification Service (8082), Scaling Monitor (8083)
- **Frontend**: React 19 with Google Maps integration and real-time WebSocket updates
- **Data Layer**: PostgreSQL 16 with time-series partitioning + Redis 7 for pub/sub messaging
- **Infrastructure**: Kubernetes (DOKS) with horizontal pod autoscaling (2-10 replicas for core services)
- **Deployment**: HTTPS routing via NGINX Ingress + Let's Encrypt SSL certificates
- **Programming Languages**: Node.js (backend), React/JavaScript (frontend), SQL (database schemas)

---

# 1. Team Information

| Name        | Student Number | Email                       | Role                              |
|------------|----------------|-----------------------------|-----------------------------------|
| Qiwen Lin  | 1012495104     | qw.lin@mail.utoronto.ca     | Backend & Architecture Lead       |
| Liz Zhu    | 1011844943     | liz.zhu@mail.utoronto.ca    | Data & Infrastructure Engineer    |
| Zongyan Yao| 1005200836     | zongyan.yao@mail.utoronto.ca| Frontend & UI/UX                  |
| Yuhao Cao  | 1004145329     | davidyh.cao@mail.utoronto.ca| Observability & QA Engineer       |

---

# 2. Motivation, Problem & Objectives

### Problem
Local small businesses often rely on manual tools (spreadsheets, group chats, phone calls) for deliveries, resulting in:
- No real-time order visibility
- Slow driver coordination
- No operational data
- High dependency on expensive marketplaces

### Motivation
We aimed to build a self-hostable, low-cost, and cloud-native logistics system that gives businesses:
- Full ownership over their data
- A modern real-time tracking experience
- Scalable backend infrastructure
- Hands-on demonstration of Kubernetes, microservices, and DevOps pipelines

We chose a **microservices architecture** to demonstrate service separation, independent scaling, and cloud-native deployment patterns using Kubernetes. This approach allows each service to scale independently based on load, enables technology diversity, and provides fault isolation.

### Objectives
Demonstrate containerization, orchestration, and observability. Provide a working microservice system with:
- Shipment & driver management with real-time coordination
- High-frequency GPS location tracking (30-second intervals)
- Persistent PostgreSQL storage with partitioned time-series tables
- Kubernetes orchestration with horizontal pod autoscaling
- React dashboard for customers and drivers with live maps

---

# 3. Features

## 3.1 Real-Time Capabilities

**Live Driver Tracking**
- GPS location updates every 30 seconds from driver devices with automatic geolocation
- 10-second cooldown on manual location refreshes to prevent API abuse
- Redis pub/sub broadcasting to all connected clients for instant updates
- Location history with queryable timestamps stored in partitioned PostgreSQL tables
- Accuracy metrics tracked per location update

**Instant Notifications**
- Socket.IO WebSocket server (port 8082) for persistent client connections
- Room-based subscriptions allowing clients to join specific channels (`driver:123`, `shipment:456`)
- Pattern-based Redis subscriptions (`driver:*`, `shipment:*`) for event routing
- Push notifications to mobile devices via Expo Server SDK for offline users
- Multiple notification types: shipment status changes, package updates, driver proximity alerts

**Live Maps & Routes**
- Google Maps API integration with custom markers (pickup in green, delivery in red, driver in blue)
- Route visualization using Google Directions API showing turn-by-turn navigation
- Real-time driver marker movement as GPS updates arrive
- Address autocomplete with automatic coordinate extraction using Google Places API
- Distance and ETA calculations between driver and delivery locations

**WebSocket Events**
- `driver_location_updated`: Broadcast when driver GPS coordinates change
- `shipment_status_updated`: Emitted on status transitions (pending → assigned → in_transit → delivered)
- `package_status_updated`: Granular package delivery milestones (out_for_delivery, in_transit, delivered)

## 3.2 Microservices Architecture

**Service Separation**

Our platform consists of four specialized microservices, each handling distinct responsibilities:

- **Core API (Port 8080)**: Business logic, user authentication, shipment CRUD operations, driver management, order processing, and push notification orchestration
- **Location Service (Port 8081)**: High-frequency GPS location ingestion, geospatial queries, location history retrieval, and Redis pub/sub broadcasting
- **Notification Service (Port 8082)**: WebSocket server for real-time client connections, Redis subscriber with pattern matching, and room-based event broadcasting
- **Scaling Monitor (Port 8083)**: Kubernetes HPA event monitoring, metrics collection (CPU/memory utilization), and SendGrid email alerts for scaling events

**Independent Scaling**

Each service scales independently based on its workload characteristics:

- **Core API**: 2-10 replicas (HPA based on 90% CPU and 90% memory thresholds)
- **Location Service**: 1-5 replicas (handles burst GPS traffic during peak delivery hours)
- **Notification Service**: 1-3 replicas (WebSocket connection pooling across instances)
- **Scaling Monitor**: Fixed 1 replica (lightweight administrative service)

**Service Communication**

- **Synchronous**: REST APIs for request-response operations (e.g., create shipment, fetch driver details)
- **Asynchronous**: Redis pub/sub for event-driven updates (e.g., location broadcasts, status changes)
- **Shared State**: PostgreSQL as the single source of truth for persistent data
- **Routing**: NGINX Ingress with domain-based routing:
  - `api.dropmate.ca` → Core API
  - `location.dropmate.ca` → Location Service
  - `notify.dropmate.ca` → Notification Service

**Container Orchestration**

- Docker containers for consistent environments across development and production
- Kubernetes Deployments with rolling updates for zero-downtime deployments
- Health checks (liveness and readiness probes) on all services
- Resource limits and requests to prevent resource starvation
- ConfigMaps for environment-specific configuration
- Secrets for sensitive credentials (database passwords, API keys, Firebase credentials)

## 3.3 User Features

**Customer Dashboard** (React-based 3-panel interface)

- **Package List Panel**: Displays all tracked shipments with tracking numbers and current status
- **Package Details Panel**: Shows comprehensive shipment information including:
  - Tracking number and current status badges
  - Sender and receiver details (name, phone, address)
  - Package specifications (weight, dimensions, description, fragile flag)
  - Status timeline with event history and timestamps
- **Live Map Panel**: Real-time Google Maps view with:
  - Driver location marker (updated every 10 seconds)
  - Route visualization from pickup to delivery
  - Custom markers for pickup (green) and delivery (red) locations
- **Package Creation**: Modal-based workflow with address autocomplete, coordinate extraction, and package detail input
- **Package Tracking**: Enter tracking number to add existing packages to dashboard

**Driver Dashboard** (2-tab interface)

- **My Deliveries Tab**:
  - List of assigned deliveries with customer information and addresses
  - Package tracking numbers and status indicators
  - Action buttons to start delivery and mark as delivered
  - Real-time status updates via WebSocket
- **Available Packages Tab**:
  - Grid display of unassigned packages available for pickup
  - Customer contact information and delivery addresses
  - Package value and compensation details
  - Claim button to accept delivery assignments
- **Automatic GPS Tracking**:
  - 30-second interval location updates when delivery is "in_transit"
  - Background geolocation using browser Geolocation API
  - Automatic tracking stop on delivery completion
- **Route Planning**:
  - Google Maps integration showing route from current location to delivery
  - Turn-by-turn directions via Google Directions API
  - Distance and estimated time of arrival calculations

**Authentication & Access Control**

- **Firebase Authentication**: Email/password authentication with secure token management
- **Role-Based Access**: Three roles (customer, driver, admin) with endpoint-level authorization
- **Driver Registration**: Workflow to create driver profile with vehicle type and license information
- **Protected Routes**: Frontend route guards ensuring only authenticated users access dashboards
- **Token Refresh**: Automatic Firebase ID token refresh for seamless session management

## 3.4 Data Architecture

**PostgreSQL 16 Schema**

- **Core Tables**:
  - `users`: Authentication records (email, password_hash, role, firebase_uid)
  - `customers`: Customer profiles linked to user accounts
  - `drivers`: Driver profiles with vehicle_type, license_number, and status (offline/available/on_delivery)
  - `orders`: Order records with total_amount and status tracking
  - `shipments`: Package details with tracking_number, pickup/delivery addresses, geospatial coordinates (DECIMAL 10,8), package_details (JSONB), and soft delete support
  - `messages`: Customer-driver communication records
  - `push_tokens`: Expo push notification device tokens per user

- **Time-Series Partitioned Tables**:
  - `driver_location_events`: GPS coordinates partitioned by occurred_at (monthly ranges), indexed on (driver_id, occurred_at DESC) for fast latest-location queries
  - `shipment_events`: Status change audit log with event_type, from_status, to_status, and metadata (JSONB), partitioned for efficient event timeline retrieval
  - `webhook_events`, `ad_impressions`: Reserved for future integrations and analytics

**Geospatial Indexing**

- Latitude/longitude stored as DECIMAL(10, 8) for ~1.1mm precision
- Indexed columns for proximity searches (idx_shipments_pickup_location, idx_shipments_delivery_location)
- Used for geospatial correlation (finding nearby drivers for shipments)

**Redis 7 Caching & Pub/Sub**

- **Location Channels**: `driver:{driverId}:location` for GPS broadcasts
- **Shipment Channels**: `shipment:{shipmentId}:location` for driver-to-shipment correlation
- **Pattern Subscriptions**: `driver:*` and `shipment:*` patterns in Notification Service
- **TTL-Based Caching**: Frequently accessed data cached with automatic expiration
- **Connection Pooling**: Redis client connection reuse across requests

**Performance Optimizations**

- **Table Partitioning**: Monthly partitions on high-volume event tables for query performance
- **Strategic Indexing**: Indexes on tracking_number, driver_id, occurred_at for fast lookups
- **Connection Pooling**: PostgreSQL client (pg library) connection pool to reduce overhead
- **Horizontal Scaling**: HPA scales service replicas based on CPU/memory load
- **Soft Deletes**: Preserve data integrity while allowing users to "delete" shipments

---

# 4. User Guide

## Demo Login Credentials

| Role | Email | Password |
|------|--------|-----------|
| Customer | admin@dropmate.com | test123 |
| Driver   | driverTest@dropmate.com | test123 |

## Video Demo

[Watch Demo Video](https://youtu.be/hpUit9Y83YA?si=r5JnPqBDB8Wenugw)

## Main User Flows

### Customer Flow
1. Log in → View 3-panel dashboard (package list, details, live map)
2. Track existing package by entering tracking number, or create new shipment with address autocomplete
3. Watch real-time driver location and status updates on map

### Driver Flow
1. Log in → View assigned deliveries or claim available packages
2. Start delivery → Automatic GPS tracking (30-second intervals)
3. Mark as delivered → Customer receives instant notification

---

# 5. System Architecture

## 5.1 Architecture Diagram

```
┌──────────────────────────────────────────────┐
│          Internet / Users                     │
└──────────────────┬───────────────────────────┘
                   │
         ┌─────────┴─────────┐
         ▼                   ▼
   ┌──────────┐       ┌──────────┐
   │ Frontend │       │  Mobile  │
   │ (React)  │       │  Clients │
   └──────────┘       └──────────┘
         │                   │
         └─────────┬─────────┘
                   ▼
        ┌────────────────────┐
        │ NGINX Ingress      │
        │ (Load Balancer)    │
        │ TLS + Routing      │
        └────────────────────┘
                   │
     ┌─────────────┼─────────────┐
     │             │             │
     ▼             ▼             ▼
┌─────────┐  ┌─────────┐  ┌─────────┐
│Core-API │  │Location │  │ Notify  │
│Service  │  │Service  │  │ Service │
│:8080    │  │:8081    │  │:8082    │
└─────────┘  └─────────┘  └─────────┘
     │             │             │
     ▼             ▼             ▼
┌─────────────────────────────────────────┐
│     Kubernetes Pod Layer (DOKS)         │
│                                         │
│  ┌──────────┐   ┌──────────┐            │
│  │Core-API  │   │Core-API  │            │
│  │Pod 1     │   │Pod 2     │            │
│  │HPA: 2-10 │   │HPA: 2-10 │            │
│  └──────────┘   └──────────┘            │
│                                         │
│  ┌──────────┐   ┌──────────┐            │
│  │Location  │   │ Notify   │            │
│  │Service   │   │ Service  │            │
│  │HPA: 1-5  │   │HPA: 1-3  │            │
│  └──────────┘   └──────────┘            │
│                                         │
│  ┌──────────┐   ┌──────────┐            │
│  │ Redis    │   │Scaling   │            │
│  │Pub/Sub   │   │Monitor   │            │
│  └──────────┘   │(Fixed 1) │            │
│                 └──────────┘            │
└─────────────────────────────────────────┘
                   │
     ┌─────────────┼─────────────┐
     ▼             ▼             ▼
┌──────────┐  ┌──────────┐  ┌──────────┐
│PostgreSQL│  │ SendGrid │  │Kubernetes│
│StatefulSet│ │  Email   │  │   API    │
│(10Gi PV) │  └──────────┘  └──────────┘
└──────────┘
```

**Data Flow Examples:**

**[1] GPS Tracking:** Driver → Location Service → PostgreSQL + Redis → Notification Service → Customer Browser

**[2] Delivery Complete:** Driver → Core API → PostgreSQL → WebSocket → Customer real-time update

**[3] Auto-Scaling:** Traffic surge → HPA scales pods → Scaling Monitor → SendGrid email alert

## 5.2 Microservices Overview

### Core API (Port 8080)

**Responsibilities:**
- User authentication and authorization via Firebase Admin SDK
- Shipment CRUD operations (create, read, update, delete with soft delete support)
- Driver management (list, retrieve, update status)
- Order processing and customer profile management
- Push notification orchestration via Expo Server SDK
- WebSocket event broadcasting for shipment status changes
- Customer-driver messaging

**Key Technologies:**
- Express 4.19.2 (REST API framework)
- Socket.IO 4.7.5 (WebSocket server)
- Firebase Admin SDK 13.6.0 (server-side authentication)
- Expo Server SDK 3.7.0 (push notifications to mobile devices)
- pg 8.11.3 (PostgreSQL client with connection pooling)
- bcrypt 5.1.1 (password hashing)
- jsonwebtoken 9.0.2 (JWT token signing/verification)

**Key API Endpoints:**
- `GET /api/shipments` - List all shipments
- `GET /api/shipments/:id` - Get shipment details with status history
- `GET /api/shipments/track/:trackingNumber` - Public tracking by tracking number
- `GET /api/shipments/:id/location` - Get shipment with live driver location
- `POST /api/shipments/:id/assign-driver` - Assign driver to shipment
- `PATCH /api/shipments/:id/status` - Update shipment status (assigned/in_transit/delivered)
- `PATCH /api/shipments/:id/package-status` - Update granular package status
- `GET /api/shipments/:id/events` - Retrieve status change timeline
- `GET /api/users/me` - Get current user profile
- `PATCH /api/users/me` - Update user profile
- `GET /api/users/me/shipments` - List user's shipments
- `POST /api/users/me/shipments` - Create new shipment
- `DELETE /api/users/me/shipments/:id` - Soft delete shipment (pending/delivered only)
- `GET /api/drivers` - List all drivers
- `GET /api/drivers/:id` - Get driver details
- `POST /api/messages` - Send customer-driver message

**Scaling Configuration:**
- Horizontal Pod Autoscaler: 2-10 replicas
- Scale triggers: CPU utilization > 90% OR Memory utilization > 90%
- Resource requests: 200m CPU, 256Mi memory
- Resource limits: 500m CPU, 512Mi memory

### Location Service (Port 8081)

**Responsibilities:**
- High-frequency GPS location ingestion from driver devices
- Location history storage and retrieval with time-range filtering
- Geospatial correlation (finding which shipments a driver is delivering)
- Redis pub/sub broadcasting for real-time location updates
- Partitioned table writes for efficient time-series data storage

**Key Technologies:**
- Express 4.19.2 (REST API framework)
- Redis 4.6.13 (pub/sub client for real-time broadcasting)
- pg 8.11.3 (PostgreSQL client)

**Key API Endpoints:**
- `POST /api/location/:driverId` - Submit GPS coordinates (latitude, longitude, accuracy, timestamp)
- `GET /api/location/:driverId/latest` - Get most recent driver location
- `GET /api/location/:driverId/history` - Query location history with time range
- `GET /api/location/shipment/:shipmentId` - Get driver location for a specific shipment

**Data Flow:**
1. Driver device submits location → POST /api/location/:driverId with {latitude, longitude, accuracy}
2. Insert into PostgreSQL `driver_location_events` table (partitioned by occurred_at)
3. Publish to Redis channel `driver:{driverId}:location`
4. Query active shipments for this driver
5. Publish to Redis channels `shipment:{shipmentId}:location` for each shipment
6. Notification Service subscribers receive updates

**Scaling Configuration:**
- Horizontal Pod Autoscaler: 1-5 replicas (handles burst traffic during peak hours)
- Optimized for high write throughput with partitioned tables

### Notification Service (Port 8082)

**Responsibilities:**
- WebSocket server for persistent client connections
- Redis subscriber with pattern matching (`driver:*`, `shipment:*`)
- Room-based broadcasting (clients join specific driver/shipment rooms)
- Event distribution to subscribed clients
- Connection state management (connect, disconnect, room join/leave)

**Key Technologies:**
- Socket.IO 4.7.5 (WebSocket server)
- Redis 4.6.13 (subscriber client)
- Express 4.19.2 (HTTP server for health checks)

**WebSocket Events (Client → Server):**
- `subscribe:driver` - Join driver location room
- `subscribe:shipment` - Join shipment status room
- `unsubscribe:driver` - Leave driver room
- `unsubscribe:shipment` - Leave shipment room

**WebSocket Events (Server → Client):**
- `driver_location_updated` - Emitted when GPS coordinates change
- `shipment_location_updated` - Emitted when shipment's driver location changes
- `shipment_status_updated` - Emitted on status transitions
- `package_status_updated` - Emitted on package milestone changes
- `connected` - Connection acknowledgment with client ID

**Redis Pattern Subscriptions:**
- `driver:*:location` → Broadcasts to clients in corresponding driver rooms
- `shipment:*:location` → Broadcasts to clients in corresponding shipment rooms

**Scaling Configuration:**
- Horizontal Pod Autoscaler: 1-3 replicas
- WebSocket connections distributed across replicas via NGINX Ingress sticky sessions

### Scaling Monitor Service (Port 8083)

**Responsibilities:**
- Watch Kubernetes HPA (Horizontal Pod Autoscaler) events for all services
- Collect metrics from Kubernetes API (CPU usage, memory usage, pod counts)
- Send email alerts via SendGrid when scaling occurs
- Deduplication of alerts (5-minute window to prevent spam)
- Provide health/stats endpoint for monitoring

**Key Technologies:**
- @kubernetes/client-node 0.20.0 (Kubernetes API client)
- SendGrid 7.7.0 (email service)
- Express 4.19.2 (HTTP server)

**Monitored Events:**
- Pod scaling up (e.g., Core API 2→5 replicas)
- Pod scaling down (e.g., Core API 5→2 replicas)
- HPA metric threshold breaches
- Deployment rollouts and updates

**RBAC Requirements:**
- ServiceAccount with permissions to get/list/watch HPAs and Pods
- ClusterRole binding for read-only access to autoscaling resources

**Scaling:** Fixed 1 replica

---

# 6. Technical Stack

## Frontend Layer
- **React 19.2.0** - UI library with concurrent rendering and automatic batching
- **Vite (Rolldown 7.2.2)** - Build tool and dev server for fast hot module replacement
- **React Router DOM 7.9.6** - Client-side routing with nested routes and loaders
- **Socket.IO Client 4.8.1** - WebSocket client for real-time server communication
- **@react-google-maps/api 2.20.7** - React wrapper for Google Maps JavaScript API
- **Firebase 12.6.0** - Client-side authentication SDK (onAuthStateChanged, signInWithEmailAndPassword)

## Backend Layer
- **Node.js** - JavaScript runtime with ESM module support
- **Express 4.19.2** - Minimalist web framework for REST APIs (all 4 services)
- **Socket.IO 4.7.5** - WebSocket library for bi-directional real-time communication (Core API, Notification Service)
- **Firebase Admin SDK 13.6.0** - Server-side Firebase authentication (token verification, user management)
- **Expo Server SDK 3.7.0** - Push notification delivery to iOS/Android devices
- **SendGrid 7.7.0** - Transactional email service for scaling alerts
- **@kubernetes/client-node 0.20.0** - Kubernetes API client for HPA monitoring
- **bcrypt 5.1.1** - Password hashing with salt rounds
- **jsonwebtoken 9.0.2** - JWT token generation and verification
- **cors 2.8.5** - Cross-origin resource sharing middleware
- **dotenv 16.3.1** - Environment variable management

## Data Layer
- **PostgreSQL 16** - Relational database with ACID guarantees
  - **pg 8.11.3** - Node.js PostgreSQL client with connection pooling
  - **StatefulSet** - Kubernetes workload with 10Gi persistent volume
  - **Table Partitioning** - Monthly partitions on driver_location_events and shipment_events for query performance
  - **Geospatial Indexing** - DECIMAL(10, 8) precision for latitude/longitude with B-tree indexes
  - **JSONB Support** - Flexible schema for package_details and event metadata
- **Redis 7** - In-memory data store for caching and messaging
  - **redis 4.6.13** - Node.js Redis client with pub/sub support
  - **Pattern-Based Subscriptions** - `driver:*`, `shipment:*` for event routing
  - **TTL Expiration** - Automatic key expiration for cache invalidation and deduplication

## Infrastructure Layer
- **DigitalOcean Kubernetes (DOKS)** - Managed Kubernetes cluster (NYC1 region, 2x s-2vcpu-4gb nodes)
- **NGINX Ingress Controller** - Layer 7 load balancer with HTTP/HTTPS routing
- **cert-manager** - Automatic SSL certificate provisioning and renewal via Let's Encrypt
- **Horizontal Pod Autoscaler (HPA)** - Automatic scaling based on CPU/memory metrics
- **Kubernetes Secrets** - Encrypted storage for sensitive credentials (postgres-secret, app-secret, firebase-secret, expo-secret, sendgrid-secret)
- **Kubernetes ConfigMaps** - Environment-specific configuration (NODE_ENV, CORS_ORIGIN)
- **Docker** - Container runtime for consistent environments
- **DigitalOcean Container Registry** - Private Docker image registry

## DevOps & Tools
- **kubectl** - Kubernetes CLI for cluster management
- **doctl** - DigitalOcean CLI for infrastructure provisioning
- **Git** - Version control with GitHub remote repositories
- **Nodemon 3.0.3** - Development auto-restart on file changes
- **ESLint** - JavaScript/React linting for code quality
- **cloc** - Lines of code counter for project metrics

## External Services & APIs
- **Google Maps API** - Maps display, geocoding, and directions
- **Google Places API** - Address autocomplete with coordinate extraction
- **Firebase Authentication** - User identity and access management
- **SendGrid** - Transactional email delivery (scaling alerts)
- **Expo Push Notifications** - Mobile push notification infrastructure

---

# 7. Database Schema

## Core Tables

### Authentication & Users
- **users** - Authentication records
  - Columns: `id` (PK), `email` (UNIQUE), `password_hash`, `role` (customer/driver/admin), `firebase_uid`, `created_at`, `updated_at`
  - Indexes: `idx_users_email`, `idx_users_firebase_uid`

- **customers** - Customer profiles linked to users
  - Columns: `id` (PK), `user_id` (FK → users.id), `name`, `phone`, `created_at`, `updated_at`
  - Foreign Key: `user_id` with CASCADE delete

- **drivers** - Driver profiles with vehicle information
  - Columns: `id` (PK), `user_id` (FK → users.id), `name`, `vehicle_type` (Bike/Scooter/Car/Van/Truck), `license_number`, `status` (offline/available/on_delivery), `created_at`, `updated_at`
  - Indexes: `idx_drivers_status` (for finding available drivers)

### Orders & Shipments
- **orders** - Order records
  - Columns: `id` (PK), `customer_id` (FK → customers.id), `total_amount` (DECIMAL 10,2), `status` (pending/confirmed/shipped/delivered), `created_at`, `updated_at`

- **shipments** - Package details and tracking
  - Columns:
    - `id` (PK), `order_id` (FK → orders.id), `driver_id` (FK → drivers.id, NULLABLE)
    - `tracking_number` (VARCHAR 50, UNIQUE) - Publicly shareable tracking ID
    - `status` (VARCHAR 50) - Shipment lifecycle (pending/assigned/in_transit/delivered)
    - `package_status` (VARCHAR 50) - Granular delivery status (out_for_delivery/in_transit/delivered/exception)
    - Pickup location: `pickup_address` (TEXT), `pickup_latitude` (DECIMAL 10,8), `pickup_longitude` (DECIMAL 11,8)
    - Delivery location: `delivery_address` (TEXT), `delivery_latitude` (DECIMAL 10,8), `delivery_longitude` (DECIMAL 11,8)
    - Sender: `sender_name` (VARCHAR 100), `sender_phone` (VARCHAR 20)
    - Receiver: `receiver_name` (VARCHAR 100), `receiver_phone` (VARCHAR 20)
    - Package: `package_weight` (DECIMAL 8,2), `package_description` (TEXT), `package_details` (JSONB) - Flexible metadata
    - Soft delete: `deleted_at` (TIMESTAMP NULLABLE), `deleted_by_user_id` (FK → users.id, NULLABLE)
    - Timestamps: `created_at`, `updated_at`
  - Indexes:
    - `idx_shipments_tracking_number` (UNIQUE) - Fast public tracking lookup
    - `idx_shipments_driver_id` - Driver assignment queries
    - `idx_shipments_pickup_location` - Geospatial proximity searches
    - `idx_shipments_delivery_location` - Geospatial proximity searches
    - `idx_shipments_deleted_at WHERE deleted_at IS NULL` (partial index) - Exclude soft-deleted records

### Communication
- **messages** - Customer-driver messaging
  - Columns: `id` (PK), `sender_id` (FK → users.id), `recipient_id` (FK → users.id), `shipment_id` (FK → shipments.id), `message` (TEXT), `read` (BOOLEAN DEFAULT false), `created_at`
  - Indexes: `idx_messages_shipment_id`, `idx_messages_recipient_read` (for unread count)

### Push Notifications
- **push_tokens** - Expo push notification device tokens
  - Columns: `id` (PK), `user_id` (FK → users.id), `token` (VARCHAR 255, UNIQUE), `device_type` (VARCHAR 50), `created_at`, `updated_at`
  - Indexes: `idx_push_tokens_user_id` (users can have multiple devices)

## Partitioned Tables (Time-Series Data)

**Purpose:** Handle high-volume event data with efficient querying and automatic archival.

### driver_location_events (Partitioned by occurred_at)
- **Partitioning Strategy:** Range partitioning by month (e.g., events_2024_01, events_2024_02)
- **Columns:**
  - `id` (BIGSERIAL PK), `driver_id` (FK → drivers.id), `latitude` (DECIMAL 10,8), `longitude` (DECIMAL 11,8), `accuracy` (DECIMAL 6,2), `occurred_at` (TIMESTAMP NOT NULL)
- **Indexes:** `idx_driver_location_driver_time` on `(driver_id, occurred_at DESC)` - Fast latest-location queries
- **Use Case:** GPS tracking history, location timeline for disputes
- **Retention Policy:** Can archive/drop old partitions (e.g., keep 6 months)

### shipment_events (Partitioned by occurred_at)
- **Partitioning Strategy:** Range partitioning by month
- **Columns:**
  - `id` (BIGSERIAL PK), `shipment_id` (FK → shipments.id), `event_type` (VARCHAR 50), `from_status` (VARCHAR 50), `to_status` (VARCHAR 50), `metadata` (JSONB), `latitude` (DECIMAL 10,8), `longitude` (DECIMAL 11,8), `occurred_at` (TIMESTAMP NOT NULL)
- **Indexes:** `idx_shipment_events_shipment_time` on `(shipment_id, occurred_at DESC)` - Event timeline queries
- **Event Types:** assigned, picked_up, in_transit, delivered, exception
- **Use Case:** Audit log, status change history for customer timeline

### webhook_events, ad_impressions
- **Purpose:** Reserved for future integrations (webhook logs) and analytics (ad performance tracking)
- **Partitioning:** Same monthly strategy as above

## Indexing Strategy

**Performance-Critical Indexes:**
- `idx_shipments_tracking_number` (UNIQUE) - Public tracking endpoint (`GET /shipments/track/:trackingNumber`)
- `idx_shipments_driver_id` - Driver dashboard queries (`GET /drivers/:id/shipments`)
- `idx_shipments_pickup_location`, `idx_shipments_delivery_location` - Geospatial proximity searches (finding nearby drivers)
- `idx_driver_location_driver_time` - Latest driver location retrieval with `LIMIT 1`
- `idx_shipment_events_shipment_time` - Status timeline rendering in UI

**Partial Indexes:**
- `idx_shipments_deleted_at WHERE deleted_at IS NULL` - Exclude soft-deleted shipments from active queries

## Geospatial Precision

- **Latitude:** DECIMAL(10, 8) - Range: -90.00000000 to 90.00000000 (~1.1mm precision)
- **Longitude:** DECIMAL(11, 8) - Range: -180.00000000 to 180.00000000 (~1.1mm precision at equator)
- **Use Case:** Accurate driver-to-customer distance calculations, proximity-based driver assignment

## Soft Delete Implementation

- **shipments.deleted_at** - Timestamp when user deleted shipment (NULL for active)
- **shipments.deleted_by_user_id** - User who performed deletion (for audit)
- **Constraints:** Only pending or delivered shipments can be soft-deleted (cannot delete in-transit packages)
- **Benefits:** Data recovery, audit trail, referential integrity preservation

---

# 8. Development Guide

## Prerequisites
Node.js 18+, Docker, PostgreSQL 16, Redis 7, kubectl, doctl, Git

## Backend Local Setup
Full setup instructions: [dropmate-backend repository](https://github.com/kevinlin29/dropmate-backend)

1. Clone repository and install dependencies for all 4 services
2. Run PostgreSQL migration files (schema/01-08)
3. Configure `.env` files with DATABASE_URL, REDIS_URL, Firebase credentials
4. Start services: `npm run dev` in each service directory

**Note:** Credentials sent to TA.

## Frontend Local Setup
Full setup instructions: [dropmate-frontend repository](https://github.com/kevinlin29/dropmate-frontend)

1. Clone, install: `npm install`
2. Configure `.env.local` with API URLs, Google Maps API key, Firebase config
3. Start: `npm run dev` (http://localhost:5173)

## Kubernetes Deployment
Full instructions: [dropmate-backend/k8s/digitalocean/](https://github.com/kevinlin29/dropmate-backend/tree/main/k8s/digitalocean)

1. Create DOKS cluster (2x s-2vcpu-4gb nodes)
2. Configure kubectl: `doctl kubernetes cluster kubeconfig save <cluster-id>`
3. Create namespace: `kubectl apply -f k8s/digitalocean/00-namespace.yaml`
4. Set up secrets: `kubectl create secret generic postgres-secret/app-secret/firebase-secret/expo-secret/sendgrid-secret -n dropmate`
5. Deploy services:
   ```bash
   kubectl apply -f k8s/digitalocean/02-configmaps.yaml
   kubectl apply -f k8s/digitalocean/03-postgres.yaml
   kubectl apply -f k8s/digitalocean/04-redis.yaml
   kubectl apply -f k8s/digitalocean/05-core-api.yaml
   kubectl apply -f k8s/digitalocean/06-location-service.yaml
   kubectl apply -f k8s/digitalocean/07-notification-service.yaml
   kubectl apply -f k8s/digitalocean/08-ingress.yaml
   kubectl apply -f k8s/digitalocean/10-scaling-monitor-rbac.yaml
   kubectl apply -f k8s/digitalocean/11-scaling-monitor-service.yaml
   ```
6. Verify: `kubectl get pods/services/ingress/hpa -n dropmate`
7. Access: Configure DNS A records → https://api.dropmate.ca

---

# 9. Individual Contributions

### Note for Grading

Our team collaborated using a single Linux machine for development.
Because of this setup, most commits appear under one user account.
However, all team members contributed equally to the project, and we collectively acknowledge and agree on this representation.

### Qiwen Lin
- Kubernetes cluster setup and microservice architecture design
- Core API development (shipments, users, drivers, authentication)
- NGINX Ingress configuration with TLS termination via cert-manager
- PostgreSQL schema design and partitioning strategy
- Firebase Admin SDK integration for server-side authentication

### Liz Zhu
- PostgreSQL database schema design and migration scripts
- Backend database queries optimization with indexes and partitioning
- Persistent storage testing with StatefulSets and persistent volumes
- Data modeling for shipments, events, and geospatial coordinates
- Connection pooling and query performance tuning

### Zongyan Yao
- React 19 frontend development with Vite build tooling
- Google Maps integration with live driver tracking and route visualization
- Frontend–backend connectivity via REST APIs and Socket.IO WebSocket
- Address autocomplete component using Google Places API
- Customer and driver dashboard UI/UX design and implementation

### Yuhao Cao
- Notification service development with Socket.IO WebSocket server
- Scaling monitor service with Kubernetes HPA event watching
- SendGrid integration for email alerting on scaling events
- QA testing and load testing for horizontal pod autoscaling validation
- Redis pub/sub pattern implementation for real-time event broadcasting

---

# 10. Conclusion & Lessons Learned

## Key Insights

- **Kubernetes is powerful but requires discipline in configuration:** Proper resource requests/limits, health checks, and RBAC policies are essential for stable production deployments
- **Clear microservice boundaries simplify scaling and debugging:** Separating concerns (Core API, Location, Notification) allowed independent scaling and easier troubleshooting
- **Observability tooling dramatically reduced time spent diagnosing issues:** Kubernetes logs, metrics, and HPA monitoring via Scaling Monitor Service enabled proactive problem detection
- **Frontend–backend coordination is crucial in real-time systems:** WebSocket connection management, room-based subscriptions, and event naming conventions required careful planning

## Scalability Results

Our Kubernetes HPA configuration successfully handles traffic spikes by scaling Core API from 2 to 10 replicas based on CPU/memory thresholds (90%). During peak testing, the Location Service processed over 1000 GPS updates per minute while maintaining sub-100ms response times through partitioned tables and Redis pub/sub architecture. The Notification Service distributed WebSocket connections across 3 replicas, supporting 500+ concurrent clients with minimal latency.

## Final Remarks

DropMate evolved from a class project into a realistic cloud-native platform demonstrating:

- **Containerization** with Docker and multi-stage builds for optimized images
- **Microservice architecture** with 4 specialized services (Core API, Location, Notification, Scaling Monitor)
- **Stateful and stateless workloads** (PostgreSQL StatefulSet vs. Deployment for services)
- **Real-time communication** via Socket.IO WebSockets and Redis pub/sub messaging
- **Horizontal scaling** with Kubernetes HPA responding to CPU/memory metrics
- **Secure TLS-enabled deployment** with automatic certificate renewal via cert-manager

This project provided hands-on experience with designing, deploying, and operating a real distributed system in production-like conditions. We gained deep understanding of Kubernetes orchestration, microservice design patterns, real-time event-driven architectures, and cloud infrastructure management.

---
