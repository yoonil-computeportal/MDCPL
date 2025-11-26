# MDCL - Midnight Decentralized Compute Layer

A privacy-preserving, decentralized compute infrastructure for the Midnight/Cardano ecosystem. MDCL enables encrypted workload execution with verifiable results, allowing DApps to run confidential computations without exposing sensitive data.

## Overview

MDCL provides a secure compute layer where:

- **Workloads are encrypted end-to-end** - Data is never exposed in plaintext outside the client
- **Execution is verifiable** - Cryptographic proofs ensure computation integrity
- **Infrastructure is decentralized** - Multiple worker nodes prevent single points of failure
- **Results are private** - Only the submitting client can decrypt results

```
┌──────────────┐     ┌──────────────────┐     ┌─────────────────────────────┐
│  Midnight    │     │   MDCL Gateway   │     │       Compute Pool          │
│  DApp/Client │────▶│   (API Layer)    │────▶│   (Worker Nodes)            │
└──────────────┘     └──────────────────┘     └─────────────────────────────┘
       │                                                │
       └────────────────────────────────────────────────┘
                    Encrypted Results Return
```

## Features

- **Hybrid Encryption**: AES-256-GCM + RSA-4096 for efficient, secure data handling
- **RESTful API**: Simple endpoints for job submission, status, and result retrieval
- **Execution Proofs**: Cryptographic signatures verify computation integrity
- **Priority Queues**: High-priority jobs processed first
- **Auto-Retry**: Failed jobs automatically retried with exponential backoff
- **Python SDK**: Easy integration for Python applications
- **CLI Tool**: Command-line interface for quick testing
- **Web Demo**: Browser-based demonstration of the full workflow

## Quick Start

### Prerequisites

- Python 3.11+
- Docker & Docker Compose
- Redis 7.x (included in Docker Compose)

### Installation

```bash
# Clone the repository
git clone https://github.com/yoonil-computeportal/MDCL.git
cd MDCL

# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Start services
docker-compose up -d
```

### Run the Demo

```bash
# Run the interactive demo
python demo/run_demo.py
```

### Using the SDK

```python
from MDCL import MDCLClient

# Initialize client
client = MDCLClient(
    gateway_url="http://localhost:8000",
    api_key="your-api-key"
)

# Generate encryption keys
client.generate_keys()

# Submit an encrypted workload
job = await client.submit(
    code="def compute(x): return x * 2",
    input_data={"x": 42}
)

# Wait for result
result = await client.get_result(job.job_id)

# Verify execution proof
assert client.verify_proof(result.execution_proof)

print(result.decrypted_data)  # {"result": 84}
```

### Using the CLI

```bash
# Generate keys
MDCL keys generate

# Submit a job
MDCL submit --code "def compute(x): return x * 2" --input '{"x": 42}'

# Check status
MDCL status <job_id>

# Get result
MDCL result <job_id>
```

## API Reference

### POST /api/submit

Submit an encrypted workload for execution.

```bash
curl -X POST http://localhost:8000/api/submit \
  -H "X-API-Key: your-api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "encrypted_payload": "<base64>",
    "encrypted_key": "<base64>",
    "client_pubkey": "<base64>",
    "workload_type": "general"
  }'
```

Response:
```json
{
  "job_id": "abc123-def456",
  "status": "queued",
  "created_at": "2025-01-01T00:00:00Z"
}
```

### GET /api/status/{job_id}

Check the status of a submitted job.

```bash
curl http://localhost:8000/api/status/abc123-def456 \
  -H "X-API-Key: your-api-key"
```

Response:
```json
{
  "job_id": "abc123-def456",
  "status": "processing",
  "progress": 50,
  "worker_id": "worker-1"
}
```

### GET /api/result/{job_id}

Retrieve the encrypted result of a completed job.

```bash
curl http://localhost:8000/api/result/abc123-def456 \
  -H "X-API-Key: your-api-key"
```

Response:
```json
{
  "job_id": "abc123-def456",
  "encrypted_result": "<base64>",
  "execution_proof": {
    "result_hash": "sha256...",
    "worker_signature": "<base64>",
    "execution_time_ms": 234
  }
}
```

## Project Structure

```
MDCL/
├── src/MDCL/
│   ├── gateway/           # API Layer (FastAPI)
│   │   ├── routes/        # API endpoints
│   │   ├── middleware/    # Auth, rate limiting
│   │   └── schemas/       # Request/response models
│   ├── compute_pool/      # Compute Layer
│   │   ├── scheduler.py   # Job distribution
│   │   ├── worker.py      # Worker node
│   │   └── executor.py    # Workload execution
│   ├── crypto/            # Cryptography
│   │   ├── encryption.py  # AES-256-GCM
│   │   ├── keys.py        # RSA key management
│   │   └── proofs.py      # Execution proofs
│   └── storage/           # Data persistence
│       ├── redis_client.py
│       ├── job_store.py
│       └── result_store.py
├── demo/
│   ├── client.py          # Python SDK
│   ├── cli.py             # CLI tool
│   ├── run_demo.py        # Demo script
│   └── ui/                # Web demo
├── tests/                 # Test suite
├── docs/                  # Documentation
└── specifications/        # Feature specs
```

## Documentation

- [Architecture](docs/architecture.md) - System design and component overview
- [API Reference](docs/api-reference.md) - Detailed API documentation
- [Integration Guide](docs/integration.md) - How to integrate with MDCL
- [Troubleshooting](docs/troubleshooting.md) - Common issues and solutions

## Development

### Running Tests

```bash
# Run all tests
pytest tests/

# Run with coverage
pytest tests/ --cov=src/MDCL --cov-report=html

# Run specific test file
pytest tests/test_crypto.py -v
```

### Code Quality

```bash
# Lint code
ruff check src/

# Format code
black src/ tests/

# Type check
mypy src/
```

### Docker Development

```bash
# Build images
docker-compose build

# Start services
docker-compose up -d

# View logs
docker-compose logs -f gateway
docker-compose logs -f worker

# Stop services
docker-compose down
```

## Security

MDCL is designed with security as a core principle:

- **End-to-end encryption**: All workloads encrypted with AES-256-GCM
- **Key isolation**: Private keys never leave client/worker boundaries
- **No plaintext storage**: Encrypted data only in transit and at rest
- **Execution proofs**: Cryptographic verification of computation integrity
- **Input validation**: All API inputs sanitized and validated

### Reporting Security Issues

Please report security vulnerabilities to security@computeportal.io

## Contributing

We welcome contributions! Please see [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## License

This project is licensed under the MIT License - see [LICENSE](LICENSE) file for details.

## Acknowledgments

- **HPEC DAO** - Project coordination and infrastructure
- **Compute Portal** - Technical development and expertise
- **Catalyst** - Funding and support

## Contact

- **Project Lead**: Mauricio Prieto - [LinkedIn](https://www.linkedin.com/in/mauricio-prieto-801545124/)
- **Technical Advisor**: Jerry V Hall - [LinkedIn](https://www.linkedin.com/in/jerry-v-hall-b43b937/)
- **Developer/CTO**: Yoonil Choi - [LinkedIn](https://www.linkedin.com/in/yoonil-choi/)

## Links

- [HPEC DAO](https://hpecdao.net/)
- [Compute Portal](https://computeportal.io/)
- [GitHub Organization](https://github.com/yoonil-computeportal)
