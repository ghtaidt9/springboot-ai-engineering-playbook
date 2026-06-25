# Deployment Strategy

## CI Pipeline
GitHub Actions
Stages:
- Build
- Unit Tests
- Integration Tests
- Security Scan
- Docker Build

## CD Strategy
Blue/Green Deployment
Rollback supported.
Terraform is the single source of truth.