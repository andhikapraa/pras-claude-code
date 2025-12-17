---
description: Perform comprehensive security audit with dependency scanning and OWASP checks
model: claude-opus-4-5
---

Perform a comprehensive security audit on the codebase or specified area.

## Audit Target

$ARGUMENTS

---

## MCP Tools Integration

**IMPORTANT:** Use these tools for thorough analysis:

### Required Tools
1. **Sequential Thinking** - Systematically analyze security concerns
2. **Context7** - Fetch latest security best practices for frameworks used

### Optional Tools (if available)
3. **Exa/Firecrawl** - Research recent vulnerabilities and CVEs

---

## Security Audit Phases

### Phase 1: Dependency Audit

**Run dependency scanner based on package manager:**

```bash
# pnpm (preferred)
pnpm audit
pnpm audit --fix  # Auto-fix where possible

# bun
bun audit
bun pm update    # Update vulnerable packages

# npm
npm audit
npm audit fix

# yarn
yarn audit
yarn upgrade-interactive
```

**Check for outdated packages:**
```bash
pnpm outdated
npx npm-check-updates
```

**Analyze output for:**
- Critical vulnerabilities (fix immediately)
- High severity (fix within sprint)
- Moderate (schedule fix)
- Low (monitor)

### Phase 2: Secret Scanning

**Files to check:**
```
.env*                    # Environment files
*.config.js/ts           # Config files
src/**/*.ts              # Source code
docker-compose*.yml      # Docker configs
```

**Patterns to detect:**
```regex
# API Keys
[A-Za-z0-9_]{20,}

# AWS
AKIA[0-9A-Z]{16}
aws_secret_access_key

# Database URLs
postgres://
mongodb://
mysql://

# Private Keys
-----BEGIN (RSA |EC |OPENSSH )?PRIVATE KEY-----

# Tokens
token['\"]?\s*[:=]\s*['\"][^'\"]+
secret['\"]?\s*[:=]\s*['\"][^'\"]+
password['\"]?\s*[:=]\s*['\"][^'\"]+
```

**Recommended: Install git-secrets or gitleaks**
```bash
# gitleaks
gitleaks detect --source . --verbose

# git-secrets
git secrets --scan
```

### Phase 3: OWASP Top 10 Review

#### 1. Injection (SQL, NoSQL, Command)
```typescript
// BAD: SQL injection vulnerability
const query = `SELECT * FROM users WHERE id = ${userId}`;

// GOOD: Parameterized query
const { data } = await supabase
  .from('users')
  .select('*')
  .eq('id', userId);
```

#### 2. Broken Authentication
- Check password hashing (bcrypt, argon2)
- Verify session management
- Review token expiration
- Check MFA implementation

#### 3. Sensitive Data Exposure
- Verify HTTPS everywhere
- Check data encryption at rest
- Review logging (no sensitive data)
- Validate API responses (no extra fields)

#### 4. XML External Entities (XXE)
- Disable XML external entity processing
- Use JSON instead of XML where possible

#### 5. Broken Access Control
```typescript
// Check authorization on every request
// BAD
app.get('/api/user/:id', async (req, res) => {
  const user = await getUser(req.params.id);
  return res.json(user);
});

// GOOD
app.get('/api/user/:id', authMiddleware, async (req, res) => {
  if (req.user.id !== req.params.id && !req.user.isAdmin) {
    return res.status(403).json({ error: 'Forbidden' });
  }
  const user = await getUser(req.params.id);
  return res.json(user);
});
```

#### 6. Security Misconfiguration
- Check security headers (CSP, HSTS, X-Frame-Options)
- Review CORS configuration
- Verify error handling (no stack traces in production)
- Check default credentials

#### 7. Cross-Site Scripting (XSS)
```typescript
// BAD: Direct HTML rendering
<div dangerouslySetInnerHTML={{ __html: userInput }} />

// GOOD: Use sanitization
import DOMPurify from 'dompurify';
<div dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(userInput) }} />

// BEST: Avoid dangerouslySetInnerHTML entirely
<div>{userInput}</div>
```

#### 8. Insecure Deserialization
- Validate and sanitize all input
- Use schema validation (Zod, Yup)

#### 9. Using Components with Known Vulnerabilities
- Regular dependency audits
- Automated security updates (Dependabot, Renovate)

#### 10. Insufficient Logging & Monitoring
- Log security events
- Set up alerting
- Monitor for anomalies

### Phase 4: API Security

**Check for:**
- Rate limiting
- Input validation
- Output encoding
- Authentication on all endpoints
- Authorization checks
- CORS configuration

```typescript
// Rate limiting example (Next.js)
import { Ratelimit } from '@upstash/ratelimit';
import { Redis } from '@upstash/redis';

const ratelimit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(10, '10 s'),
});

export async function middleware(request: NextRequest) {
  const ip = request.ip ?? '127.0.0.1';
  const { success } = await ratelimit.limit(ip);

  if (!success) {
    return new Response('Too Many Requests', { status: 429 });
  }
}
```

### Phase 5: Infrastructure Security

**Check:**
- Environment variable handling
- Docker security (non-root user, minimal base image)
- Cloud permissions (least privilege)
- Network security (firewalls, VPCs)

---

## Output Format

### Security Audit Report

```markdown
# Security Audit Report
Date: [Date]
Scope: [What was audited]

## Executive Summary
- Critical: X issues
- High: X issues
- Medium: X issues
- Low: X issues

## Dependency Audit Results
| Package | Current | Vulnerability | Severity | Fix |
|---------|---------|--------------|----------|-----|
| lodash  | 4.17.19 | Prototype Pollution | High | 4.17.21 |

## Code Security Issues

### Critical

#### [Issue Title]
- **Location**: `src/api/users.ts:45`
- **Description**: SQL injection vulnerability
- **Impact**: Database compromise
- **Remediation**: Use parameterized queries
- **Code Fix**:
  ```typescript
  // Before
  ...
  // After
  ...
  ```

### High
...

### Medium
...

### Low
...

## Recommendations

### Immediate Actions (Critical)
1. [ ] Fix SQL injection in users API
2. [ ] Update vulnerable dependencies

### Short-term (This Sprint)
1. [ ] Implement rate limiting
2. [ ] Add security headers

### Long-term (Roadmap)
1. [ ] Set up security monitoring
2. [ ] Implement WAF

## Security Headers Checklist
- [ ] Content-Security-Policy
- [ ] X-Frame-Options
- [ ] X-Content-Type-Options
- [ ] Strict-Transport-Security
- [ ] X-XSS-Protection (legacy)
- [ ] Referrer-Policy

## Compliance Status
- [ ] OWASP Top 10 addressed
- [ ] No hardcoded secrets
- [ ] Dependencies up to date
- [ ] Logging implemented
```

---

## Best Practices

**DO:**
- Run audits regularly (weekly minimum)
- Automate dependency updates
- Use security linters (eslint-plugin-security)
- Implement defense in depth
- Keep audit logs

**DON'T:**
- Ignore low severity issues
- Commit secrets to git
- Trust client-side validation alone
- Use deprecated crypto algorithms
- Skip security in development

---

Generate a thorough security audit report with actionable recommendations.

---
*Originally created by [Pras](https://github.com/andhikapraa/pras-claude-code)*
