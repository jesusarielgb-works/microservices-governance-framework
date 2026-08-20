# Deployment Environments

> Defines the system's environments, their purposes, access, and key configurations.
> **Rule:** If you need to access production directly, something is wrong in the process.
> Most problems should be detected in staging.

---

## Project environments

| Environment | Purpose | Update | Access | Data |
|-------------|---------|--------|--------|------|
| **local** | Development on the dev's machine | Manual | Individual dev | Synthetic / seed |
| **dev** | Continuous integration | Automatic (each PR merge to `develop`) | Entire team | Synthetic |
| **staging** | Pre-production, QA, demos | Automatic (each merge to `main`) | Team + PO | Anonymized from prod |
| **production** | Real users | Manual (approval required) | Tech Lead + DevOps only | Real |

---

## Environment variables per environment

> Specific variables go in the environment's secrets system (Vault, AWS Secrets, K8s Secrets).
> Local values go in `.env.example` (without real data).

### Naming convention

```
APP_[SERVICE]_[VARIABLE]

Examples:
  APP_AUTH_JWT_SECRET
  APP_ORDERS_DATABASE_URL
  APP_NOTIFICATIONS_SMTP_HOST
```

### Variables per environment (example values — real ones in the vault)

| Variable | Local | Dev | Staging | Production |
|----------|-------|-----|---------|------------|
| `DATABASE_URL` | `postgresql://dev:dev@localhost/dev_db` | Vault | Vault | Vault |
| `REDIS_URL` | `redis://localhost:6379` | Vault | Vault | Vault |
| `JWT_SECRET` | `dev-secret-only-local` | Vault | Vault | Vault |
| `LOG_LEVEL` | `debug` | `info` | `info` | `warn` |
| `NODE_ENV` | `development` | `development` | `staging` | `production` |
| `CORS_ORIGIN` | `http://localhost:3000` | `https://dev.[domain]` | `https://staging.[domain]` | `https://[domain]` |

---

## CI/CD Pipeline

```
Developer
  │
  │ git push feature/HU-XXX
  │
  ▼
┌─────────────────────────────────────────────────────────┐
│  PR PIPELINE (runs on every push to a branch)           │
│  ─────────────────────────────────────────────          │
│  1. Lint + Format check      (< 2 min)                  │
│  2. Unit Tests               (< 5 min)                  │
│  3. Build                    (< 3 min)                  │
│  ─────────────────────────────────────────────          │
│  If it fails: blocks the PR merge                       │
└─────────────────────────────────────────────────────────┘
  │
  │ PR approved and merged to `develop`
  │
  ▼
┌─────────────────────────────────────────────────────────┐
│  DEVELOP PIPELINE (deploy to dev)                       │
│  ─────────────────────────────────────────────          │
│  1. Lint + Build + Unit Tests   (< 10 min)              │
│  2. Integration Tests           (< 15 min)              │
│  3. Build Docker image          (< 5 min)               │
│  4. Deploy to dev               (< 5 min)               │
│  5. Smoke tests in dev          (< 3 min)               │
└─────────────────────────────────────────────────────────┘
  │
  │ Release: merge develop → main
  │
  ▼
┌─────────────────────────────────────────────────────────┐
│  MAIN PIPELINE (deploy to staging)                      │
│  ─────────────────────────────────────────────          │
│  1. All tests                   (< 20 min)              │
│  2. Contract Tests              (< 5 min)               │
│  3. Build + push image with tag (< 5 min)               │
│  4. Deploy to staging           (< 5 min)               │
│  5. Smoke tests in staging      (< 5 min)               │
│  6. Performance tests (k6)      (< 15 min)              │
│  7. Notify PO (ready for QA)                            │
└─────────────────────────────────────────────────────────┘
  │
  │ Manual approval by Tech Lead
  │
  ▼
┌─────────────────────────────────────────────────────────┐
│  DEPLOY TO PRODUCTION                                   │
│  ─────────────────────────────────────────────          │
│  1. Canary deploy (10% of traffic)                      │
│  2. Monitor metrics for 15 min                          │
│  3. If OK: 100% of traffic                              │
│  4. If KO: automatic rollback                           │
└─────────────────────────────────────────────────────────┘
```

---

## Production deploy strategy

### Available options

| Strategy | Description | Risk | Rollback |
|----------|-------------|------|---------|
| **Canary** | 10% → 50% → 100% with validation at each step | Low | Automatic |
| **Blue-Green** | Instant switch between two versions | Medium | Immediate |
| **Rolling** | Replaces instances one by one | Low | Slow |
| **Recreate** | Takes everything down and starts the new version | High | Manual |

**Adopted strategy:** [Canary / Blue-Green / Rolling]

### Production deploy checklist

- [ ] The release PR was reviewed and approved
- [ ] Staging tests passed (including performance)
- [ ] The PO approved the HUs included in this release
- [ ] The Tech Lead approved the deploy
- [ ] Someone is available to monitor the first 30 min post-deploy
- [ ] If there are database changes: migration ran before the code deploy
- [ ] The rollback runbook is available: `[link to runbook]`

### Rollback

```bash
# Rollback Kubernetes to the previous version
kubectl rollout undo deployment/[service-name] -n [namespace]

# Verify the rollback
kubectl rollout status deployment/[service-name] -n [namespace]

# If there is an incompatible DB migration (EMERGENCY — consult Tech Lead first)
npm run db:migrate:revert
```

---

## Deploy log

| Date | Service | Version | Environment | Deployed by | Issues |
|------|---------|---------|-------------|-------------|--------|
| [date] | [service] | v[X.Y.Z] | production | [name] | None |

---

## Correlations

- Local setup → `10-devops/local-setup.md`
- Post-deploy observability → `13-operations/observability.md`
- Runbook per service → `09-microservices/services/XX/runbook.md`
- Branch policy → `00-governance/git-conventions.md`
