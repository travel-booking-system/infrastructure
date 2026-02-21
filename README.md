# Infrastructure Repository

This repository contains the shared infrastructure configuration for the **Travel Booking Platform**.

It is responsible for:

- Local development orchestration
- Docker Compose configuration
- Environment management
- Secret handling
- CI integration test orchestration

This repository does **not** contain application business logic.

---

## Repository Structure

```
infrastructure/
│
├── .github/workflows/
│   └── ci.yml
│
├── docker/
│   ├── docker-compose.yml
│   ├── docker-compose.dev.yml
│   └── docker-compose.ci.yml
│   
│
├── env/
│   ├── dev.env.example
│   └── dev.env               (ignored)
│
├── secrets/
│   └── auth-service/
│       └── db_password.txt   (ignored)
│
└── README.md
```

---

## Prerequisites

Make sure you have:

- Docker (latest stable)
- Docker Compose v3
- Git

---

## Required Service Repositories

This repository orchestrates services but does not contain their code.

To run the full system locally, clone the required services in the same parent directory.

Example structure:

```
projects/
│
├── infrastructure/
├── auth-service/
└── api-gateway/
```

Example cloning:

```bash
git clone https://github.com/your-org/infrastructure.git
git clone https://github.com/your-org/auth-service.git
git clone https://github.com/your-org/api-gateway.git
```

Ensure image names or build contexts in `docker-compose` files match your services.

---

## Environment Setup

Inside `/env` you will find:

```
dev.env.example
```

### Step 1 – Create your local environment file

```bash
cp env/dev.env.example env/dev.env
```

Edit `dev.env` and provide required values:

```env
AUTH_DB_NAME=authdb
AUTH_DB_USER=authuser
AUTH_SERVICE_PROFILE=dev
GATEWAY_PROFILE=dev
```

> ⚠️ `dev.env` is ignored and must never be committed.

---

## Secrets Setup

Secrets are scoped per service to prevent configuration sprawl.

Current structure:

```
secrets/
└── auth-service/
    └── db_password.txt
```

### Step 2 – Create required secret files

Create:

```
secrets/auth-service/db_password.txt
```

Add the password inside:

```
your_secure_password_here
```

**Important:**

- Never commit secrets
- The `/secrets` directory is gitignored
- Each service owns its own secret folder

Naming convention:

```
<service>_<purpose>
```

Examples:

```
auth_db_password
auth_jwt_secret
```

---

## Running the System (Local Development)

From the root of the infrastructure repository:

```bash
docker compose \
  --env-file env/dev.env \
  -f docker/docker-compose.yml \
  -f docker/docker-compose.dev.yml \
  up --build
```

To stop:

```bash
docker compose down -v
```

---

## Running the CI Stack Locally

To simulate CI behavior:

```bash
docker compose \
  --env-file env/dev.env \
  -f docker/docker-compose.ci.yml \
  up --build
```

This stack:

- Uses production-like images
- Runs integration tests
- Uses isolated Docker networking

---

## Configuration Strategy

This repository follows a clear separation strategy.

**Non-sensitive configuration** is stored in `.env` files and managed per environment.

**Sensitive configuration** is stored as Docker secrets, scoped per service, and never committed.

**Environments:**

| Name | Purpose |
|------|---------|
| `dev` | Local development |
| `ci` | Integration testing |
| `prod` | Future production configuration |

---

## Logging & Resource Policies

Services use:

- JSON logging driver with rotation
- CPU and memory limits
- Restart policies on failure

These are defined in Docker Compose files.

---

## CI/CD Behavior

The GitHub Actions workflow:

1. Pulls service images from GHCR
2. Starts stack using `docker-compose.ci.yml`
3. Runs integration tests
4. Tears down the environment

Each service repository is responsible for building its Docker image and publishing it to GHCR. The infrastructure repository only orchestrates them.

---

## Adding a New Service

When introducing a new service:

1. Add service definition to Docker Compose
2. Create `secrets/<service-name>/`
3. Add required secret files
4. Update `dev.env.example`
5. Document new variables in this README

---

## What Is Not Included

Currently not included:

- Kubernetes manifests
- Cloud provider configuration
- Secret managers (Vault, AWS Secrets Manager)
- Production deployment automation

These will be added incrementally as the platform evolves.

---

## Design Principles

This infrastructure is built with clear service isolation, explicit configuration, and a production-minded Docker setup. The goal is to grow from local Docker Compose to full production-grade infrastructure without architectural rewrites.

| Principle | Implementation |
|-----------|---------------|
| Service isolation | Scoped secrets and env vars per service |
| Explicit configuration | No implicit defaults; all values declared |
| Production-minded setup | Resource limits, log rotation, restart policies |
| Scalable secret organization | Per-service secret directories |
| CI-ready orchestration | Dedicated `docker-compose.ci.yml` |