# Runbook — [Service Name]

> A runbook is for someone on-call at 3am. Each section must be executable
> without needing additional context. If a procedure has no concrete commands
> or does not say who executes it, it is not complete.

**Service:** [name]
**Repository:** [URL]
**Version:** 1.0
**Last updated:** YYYY-MM-DD

---

## 1. Quick service information

| Field | Value |
|-------|-------|
| Local port | [8001] |
| Production URL | [https://api.domain.com/service] |
| Staging URL | [https://staging.api.domain.com/service] |
| Dashboard | [Grafana URL] |
| Alert channel | [#alerts on Slack / PagerDuty] |
| Escalation | [name and contact of the responsible person] |
| Target RTO | [30 min] |
| Target RPO | [5 min] |

---

## 2. Verify the service is healthy

```bash
# Health check
curl https://api.domain.com/service/health

# Expected response:
# {"status": "ok", "version": "1.2.3", "db": "connected"}
```

---

## 3. Frequent alerts and what to do

### Alert: High 5xx error rate

**Symptom:** Error rate > 2% for 5 minutes.

```bash
# 1. View recent logs
kubectl logs -n [namespace] -l app=[service-name] --tail=100 | grep ERROR

# 2. View pod status
kubectl get pods -n [namespace] -l app=[service-name]

# 3. View recent deployment events
kubectl describe deployment [service-name] -n [namespace]
```

**Decision tree:**
- Crash loops present? → See section 3.3
- DB errors? → See section 4.1
- External service errors? → See section 4.2
- Recent deploy? → Evaluate rollback (section 5.3)

---

### Alert: High latency (p95 > SLO)

```bash
# View pod resource usage
kubectl top pods -n [namespace]

# View slow queries in the DB
kubectl exec -n [namespace] [postgres-pod] -- psql -U [user] -d [db] \
  -c "SELECT pid, now()-query_start AS duration, query
      FROM pg_stat_activity
      WHERE state != 'idle' AND now()-query_start > interval '5 seconds'
      ORDER BY duration DESC LIMIT 5;"
```

---

### Alert: Pod in CrashLoopBackOff

```bash
# View crash reason
kubectl describe pod [pod-name] -n [namespace]
kubectl logs [pod-name] -n [namespace] --previous

# Manual restart
kubectl rollout restart deployment/[service-name] -n [namespace]
```

---

## 4. Procedures by component

### 4.1 Database

```bash
# Verify health
kubectl exec -n [namespace] [postgres-pod] -- pg_isready -U [user]

# View active connections
kubectl exec -n [namespace] [postgres-pod] -- psql -U [user] -d [db] \
  -c "SELECT state, count(*) FROM pg_stat_activity GROUP BY state;"
```

### 4.2 External dependencies

**If [external service X] fails:**
- The circuit breaker activates after [N] failures
- The service responds with [degraded behavior]
- External service status: [status page URL]

---

## 5. Maintenance operations

### 5.1 Scale the service

```bash
kubectl scale deployment/[service-name] -n [namespace] --replicas=[N]
kubectl get pods -n [namespace] -l app=[service-name]  # verify
```

### 5.2 Emergency deploy (hotfix)

```bash
# Only if the image has already been tested in staging
kubectl set image deployment/[service-name] \
  [container]=[image]:[hotfix-tag] -n [namespace]
kubectl rollout status deployment/[service-name] -n [namespace]
```

### 5.3 Rollback

```bash
kubectl rollout history deployment/[service-name] -n [namespace]
kubectl rollout undo deployment/[service-name] -n [namespace]
```

---

## 6. Communication during incident

| Event | Channel | Sample message |
|-------|---------|---------------|
| P0 detected | #incidents | `[P0 START] [service] degraded since [HH:MM]. Investigating.` |
| Update every 15 min | #incidents | `[P0 UPDATE] Probable cause: [...]. Action in progress: [...]. ETA: [...]` |
| Resolution | #incidents + stakeholders | `[P0 RESOLVED] Duration: [X min]. Root cause: [...]. Post-mortem: [date]` |

---

## 7. Post-incident checklist

- [ ] Service stable with metrics at SLO target
- [ ] Error budget updated in dashboard
- [ ] Incident recorded in `13-operations/incident-management.md`
- [ ] Stakeholders notified of resolution
- [ ] Improvement ticket created in `15-project-control/technical-backlog.md`
- [ ] Post-mortem scheduled (if P0 or P1)
