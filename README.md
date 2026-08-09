# Full-Stack AI DevOps

A polyglot microservices project — small backend services each implemented three times in **Node.js (Express)**, **Python (Flask)**, and **Java (Spring Boot)** — used to apply and demonstrate the full DevOps lifecycle: containerization, service-to-service communication, orchestration, and CI/CD.

## DevOps Skills Demonstrated

- **Docker** — image builds, registries, container lifecycle, tagging/versioning, port mapping, multi-stack containerization (Python, Node.js, Spring Boot)
- **Kubernetes** — Pods, Deployments, Services, ConfigMaps & Secrets, self-healing replicas, scaling, zero-downtime rolling updates, multi-service (microservices) deployment
- **CI/CD with GitHub Actions** — build/test pipelines per stack, environments/secrets management, job dependencies, reusable workflows
- **Continuous Deployment to AWS EKS** — cluster provisioning and deployment of both single-service and multi-service apps to a managed Kubernetes cluster

## Repository Structure

```
docker/
├── app1-hello/                  # Minimal "hello world" service
│   ├── node/                    # Express implementation
│   ├── python/                  # Flask implementation
│   └── springboot/hello-spring/ # Spring Boot implementation
│
└── app2-tax-calculator/         # Two-service microservice demo
    ├── node/
    │   ├── service-a/           # Price API — calls service-b for tax
    │   └── service-b/           # Tax lookup API
    ├── python/
    │   ├── service-a/           # Flask equivalent of service-a
    │   └── service-b/           # Flask equivalent of service-b
    ├── spring/
    │   ├── service-a/           # Spring Boot equivalent of service-a
    │   └── service-b/           # Spring Boot equivalent of service-b
    └── tax-calculator-frontend/ # React + Vite + Tailwind UI
```

Each language variant of a given app is functionally interchangeable — same routes, same response shape — so any Node/Python/Spring combination of service-a and service-b can talk to each other.

## Apps

### app1-hello

A single endpoint that returns a greeting, the current `ENV_VALUE`, and the container/host name — useful for demonstrating environment injection and load balancing across replicas.

| Stack      | Route | Default Port | Env Vars |
|------------|-------|---------------|----------|
| Node       | `GET /` | 3000 (`PORT`) | `PORT`, `ENV_VALUE` |
| Python     | `GET /` | 3000 | `ENV_VALUE` |
| Spring Boot| `GET /` | 8080 | `ENV_VALUE` |

Example response:
```json
{ "message": "Hello from Simple App (Node)", "env": "No env set", "container": "a1b2c3d4" }
```

### app2-tax-calculator

Two cooperating services:

- **service-b** — looks up a flat tax rate for a country code (`IN`, `US`, `EU`, default `10`).
- **service-a** — accepts an `amount` and `country`, calls service-b, and returns the computed total.

| Stack      | service-a port | service-b port |
|------------|-----------------|-----------------|
| Node       | 3000 | 4000 |
| Python     | 3000 | 4000 |
| Spring Boot| 8080 | 8080 |

**service-a**
```
GET /price?amount=100&country=IN
→ { "service": "A", "amount": 100, "tax": 18, "total": 118, "container": "...", "service_b_container": "..." }
```

**service-b**
```
GET /tax?country=IN
→ { "service": "B", "country": "IN", "tax": 18, "container": "..." }
```

Key env vars for service-a: `TAX_SERVICE_URL` (defaults to `http://service-b:4000`), `FRONTEND_URL` (for CORS).

**tax-calculator-frontend** — a React + Vite + Tailwind UI that calls service-a's `/price` endpoint and displays amount, tax, total, and which containers served the request. Configure the backend URL via `VITE_API_URL` (defaults to `http://localhost:3000`).

## Running Locally

Pick one implementation per app (they're drop-in equivalents) and run natively — no Dockerfiles are included yet in this stage of the masterclass.

**Node**
```bash
cd docker/app1-hello/node
npm install
node app.js
```

**Python**
```bash
cd docker/app1-hello/python
pip install -r requirements.txt
python main.py
```

**Spring Boot**
```bash
cd docker/app1-hello/springboot/hello-spring
./mvnw spring-boot:run
```

**Tax calculator (2 services + frontend)**
```bash
# Terminal 1 — service-b (tax lookup)
cd docker/app2-tax-calculator/node/service-b
npm install && node index.js

# Terminal 2 — service-a (price, depends on service-b)
cd docker/app2-tax-calculator/node/service-a
npm install
TAX_SERVICE_URL=http://localhost:4000 node index.js

# Terminal 3 — frontend
cd docker/app2-tax-calculator/tax-calculator-frontend
npm install
npm run dev
```


