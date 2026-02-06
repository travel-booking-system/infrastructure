# CI/CD Overview

Each microservice has its own CI pipeline.

## CI responsibilities
- Build the service
- Run unit tests
- Build Docker image

## CD (future)
- Push image to registry
- Deploy to Kubernetes or cloud provider

CI/CD pipelines are defined inside each service repository.
