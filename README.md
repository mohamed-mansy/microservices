# Fibonacci Sequence

![Fibonacci Sequence](docs/screenshot-frontend.png)

**Short description**

This project is a microservices-based Fibonacci calculator deployed in a Kubernetes cluster. The system accepts an index from a React frontend, forwards it to an Express API, and calculates the Fibonacci value for that index in a distributed, fault-tolerant way.

---

## Table of contents

1. [Architecture](#architecture)
2. [Technologies](#technologies)
3. [Repository layout](#repository-layout)
4. [Local development](#local-development)
5. [Docker & Kubernetes](#docker--kubernetes)
6. [Environment variables](#environment-variables)
7. [API endpoints](#api-endpoints)
8. [Data flow](#data-flow)
9. [Troubleshooting](#troubleshooting)
10. [Testing](#testing)
11. [Contributing](#contributing)
12. [License](#license)

---

## Architecture

![Architecture Diagram](docs/architecture-diagram.png)

High-level components:

* **React (client)** — single-page app where user submits an index.
* **API (Express)** — receives index submissions and exposes API endpoints.

  * Writes received indices to **Postgres** (permanent history).
  * Pushes the index into **Redis** (as a job or key placeholder) for processing.
* **Worker** — a separate process that watches Redis for new indices, computes the Fibonacci value, and writes the result back into Redis (keyed by index).
* **Redis** — used as ephemeral store/cache and a simple task queue between the API and worker.
* **Postgres** — persistent store of all received indices (audit/history).
* **Kubernetes** — manifests for deploying client, server, worker, Redis, and Postgres.

---

## Technologies

* Frontend: React
* Backend: Node.js, Express
* Worker: Node.js
* Cache/Queue: Redis
* Database: PostgreSQL
* Containerization: Docker
* Orchestration: Kubernetes (manifests located in `/k8s`)
* CI/CD: (suggested) GitHub Actions / GitLab CI

---

## Repository layout

```text
microservices/
├─ client/                  # React app
├─ server/                  # Express API
│  ├─ worker/               # Worker that watches Redis and calculates fib
│  └─ src/
├─ k8s/                     # Kubernetes manifests (Deployments, Services, ConfigMaps, Secrets)
├─ docker-compose.yml       # Optional: compose for local dev
├─ README.md                # <- this file
└─ docs/
   ├─ architecture-diagram.png
   └─ screenshot-frontend.png
```

---

## Local development

### Prerequisites

* Docker & Docker Compose
* Node.js >= 18 (for local dev of client/server)
* kubectl (optional, for k8s testing)

### Quick start (using docker-compose)

2. From project root run (starts Postgres, Redis, server, worker, client):

```bash
docker compose up --build
```

3. Open the client in your browser (by default `http://localhost:3000`).

### Local (without docker)

Run Postgres and Redis locally (or use Docker for them), then:

```bash
# server
cd server
npm install
npm run dev

# worker (in a separate shell)
cd server/worker
npm install
npm run dev

# client
cd client
npm install
npm start
```

---

## Kubernetes

### Build images

```bash
# from repo root (example names)
docker build -t your-dockerhub-username/fibonacci-client:latest ./client
docker build -t your-dockerhub-username/fibonacci-server:latest ./server
docker build -t your-dockerhub-username/fibonacci-worker:latest ./server/worker
```

Push to your container registry and update the images in `k8s/` manifests.

### Deploy to Kubernetes (example)

```bash
kubectl apply -f k8s/
```

The `k8s/` folder should contain:

* `client-deployment.yaml`
* `client-cluster-ip-service.yaml`
* `server-deployment.yaml`
* `server-cluster-ip-service.yaml`
* `worker-deployment.yaml`
* `redis-deployment-.yaml`
* `redis-cluster-ip-service.yml`
* `postgres-deployment.yaml`
* `postgres-cluster-ip-service.yml`
* `database-persistent-volume-claim.yml`
* `ingress-service.yml`
 

Notes:

* Create your own secret object as imperative.
  
```bash
kubectl create secret generic <secret name> --form-literal <key=pass>/
```


---

## Environment variables

Example (server `.env`)

```env
PORT=5000
PG_HOST=postgres
PG_USER=postgres
PG_PASSWORD=postgrespassword
PG_DATABASE=fibdb
REDIS_HOST=redis
REDIS_PORT=6379
SECRET_KEY=some-secret
```

Example (client `.env`)

```env
REACT_APP_API_URL=http://localhost:5000
```

---
