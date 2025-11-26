# MDPCL System Architecture

## Overview

The Midnight Decentralized Private Compute Layer (MDPCL) is a privacy-preserving, decentralized compute infrastructure that enables Midnight DApps to execute encrypted workloads with verifiable results.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              MDPCL Architecture                              │
└─────────────────────────────────────────────────────────────────────────────┘

┌──────────────┐     ┌──────────────────┐     ┌─────────────────────────────┐
│  Midnight    │     │   MDPCL Gateway  │     │   Private Compute Pool      │
│  DApp/Client │────▶│   (API Layer)    │────▶│   (Worker Nodes)            │
└──────────────┘     └──────────────────┘     └─────────────────────────────┘
       │                     │                           │
       │                     ▼                           ▼
       │              ┌──────────────┐          ┌───────────────┐
       │              │  Job Queue   │          │ Result Store  │
       │              │  (Redis)     │          │ (Encrypted)   │
       │              └──────────────┘          └───────────────┘
       │                                                │
       └────────────────────────────────────────────────┘
                    Encrypted Results Return
```

## Core Components

### 1. Gateway/API Layer

The Gateway serves as the entry point for all client interactions.

```
┌─────────────────────────────────────────────────────────────────┐
│                        Gateway Service                          │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐ │
│  │ Auth Module │  │ Rate Limiter│  │ Request Validator       │ │
│  └─────────────┘  └─────────────┘  └─────────────────────────┘ │
├─────────────────────────────────────────────────────────────────┤
│                        API Endpoints                            │
│  ┌────────────────┐ ┌────────────────┐ ┌────────────────────┐  │
│  │ POST /submit   │ │ GET /status    │ │ GET /result        │  │
│  │                │ │                │ │                    │  │
│  │ Submit encrypted│ │ Check job     │ │ Retrieve encrypted │  │
│  │ workload       │ │ status        │ │ results            │  │
│  └────────────────┘ └────────────────┘ └────────────────────┘  │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────────┐   │
│  │              Encryption/Decryption Interface            │   │
│  │  • Client-side encryption key management                │   │
│  │  • Envelope encryption for workloads                    │   │
│  │  • Result integrity verification                        │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

**Responsibilities:**
- Authenticate and authorize requests (API keys, wallet signatures)
- Validate encrypted workload format
- Queue jobs to the compute pool
- Return job status and encrypted results
- Rate limiting and abuse prevention

### 2. Private Compute Pool

The compute pool consists of permissioned worker nodes that execute encrypted workloads.

```
┌─────────────────────────────────────────────────────────────────┐
│                     Private Compute Pool                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    Job Scheduler                        │   │
│  │  • Job distribution across worker nodes                 │   │
│  │  • Load balancing (round-robin / least-connections)     │   │
│  │  • Job retry and failure handling                       │   │
│  │  • Priority queue management                            │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              │                                  │
│              ┌───────────────┼───────────────┐                  │
│              ▼               ▼               ▼                  │
│  ┌───────────────┐ ┌───────────────┐ ┌───────────────┐         │
│  │  Worker Node  │ │  Worker Node  │ │  Worker Node  │         │
│  │      #1       │ │      #2       │ │      #N       │         │
│  ├───────────────┤ ├───────────────┤ ├───────────────┤         │
│  │ TEE/Enclave   │ │ TEE/Enclave   │ │ TEE/Enclave   │         │
│  │ Environment   │ │ Environment   │ │ Environment   │         │
│  ├───────────────┤ ├───────────────┤ ├───────────────┤         │
│  │ • Decrypt job │ │ • Decrypt job │ │ • Decrypt job │         │
│  │ • Execute     │ │ • Execute     │ │ • Execute     │         │
│  │ • Encrypt out │ │ • Encrypt out │ │ • Encrypt out │         │
│  │ • Sign proof  │ │ • Sign proof  │ │ • Sign proof  │         │
│  └───────────────┘ └───────────────┘ └───────────────┘         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**Worker Node Responsibilities:**
- Pull jobs from the queue
- Decrypt workloads within secure enclave
- Execute computation
- Encrypt results with client's public key
- Generate execution proof/receipt
- Report status back to scheduler

### 3. Data Flow Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           End-to-End Data Flow                              │
└─────────────────────────────────────────────────────────────────────────────┘

Step 1: Job Submission
┌──────────┐                    ┌──────────┐
│  Client  │ ──────────────────▶│ Gateway  │
│          │  Encrypted Payload │          │
│          │  + Client PubKey   │          │
└──────────┘                    └──────────┘
                                     │
Step 2: Job Queuing                  │
                                     ▼
                               ┌──────────┐
                               │  Queue   │
                               │ (Redis)  │
                               └──────────┘
                                     │
Step 3: Job Execution                │
                                     ▼
                               ┌──────────┐
                               │  Worker  │
                               │  (TEE)   │
                               │          │
                               │ Decrypt  │
                               │ Execute  │
                               │ Encrypt  │
                               │ Sign     │
                               └──────────┘
                                     │
Step 4: Result Storage               │
                                     ▼
                               ┌──────────┐
                               │  Result  │
                               │  Store   │
                               └──────────┘
                                     │
Step 5: Result Retrieval             │
┌──────────┐                         │
│  Client  │ ◀───────────────────────┘
│          │  Encrypted Result
│          │  + Execution Proof
└──────────┘
```

## Security Architecture

### Encryption Model

```
┌─────────────────────────────────────────────────────────────────┐
│                    Encryption Architecture                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Client-Side Encryption:                                        │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Workload ──▶ AES-256-GCM ──▶ Encrypted Payload         │   │
│  │                    │                                     │   │
│  │              Symmetric Key                               │   │
│  │                    │                                     │   │
│  │              RSA/ECIES Encrypt                           │   │
│  │                    │                                     │   │
│  │              Worker Public Key                           │   │
│  │                    ▼                                     │   │
│  │           Encrypted Symmetric Key                        │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  Worker-Side Processing (Inside TEE):                           │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  1. Decrypt symmetric key with worker private key        │   │
│  │  2. Decrypt payload with symmetric key                   │   │
│  │  3. Execute computation                                  │   │
│  │  4. Encrypt result with client's public key              │   │
│  │  5. Generate execution proof (hash + signature)          │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Trust Model

```
┌─────────────────────────────────────────────────────────────────┐
│                        Trust Boundaries                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  TRUSTED:                                                       │
│  ├── Client device (encryption/decryption)                      │
│  ├── Worker TEE/Enclave (isolated execution)                    │
│  └── Cryptographic proofs (verification)                        │
│                                                                 │
│  UNTRUSTED (by design):                                         │
│  ├── Gateway service (sees only encrypted data)                 │
│  ├── Job queue (encrypted payloads only)                        │
│  ├── Network transport (TLS + payload encryption)               │
│  └── Worker host OS (TEE isolation)                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## Component Specifications

### API Specification

```yaml
# POST /api/submit
Request:
  encrypted_payload: base64    # AES-256-GCM encrypted workload
  encrypted_key: base64        # RSA encrypted symmetric key
  client_pubkey: base64        # Client's public key for result encryption
  workload_type: string        # "ai_inference" | "pos_validation" | "general"
  priority: int                # 1-10 (optional)
  callback_url: string         # Webhook for completion (optional)

Response:
  job_id: uuid
  status: "queued"
  estimated_wait: int          # seconds
  created_at: timestamp

# GET /api/status/{job_id}
Response:
  job_id: uuid
  status: "queued" | "processing" | "completed" | "failed"
  progress: int                # 0-100 (if available)
  worker_id: string            # assigned worker (if processing)
  created_at: timestamp
  started_at: timestamp        # (if processing/completed)
  completed_at: timestamp      # (if completed)

# GET /api/result/{job_id}
Response:
  job_id: uuid
  status: "completed"
  encrypted_result: base64     # Encrypted with client's public key
  execution_proof:
    result_hash: string        # SHA-256 of plaintext result
    worker_signature: string   # Worker's signature over hash
    worker_pubkey: string      # For verification
    execution_time_ms: int
    worker_id: string
  completed_at: timestamp
```

### Job Schema

```json
{
  "job_id": "uuid",
  "encrypted_payload": "base64...",
  "encrypted_key": "base64...",
  "client_pubkey": "base64...",
  "workload_type": "ai_inference",
  "priority": 5,
  "status": "queued",
  "created_at": "2025-01-01T00:00:00Z",
  "started_at": null,
  "completed_at": null,
  "worker_id": null,
  "retry_count": 0,
  "max_retries": 3
}
```

## Technology Stack

```
┌─────────────────────────────────────────────────────────────────┐
│                       Technology Stack                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Gateway/API Layer:                                             │
│  ├── Language: Python 3.11+                                     │
│  ├── Framework: FastAPI                                         │
│  ├── ASGI Server: Uvicorn                                       │
│  └── Validation: Pydantic                                       │
│                                                                 │
│  Compute Pool:                                                  │
│  ├── Scheduler: Python + Celery                                 │
│  ├── Worker: Python with cryptography libraries                 │
│  ├── Queue: Redis                                               │
│  └── TEE: Intel SGX / AMD SEV (future)                          │
│                                                                 │
│  Cryptography:                                                  │
│  ├── Symmetric: AES-256-GCM                                     │
│  ├── Asymmetric: RSA-4096 or X25519                             │
│  ├── Hashing: SHA-256                                           │
│  └── Library: cryptography (Python)                             │
│                                                                 │
│  Storage:                                                       │
│  ├── Job Queue: Redis                                           │
│  ├── Results: Redis (TTL) or PostgreSQL                         │
│  └── Logs: File-based (/mnt/logs/)                              │
│                                                                 │
│  Infrastructure:                                                │
│  ├── Containers: Docker                                         │
│  ├── Orchestration: Docker Compose (POC) / K8s (production)     │
│  └── Monitoring: Prometheus + Grafana (optional)                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## Directory Structure

```
MDPCL/
├── README.md
├── LICENSE                      # MIT/Apache 2.0
├── pyproject.toml               # Project dependencies
├── docker-compose.yml           # Local development stack
├── Dockerfile                   # Gateway service image
├── Dockerfile.worker            # Worker node image
│
├── src/
│   └── mdpcl/
│       ├── __init__.py
│       │
│       ├── gateway/             # API Layer
│       │   ├── __init__.py
│       │   ├── main.py          # FastAPI app entry
│       │   ├── routes/
│       │   │   ├── __init__.py
│       │   │   ├── submit.py    # POST /api/submit
│       │   │   ├── status.py    # GET /api/status/{job_id}
│       │   │   └── result.py    # GET /api/result/{job_id}
│       │   ├── middleware/
│       │   │   ├── __init__.py
│       │   │   ├── auth.py      # API key / wallet auth
│       │   │   └── rate_limit.py
│       │   └── schemas/
│       │       ├── __init__.py
│       │       ├── job.py       # Pydantic models
│       │       └── response.py
│       │
│       ├── compute_pool/        # Compute Layer
│       │   ├── __init__.py
│       │   ├── scheduler.py     # Job distribution logic
│       │   ├── worker.py        # Worker node process
│       │   ├── tasks.py         # Celery task definitions
│       │   └── executor.py      # Workload execution engine
│       │
│       ├── crypto/              # Cryptography Module
│       │   ├── __init__.py
│       │   ├── encryption.py    # AES-256-GCM operations
│       │   ├── keys.py          # Key generation/management
│       │   └── proofs.py        # Execution proof generation
│       │
│       ├── storage/             # Data Persistence
│       │   ├── __init__.py
│       │   ├── redis_client.py  # Redis connection
│       │   ├── job_store.py     # Job CRUD operations
│       │   └── result_store.py  # Result storage
│       │
│       └── config/              # Configuration
│           ├── __init__.py
│           └── settings.py      # Environment-based config
│
├── demo/                        # Reference DApp
│   ├── mdpcl_demo.py            # CLI demo script
│   ├── client.py                # MDPCL Python client
│   └── ui/                      # Web UI (optional)
│       ├── index.html
│       ├── app.js
│       └── styles.css
│
├── tests/
│   ├── __init__.py
│   ├── conftest.py              # Pytest fixtures
│   ├── test_api.py              # API endpoint tests
│   ├── test_crypto.py           # Cryptography tests
│   ├── test_worker.py           # Worker tests
│   └── test_e2e.py              # End-to-end tests
│
├── scripts/
│   ├── setup_node.sh            # Worker node setup
│   ├── run_demo.py              # Demo runner
│   └── generate_keys.py         # Key pair generation
│
├── docs/
│   ├── architecture.md          # This document
│   ├── integration.md           # Integration guide
│   ├── api-reference.md         # API documentation
│   └── troubleshooting.md       # Common issues
│
└── specifications/
    ├── raw-requirements.txt
    └── Project-Summary.md
```

## Deployment Architecture

### Development (Docker Compose)

```yaml
# docker-compose.yml
services:
  gateway:
    build: .
    ports:
      - "8000:8000"
    depends_on:
      - redis
    environment:
      - REDIS_URL=redis://redis:6379

  worker:
    build:
      dockerfile: Dockerfile.worker
    deploy:
      replicas: 2
    depends_on:
      - redis
    environment:
      - REDIS_URL=redis://redis:6379

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
```

### Production (Conceptual)

```
┌─────────────────────────────────────────────────────────────────┐
│                    Production Deployment                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Load Balancer (nginx/HAProxy)                                  │
│           │                                                     │
│           ▼                                                     │
│  ┌─────────────────┐                                            │
│  │ Gateway Cluster │  (2+ replicas)                             │
│  └─────────────────┘                                            │
│           │                                                     │
│           ▼                                                     │
│  ┌─────────────────┐                                            │
│  │  Redis Cluster  │  (Primary + Replicas)                      │
│  └─────────────────┘                                            │
│           │                                                     │
│           ▼                                                     │
│  ┌─────────────────────────────────────────┐                    │
│  │         Worker Pool (N nodes)           │                    │
│  │  ┌────────┐ ┌────────┐ ┌────────┐       │                    │
│  │  │Worker 1│ │Worker 2│ │Worker N│       │                    │
│  │  └────────┘ └────────┘ └────────┘       │                    │
│  └─────────────────────────────────────────┘                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## Sequence Diagrams

### Job Submission Flow

```
Client              Gateway             Redis              Worker
  │                    │                  │                   │
  │ POST /api/submit   │                  │                   │
  │ (encrypted payload)│                  │                   │
  │───────────────────▶│                  │                   │
  │                    │                  │                   │
  │                    │ Validate request │                   │
  │                    │ Generate job_id  │                   │
  │                    │                  │                   │
  │                    │ LPUSH job_queue  │                   │
  │                    │─────────────────▶│                   │
  │                    │                  │                   │
  │                    │ SET job:{id}     │                   │
  │                    │─────────────────▶│                   │
  │                    │                  │                   │
  │ 202 Accepted       │                  │                   │
  │ {job_id, status}   │                  │                   │
  │◀───────────────────│                  │                   │
  │                    │                  │                   │
  │                    │                  │ BRPOP job_queue   │
  │                    │                  │◀──────────────────│
  │                    │                  │                   │
  │                    │                  │ GET job:{id}      │
  │                    │                  │◀──────────────────│
  │                    │                  │                   │
  │                    │                  │    Execute job    │
  │                    │                  │    (encrypted)    │
  │                    │                  │                   │
  │                    │                  │ SET result:{id}   │
  │                    │                  │◀──────────────────│
  │                    │                  │                   │
  │                    │                  │ UPDATE job status │
  │                    │                  │◀──────────────────│
  │                    │                  │                   │
```

### Result Retrieval Flow

```
Client              Gateway             Redis
  │                    │                  │
  │ GET /api/result    │                  │
  │    /{job_id}       │                  │
  │───────────────────▶│                  │
  │                    │                  │
  │                    │ GET job:{id}     │
  │                    │─────────────────▶│
  │                    │                  │
  │                    │ GET result:{id}  │
  │                    │─────────────────▶│
  │                    │                  │
  │ 200 OK             │                  │
  │ {encrypted_result, │                  │
  │  execution_proof}  │                  │
  │◀───────────────────│                  │
  │                    │                  │
  │ Client decrypts    │                  │
  │ result locally     │                  │
  │                    │                  │
```

## Error Handling

| Error Code | Description | Client Action |
|------------|-------------|---------------|
| 400 | Invalid payload format | Fix request format |
| 401 | Unauthorized | Provide valid API key |
| 404 | Job not found | Check job_id |
| 429 | Rate limited | Retry after delay |
| 500 | Internal error | Retry with backoff |
| 503 | Service unavailable | Retry later |

## Monitoring & Logging

```
Logs Location: /mnt/logs/
├── gateway.log          # API request/response logs
├── worker.log           # Worker execution logs
├── scheduler.log        # Job scheduling logs
└── compute.log          # Computation execution logs

Metrics (Prometheus):
├── mdpcl_jobs_submitted_total
├── mdpcl_jobs_completed_total
├── mdpcl_jobs_failed_total
├── mdpcl_job_execution_duration_seconds
├── mdpcl_queue_depth
└── mdpcl_worker_count
```

## Future Enhancements

1. **TEE Integration**: Intel SGX / AMD SEV for hardware-level isolation
2. **Multi-region**: Geographically distributed worker pools
3. **Cardano Integration**: On-chain job receipts and payment
4. **Advanced Scheduling**: Priority queues, resource-based routing
5. **Batch Processing**: Multiple workloads in single submission
