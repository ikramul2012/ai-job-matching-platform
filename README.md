# AI Job Matching Platform 

---

## Architecture Diagram (Mermaid)

```mermaid
flowchart TD
  %% Layout
  subgraph UserLayer[User / Client]
    A[Browser]
  end

  subgraph CDN_Frontend[CDN / Frontend Hosting]
    F[Vercel (Frontend)]
  end

  subgraph APIGW[API Gateway]
    GW[API Gateway (Laravel)]
  end

  subgraph Services[Microservices Cluster]
    Auth[Auth Service - JWT/OAuth]
    Profile[User Profile Service - Resume upload & parse]
    JobSvc[Job Service - CRUD + job metadata]
    ML[ML Matching Service - FastAPI + embeddings]
    Career[Career Rec Service - Skill-gap + courses]
    Notify[Notification Service - Email / Push]
  end

  subgraph DataLayer[Data & Infra]
    DB[(MySQL)]
    Vector[(Pinecone)]
    DVCStorage[(DVC Remote / S3 / Git LFS)]
    MLflow[(MLflow Tracking Server)]
    Redis[(Redis Queue / Cache)]
    Logs[(Grafana Loki / Cloud Logs)]
  end

  A -->|HTTPS| F
  F -->|HTTPS| GW

  GW --> Auth
  GW --> Profile
  GW --> JobSvc
  GW --> ML
  GW --> Career
  GW --> Notify

  Profile -->|enqueue| Redis
  Profile --> ML
  JobSvc --> ML
  ML --> Vector
  ML --> DB
  Career --> DB
  Career --> ML

  Auth --> DB
  JobSvc --> DB
  Profile --> DB

  ML -->|metrics & experiments| MLflow
  ML -->|datasets & artifacts| DVCStorage

  GW --> Logs
  ML --> Logs
  JobSvc --> Logs
  Profile --> Logs

  Redis -->|task queue| Profile
```

---

## Sequence Diagram: Job Matching Flow

```mermaid
sequenceDiagram
    participant User
    participant Frontend
    participant API_Gateway
    participant ProfileService
    participant JobService
    participant MLService
    participant CareerService

    User->>Frontend: Upload Resume
    Frontend->>API_Gateway: POST /profiles/upload
    API_Gateway->>ProfileService: Store resume, parse
    ProfileService-->>API_Gateway: Parsed profile JSON

    API_Gateway->>MLService: /embed + /upsert (candidate vector)
    JobService->>MLService: /upsert (job vectors)

    User->>Frontend: Request job recommendations
    Frontend->>API_Gateway: GET /recommend?candidate_id
    API_Gateway->>MLService: Nearest neighbor search
    MLService-->>API_Gateway: Ranked jobs
    API_Gateway->>CareerService: /gaps
    CareerService-->>API_Gateway: Missing skills + courses
    API_Gateway-->>Frontend: Combined response (jobs + gaps)
```

---

## Deployment Diagram

```mermaid
graph TD
  subgraph Cloud[Cloud / Hosting Environment]
    subgraph Cluster[Kubernetes / Docker Swarm]
      API[API Gateway]
      Auth[Auth Service]
      Profile[Profile Service]
      Job[Job Service]
      ML[ML Service]
      Career[Career Service]
      Notify[Notification Service]
    end

    DB[(Postgres DB)]
    Vector[(Chroma / FAISS Vector DB)]
    Redis[(Redis Queue)]
    MLflow[(MLflow Server)]
    DVC[(DVC Remote Storage)]
    Logs[(Grafana Loki / Monitoring)]
  end

  User[End Users]

  User -->|HTTPS| API
  API --> Auth
  API --> Profile
  API --> Job
  API --> ML
  API --> Career
  API --> Notify

  Profile --> Redis
  ML --> Vector
  ML --> DB
  Career --> DB
  Auth --> DB
  Job --> DB
  ML --> MLflow
  ML --> DVC
  API --> Logs
```

---

## CI/CD Pipeline Flow

```mermaid
flowchart LR
    A[Developer Commit Code] --> B[GitHub Actions CI]
    B --> C[Lint + Unit Tests]
    C --> D[Build Docker Images]
    D --> E[Push to Registry]
    E --> F[Deploy to Staging]
    F --> G[Integration + E2E Tests]
    G --> H{Tests Passed?}
    H -- Yes --> I[Deploy to Production]
    H -- No --> J[Notify Developer]
```

---

## One-line System Description

A microservice-based, modular platform that ingests candidate CVs, extracts structured profiles, indexes both candidate and job embeddings into a vector store, returns ranked job matches, computes skill gaps, and suggests personalized learning paths (free + paid) — all with reproducible MLOps and CI/CD.

---

## Microservices & Responsibilities

1. **API Gateway / BFF** — Entry point, JWT validation, orchestration.
2. **Auth Service** — Users, roles, tokens, SSO.
3. **User Profile Service** — Resume upload, parsing, embedding enqueue.
4. **Job Service** — CRUD jobs, metadata, embeddings.
5. **ML Matching Service** — Embeddings, vector store, recommendations.
6. **Career Recommendation Service** — Skill gaps + course mapping.
7. **Notification Service** — Email, SMS, push notifications.
8. **Infrastructure** — DB, vector store, Redis, DVC, MLflow, monitoring.

---

## API Contracts

* **Auth**: `/auth/register`, `/auth/login`
* **Profile**: `/profiles/upload`, `/profiles/:id`
* **Jobs**: `/jobs`, `/jobs/:id`
* **ML**: `/embed`, `/upsert`, `/recommend`, `/explain`
* **Career**: `/gaps`
* **Admin**: `/admin/reindex`

---

## Database Schema

Core tables: `users`, `candidates`, `jobs`, `skills`, `job_skills`, `recommendations`.

---

## Repo Layout

```
/ai-job-matching-platform
├─ /frontend          
├─ /backend           
├─ /ml_service        
├─ /ml                
├─ /infra             
├─ /scripts           
├─ /docs              
└─ .github/workflows  
```

---

## docker-compose

Includes Mysql, Redis, Pinecone, ML Service, Backend.

---

## Starter ML Service (FastAPI)

Minimal app with `/embed`, `/upsert`, `/recommend` endpoints using `sentence-transformers` + Pinecone.

---

## Quickstart

1. Clone repo
2. Setup `.env`
3. `docker-compose up --build`
4. Access backend (8002), ML service (8001).

---

## CI/CD

GitHub Actions: lint, test, build, deploy.

---

## MLOps

* DVC for datasets & artifacts
* MLflow for tracking experiments
* Eval metrics: Precision, Recall

---

## Monitoring

Grafana, Prometheus, Loki. Track latency, errors, embedding performance.

---

## Security

HTTPS, encryption, masked logs, rate limits, rotated credentials.

---

## Contribution & License

* `MIT` or `Apache-2.0`
* Contributing guidelines
* Contact info
