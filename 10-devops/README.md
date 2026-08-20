# 10 — DevOps

> **What is this?** How code goes from the developer's computer to production,
> and how it is maintained in each environment. CI/CD, environments, and local configuration.

## Why this section exists

Without documented DevOps:
- Each developer has their own deploy process (and fails differently)
- "It works on my machine" is the most expensive answer in software
- Releases are high-risk events instead of routine

---

## What is here and how to fill it in

### `local-setup.md` ⭐ (Most urgent)
How to start THE ENTIRE SYSTEM on a new computer from scratch.
**Fill in:** step by step, omitting nothing, assuming the reader has never touched the project.
Include: software prerequisites, environment variables, exact commands, how to verify it works.

**Format:**
```markdown
## Prerequisites
- [ ] Docker Desktop >= 24.0
- [ ] [Language/Runtime] >= [version]
- [ ] [Tool] >= [version]

## Steps
1. Clone the main repository: `git clone [url]`
2. Copy environment variables: `cp .env.example .env`
3. Start infrastructure: `docker-compose up -d`
4. Wait for services to be ready: `./scripts/wait-for-services.sh`
5. Verify: `curl http://localhost:8080/health`

## Local ports
| Service | Port |
|---------|------|
| API Gateway | 8080 |
| [Service 1] | 8001 |
| RabbitMQ UI | 15672 |
| Adminer (DB) | 8090 |

## Common issues
- **Port in use:** `lsof -i :8080` to see what is using it
- **DB not connecting:** verify Docker is running
```

### `environments.md` ⭐
Description of all project environments.
**Fill in:** local, development, staging, production. For each: purpose, URL, who has access,
how it is deployed, what data it has.

**Format:**
```markdown
| Environment | URL | Purpose | Access | DB | Deploy |
|-------------|-----|---------|--------|-----|--------|
| local | localhost | Individual development | All | Test data | Manual |
| dev | dev.api.domain.com | Continuous integration | Team | Test data | Automatic (push to dev) |
| staging | staging.api.domain.com | QA / validation | Team + PO | Anonymized data | Manual (approval) |
| prod | api.domain.com | Production | Ops team | Real data | Manual (double approval) |
```

### `ci-cd.md` ⭐
Description of the continuous integration and deployment pipeline.
**Fill in:** what tool (GitHub Actions, GitLab CI, Jenkins), what steps it executes,
when each pipeline is triggered, what it verifies before allowing a merge.

**Format:**
```markdown
## PR Pipeline (runs on every Pull Request)
1. Lint and code format
2. Unit tests
3. Integration tests
4. Coverage analysis (minimum [X]%)
5. Security analysis (SAST)

## Pipeline on merge to `dev`
1. All of the above
2. Docker image build
3. Deploy to dev environment
4. Smoke tests in dev

## Release pipeline to production
1. Manual approval by [role]
2. Deploy to staging
3. Acceptance test suite
4. Manual approval by [role]
5. Deploy to production with blue-green / canary
```

---

## Correlations with other sections

| This section is fed by... | And feeds into... |
|---------------------------|-------------------|
| `05-architecture/deployment.md` → how it is deployed | Pipeline implementation |
| `11-quality/testing-strategy.md` → which tests to run | Pipeline steps |
| `09-microservices/` → each service to deploy | Which images the pipeline builds |
| `13-operations/` → post-deploy monitoring | What the pipeline verifies |

---

## Questions this section must answer

- How do I start the system locally in 30 minutes?
- What happens automatically when I push?
- How does code get to production?
- How many environments are there and what is each one for?
