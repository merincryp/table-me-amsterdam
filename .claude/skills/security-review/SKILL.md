---
name: security-review
description: Auto-invoked security review for the website
trigger: When modifying forms, external links, scripts, or adding third-party resources
---

# Security Review

## Checklist
1. **XSS Prevention**: No `innerHTML` with user input, sanitize all dynamic content
2. **External Links**: All have `rel="noopener noreferrer"`
3. **CSP Headers**: Content Security Policy configured in `vercel.json` if needed
4. **HTTPS**: All external resources loaded over HTTPS
5. **No Secrets**: No API keys, tokens, or credentials in code
6. **Form Security**: Honeypot fields, rate limiting awareness
7. **Dependencies**: No untrusted third-party scripts
8. **Subresource Integrity**: SRI hashes for external scripts/styles when possible
