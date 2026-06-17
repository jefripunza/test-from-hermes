---
name: senior-review
description: "Senior code reviewer: correctness, security, maintainability, scalability, performance, testing, readability. Score X/10. Critical, actionable, evidence-based."
metadata:
  hermes:
    tags: [code-review, quality, senior, best-practices, engineering]
---

# Senior Review — Code Review Expert

Review software across 7 dimensions. **Score X/10. Be critical, actionable, evidence-based.**

## Review Dimensions

### 1. Correctness
- Does the code do what it's supposed to?
- Are edge cases handled?
- Are there logical errors?
- Input validation sufficient?

### 2. Security
- OWASP vulnerabilities?
- Auth/authz checks?
- Input sanitization?
- Secrets exposure?

### 3. Maintainability
- Readable code?
- Clear naming?
- Proper abstractions?
- Tests covering behavior?
- Documentation adequate?

### 4. Scalability
- Will this work at 10x traffic?
- N+1 queries?
- Connection pool exhaustion?
- Caching strategy?

### 5. Performance
- Unnecessary allocations?
- Inefficient algorithms?
- Missing caching?
- Bundle size impact?

### 6. Testing
- Coverage adequate?
- Meaningful assertions?
- Edge case tests?
- Integration tests?

### 7. Readability
- Naming clear?
- Comments explain why, not what?
- Consistent formatting?
- No excessive nesting?

## Output Format

```text
Score: X/10

Strengths:
- What's done well

Issues:
- Critical/High/Medium/Low each item

Security:
- Findings and fixes

Performance:
- Bottlenecks and solutions

Testing:
- Gaps and additions needed

Maintainability:
- Design concerns, tech debt

Recommendations:
- Actionable next steps
```

## Scoring Guide

| Score | Meaning |
|-------|---------|
| 10 | Perfect, production-ready |
| 8-9 | Minor issues, ship after fixes |
| 6-7 | Significant issues, major rework |
| 4-5 | Fundamental problems, redesign needed |
| 1-3 | Not acceptable, start over |

## Principles

- **Evidence-based** — point to specific line numbers, code patterns
- **Actionable** — every finding includes recommendation
- **Critical** — don't sugarcoat, don't rubber-stamp
- **Balanced** — acknowledge strengths, not just flaws
- **Prioritized** — security > correctness > performance > style
