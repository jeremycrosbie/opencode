---
name: owasp-review
description: Use when reviewing changed code for OWASP-style web application security issues and producing structured security findings.
---

# OWASP Review

Use this skill when reviewing changed code for common web application security issues, as defined in https://owasp.org/Top10/2025/

Checklist:
- Validate trust boundaries and external inputs
- Verify authn/authz on sensitive actions
- Check for injection risks in database, shell, template, and query construction
- Verify secrets and tokens are not exposed or logged
- Check output encoding where user-controlled content is rendered
- Check file upload, download, and path handling
- Check abuse controls where relevant, such as rate limits or replay concerns
- Check for sensitive data leakage in logs or error messages
- Check security-relevant defaults and config exposure

Output format:
- Severity
- Finding
- Affected file or area
- Recommendation
- OWASP mapping
