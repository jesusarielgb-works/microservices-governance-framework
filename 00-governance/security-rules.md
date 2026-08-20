# Technical Security Rules

> Mandatory technical controls that apply to all project code.
> These rules complement the security policy (`security-policy.md`) with
> concrete implementation practices.

---

## OWASP Top 10 — Controls per category

### A01 — Broken Access Control

```typescript
// ❌ BAD — trusting frontend data
const userId = req.body.userId;

// ✅ GOOD — extract from verified JWT token
const userId = req.user.sub; // req.user comes from the authentication middleware
```

**Rules:**
- Every protected endpoint MUST have the authentication middleware applied
- Permissions are verified in the Use Case, not in the Controller
- A resource is only returned if the user has `[resource]:read` permission
- Write actions require `[resource]:write` or `[resource]:delete` permission

### A02 — Cryptographic Failures

**Rules:**
- Passwords: use **bcrypt** with cost factor ≥ 12. Never MD5 or SHA-1 for passwords
- JWTs: sign with RS256 (asymmetric). Never HS256 in production with a weak secret
- Sensitive data in transit: HTTPS mandatory in all environments except local
- Sensitive data at rest: encrypt with AES-256-GCM the fields marked as PII
- Never log passwords, tokens, or credit card data

### A03 — Injection

**SQL:**
```typescript
// ❌ BAD — direct concatenation
const user = await db.query(`SELECT * FROM users WHERE id = ${userId}`);

// ✅ GOOD — parameterized query
const user = await db.query('SELECT * FROM users WHERE id = $1', [userId]);

// ✅ GOOD — ORM with typed parameters
const user = await userRepository.findOne({ where: { id: userId } });
```

**Rules:**
- Parameterized queries ALWAYS. Zero concatenated strings in SQL
- Validate and sanitize all inputs with a validation library (Zod, Joi, class-validator)
- In GraphQL: limit query depth with `graphql-depth-limit`

### A04 — Insecure Design

- Every HU that exposes user data must undergo privacy review
- Bulk query endpoints have mandatory pagination (maximum [100] records per page)
- Do not expose sequential internal IDs; use UUIDs

### A05 — Security Misconfiguration

```
# Verification checklist per environment
□ Stack traces NOT visible in production
□ Security headers configured (Helmet.js or equivalent):
  - X-Content-Type-Options: nosniff
  - X-Frame-Options: DENY
  - Content-Security-Policy defined
  - Strict-Transport-Security in production
□ Unnecessary ports closed
□ Development credentials NOT in production
```

### A06 — Vulnerable Components

**Rules:**
- Run `npm audit` (or equivalent) before each release
- **Critical/High** vulnerabilities block the deploy
- Renew dependencies each sprint (at least once)
- Do not use `latest` versions without pinning in `package.json`; use exact versions or conservative ranges

### A07 — Identification and Authentication Failures

- JWT with maximum expiration of **1 hour** for access tokens
- Refresh tokens with expiration of **[7 days / 30 days]** and rotation on each use
- Rate limiting on `/auth/login`: maximum [10] attempts per IP in 5 minutes
- Account lockout after [5] consecutive failed attempts

### A08 — Software and Data Integrity Failures

- Verify Docker image checksum before using in production
- Third-party webhooks must verify cryptographic signature
- Validate that messages from the broker (Kafka/RabbitMQ) have the expected schema

### A09 — Security Logging and Monitoring Failures

- Every failed authentication must be logged with IP, timestamp, and user-agent
- Log delete operations with who, when, and what was deleted
- Security logs are retained for a minimum of **90 days**
- Automatic alerts configured for:
  - More than [50] 401/403 errors in 5 minutes
  - Access to a resource from an unexpected country (if applicable)

### A10 — Server-Side Request Forgery (SSRF)

- URLs constructed from user input MUST be validated against an allowlist of permitted domains
- Do not fetch from private IPs (192.168.x.x, 10.x.x.x, 127.x.x.x) from the server

---

## User input handling

```typescript
// Example with Zod — always validate in the Controller/Adapter layer
const CreateUserSchema = z.object({
  email: z.string().email().max(255),
  name: z.string().min(1).max(100).trim(),
  role: z.enum(['ADMIN', 'USER', 'VIEWER']),
});

// The result is typed and sanitized
const parsed = CreateUserSchema.parse(req.body);
```

**Rule:** All external inputs (HTTP body, query params, path params, broker messages)
pass through a validation schema before reaching the domain.

---

## Secure error handling

```typescript
// ❌ BAD — exposes internal details
res.status(500).json({ error: error.message, stack: error.stack });

// ✅ GOOD — generic message + traceId for internal correlation
res.status(500).json({
  error: 'INTERNAL_SERVER_ERROR',
  message: 'Internal server error',
  traceId: req.headers['x-trace-id'],
});
```

---

## Correlations

- Security policy (management, access, vault) → `00-governance/security-policy.md`
- System threat model → `05-architecture/security-threat-model.md`
- Authentication and JWT → `07-api/authentication.md`
- Observability and security logs → `13-operations/observability.md`
