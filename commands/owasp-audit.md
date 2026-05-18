---
description: Audit the current changes for security issues
agent: owasp-reviewer
subtask: true
---

Audit the current work for security issues defined by OWASP.

Review:
- the current diff
- nearby auth/authz code paths
- input validation
- secrets handling
- logging
- file handling
- injection risks
- unsafe configuration

Use the OWASP review skill if available.

Return:
## Critical Issues
## Warnings
## Suggestions

For each finding include:
- severity
- file or area
- issue
- remediation
- OWASP mapping where applicable

Do not modify files.
