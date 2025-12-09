
# DropMate — Cloud-Native Delivery Management

**Live App:** https://dropmate-frontend-ovkf3.ondigitalocean.app/

DropMate is a cloud-native delivery management system built as a set of small, focused microservices running on Kubernetes. It provides real-time driver tracking, WebSocket-based notifications, and a React dashboard for customers and drivers. The project demonstrates containerization, orchestration, observability, and production-ready deployment patterns suitable for small-to-medium delivery operations.

Quick highlights:
- 4 microservices (Core API, Location, Notification, Scaling Monitor)
- React frontend with Google Maps and Socket.IO for real-time updates
- PostgreSQL 16 with monthly partitioning for time-series events
- Redis for pub/sub and caching
- Kubernetes (DigitalOcean Kubernetes) with HPA, NGINX Ingress and cert-manager

---

## Contents
- Overview
- Features
- Architecture
- Tech stack
- Development (local) and Kubernetes deployment
- Demo credentials & video
- Team & contributions

---

## Overview

DropMate separates responsibilities into specialized services so each can scale and be operated independently:

- Core API (business logic, auth, shipment management)
- Location Service (high-frequency GPS ingestion, location history)
- Notification Service (WebSocket distribution via Socket.IO)
- Scaling Monitor (observability of HPA events and email alerts)

The system uses PostgreSQL as the single source of truth and Redis pub/sub to propagate events between services and connected clients.

---

## Key Features

- Real-time driver location (30s device updates, live UI markers)
- Redis pub/sub broadcasting to clients via Notification Service
- WebSocket-based notifications and room-based subscriptions
- Partitioned time-series storage for driver location and shipment events
- Google Maps & Directions integration for route visualization and ETA
- Role-based access control with Firebase Authentication (customer/driver/admin)

---

## Architecture (summary)

Clients (React frontend / mobile) ↔ NGINX Ingress (TLS) ↔ Microservices (Core API, Location, Notification, Scaling Monitor)

- PostgreSQL StatefulSet for persistent data
- Redis for pub/sub and fast caches
- HPA for Core, Location and Notification services

Example event flows:

- GPS update: Driver → Location Service → PostgreSQL + Redis → Notification Service → Frontend
- Delivery update: Driver → Core API → PostgreSQL → WebSocket → Frontend

---

## Technical Stack

- Frontend: React (Vite), Google Maps, Socket.IO client, Firebase
- Backend: Node.js, Express, Socket.IO, Firebase Admin SDK
- Data: PostgreSQL 16 (partitioning), Redis 7
- Infrastructure: Docker, Kubernetes (DOKS), NGINX Ingress, cert-manager
- DevOps: kubectl, doctl, DigitalOcean Container Registry

---

## Development Guide

Prerequisites: Node.js 18+, Docker, PostgreSQL 16, Redis 7, kubectl, doctl

Backend local (overview):
1. See `dropmate-backend` for service source and scripts.
2. Run SQL migrations in `dropmate-backend/schema/` (01-08).
3. Create `.env` with `DATABASE_URL`, `REDIS_URL`, Firebase credentials, etc.
4. Start each service with `npm run dev` in its folder.

Frontend local (overview):
1. See `dropmate-frontend` for the React app.
2. Install: `npm install` and set `.env.local` with API URLs, Google Maps key and Firebase config.
3. Start dev server: `npm run dev` (default port `5173`).

---

## Kubernetes Deployment (summary)

Full manifests and scripts are under `dropmate-backend/k8s/digitalocean/`.

Typical steps:

1. Create DOKS cluster and save kubeconfig: `doctl kubernetes cluster kubeconfig save <cluster-id>`
2. Create namespace and apply configmaps/secrets from `k8s/digitalocean/`.
3. Apply manifests in order: Postgres → Redis → Core API → Location → Notification → Ingress → Scaling Monitor.

Example apply sequence:

```powershell
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

Verify with: `kubectl get pods,svc,hpa -n dropmate`

---

## Demo credentials

Role | Email | Password
---|---:|---
Customer | admin@dropmate.com | test123
Driver | driverTest@dropmate.com | test123

---

## Video Demo

[Watch Demo Video](https://youtu.be/hpUit9Y83YA?si=r5JnPqBDB8Wenugw)

---

## Team & Contributions

| Name | Student Number | Email | Role |
|---|---:|---|---|
| Qiwen Lin | 1012495104 | qw.lin@mail.utoronto.ca | Backend & Architecture Lead |
| Liz Zhu | 1011844943 | liz.zhu@mail.utoronto.ca | Data & Infrastructure Engineer |
| Zongyan Yao | 1005200836 | zongyan.yao@mail.utoronto.ca | Frontend & UI/UX |
| Yuhao Cao | 1004145329 | davidyh.cao@mail.utoronto.ca | Observability & QA Engineer |

### Individual contributions (short)

- Qiwen Lin: Kubernetes architecture, Core API, NGINX ingress, PostgreSQL partitioning, Firebase Admin
- Liz Zhu: DB schema & migrations, indexing & partitioning, PV/StatefulSet testing, query tuning
- Zongyan Yao: React frontend, Google Maps & Directions integration, UI/UX, address autocomplete
- Yuhao Cao: Notification Service (Socket.IO), Scaling Monitor (HPA watcher + SendGrid), QA & load testing

*(Note: commits may appear under one user due to shared development environment; all members contributed equally.)*

---

## Where to look in the repo

- `dropmate-backend/` — backend services, Dockerfiles, k8s manifests and database schema
- `dropmate-frontend/` — React app, components, Firebase integration

---

If you want, I can:

- run a quick repo scan for TODOs and env samples
- create a condensed developer quickstart focused on local dev
- produce `ENV_TEMPLATE.md` with the environment variables required by each service

Which of these would you like next?

