# MCP Security Checklist

> Production-ready security audit checklist for MCP server deployments.

---

## Pre-Deployment Checklist

### Server Configuration
- [ ] All servers installed from official/verified sources
- [ ] Server versions pinned (no auto-updates without review)
- [ ] Package integrity verified (checksums/signatures)
- [ ] No unnecessary servers connected
- [ ] Each server follows single responsibility principle

### Authentication & Secrets
- [ ] No secrets hard-coded in configuration files
- [ ] API keys stored in environment variables or OS keychain
- [ ] OAuth 2.1 with PKCE enabled for all remote servers
- [ ] Access tokens have short expiry (≤1 hour)
- [ ] Refresh tokens implement rotation
- [ ] All secrets excluded from version control (`.gitignore`)

### Network & Transport
- [ ] TLS enabled for all remote connections
- [ ] Certificate validation not disabled
- [ ] Local servers use stdio (not exposed to network)
- [ ] No servers listening on 0.0.0.0 unnecessarily
- [ ] Firewall rules restrict MCP server ports

### Input Validation
- [ ] All tool parameters validated before processing
- [ ] SQL injection prevention (parameterized queries)
- [ ] Path traversal prevention (canonicalize file paths)
- [ ] Command injection prevention (no shell interpolation)
- [ ] File type/size limits enforced

### Output Security
- [ ] Large responses truncated with pagination
- [ ] No sensitive data in error messages
- [ ] Logging goes to stderr (not stdout for stdio)
- [ ] No tokens or credentials in log output
- [ ] Response content filtered for injection attempts

### Permission Model
- [ ] Least privilege: each server has minimum necessary access
- [ ] Destructive operations require explicit user consent
- [ ] Read operations separated from write operations
- [ ] Resource access scoped to relevant directories/databases
- [ ] Roots configured to limit filesystem access

---

## Runtime Monitoring

### Audit Logging
- [ ] All tool invocations logged with timestamp
- [ ] Tool parameters logged (sanitized of secrets)
- [ ] Response sizes tracked
- [ ] Error rates monitored
- [ ] Unusual patterns trigger alerts

### Rate Limiting
- [ ] Per-server request rate limits configured
- [ ] Per-tool invocation limits set
- [ ] Bulk/destructive operations throttled
- [ ] Circuit breakers for failing servers

---

## Enterprise Additional Checks

### Gateway & Proxy
- [ ] MCP gateway deployed for centralized auth
- [ ] Request routing validated
- [ ] Cross-server communication blocked
- [ ] Admin audit trail enabled

### Compliance
- [ ] Data classification labels applied
- [ ] PII handling rules enforced
- [ ] Data retention policies configured
- [ ] Incident response plan includes MCP scenarios

---

## Quick Severity Guide

| Issue Found | Severity | Action |
|-------------|----------|--------|
| Hard-coded secret | 🔴 Critical | Remove immediately, rotate credential |
| No TLS on remote | 🔴 Critical | Enable TLS before deployment |
| No input validation | 🟠 High | Add validation before production |
| No user consent for writes | 🟠 High | Implement consent flow |
| Verbose error messages | 🟡 Medium | Sanitize error output |
| No audit logging | 🟡 Medium | Add logging before production |
| Missing rate limits | 🟢 Low | Add for production scale |
