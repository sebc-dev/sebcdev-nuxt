# Technical Debt Register

> Source canonique des améliorations techniques nécessaires.
> Distinct du backlog fonctionnel.

## Summary

| Priority | Count | Estimated Effort |
|----------|-------|------------------|
| Critical | 0 | 0 days |
| High | 0 | 0 days |
| Medium | 0 | 0 days |
| Low | 0 | 0 days |

**Technical Debt Ratio Target:** < 5%

---

## TD-001: [Title]

| Attribute | Value |
|-----------|-------|
| **Status** | Open / In Progress / Resolved |
| **Priority** | Critical / High / Medium / Low |
| **Type** | Performance / Security / Maintainability / Architecture |
| **Effort** | Low (< 1 day) / Medium (2-3 days) / High (> 3 days) |
| **Owner** | @username |
| **Issue** | [#XXX](link) |
| **Created** | YYYY-MM-DD |

**Description:**
Brief description of the technical debt item.

**Impact:**
- Bullet points describing the negative impact
- On performance, maintainability, security, or developer experience

**Proposed Solution:**
Description of the recommended fix or improvement.

**Acceptance Criteria:**
- [ ] Criterion 1
- [ ] Criterion 2

---

## Prioritization Matrix

```
            │ Low Effort    │ High Effort
────────────┼───────────────┼─────────────────
High Impact │ QUICK WINS    │ BIG BETS
            │ Do immediately│ Plan in sprints
────────────┼───────────────┼─────────────────
Low Impact  │ FILL-INS      │ MONEY PITS
            │ When time     │ Defer or drop
            │ permits       │
```

## Types of Technical Debt

| Type | Description | Examples |
|------|-------------|----------|
| **Performance** | Code that degrades speed or efficiency | Missing caching, N+1 queries |
| **Security** | Vulnerabilities or weak practices | Outdated deps, missing validation |
| **Maintainability** | Code that's hard to understand/modify | No tests, poor naming, duplication |
| **Architecture** | Structural issues limiting evolution | Tight coupling, wrong patterns |

## Process

1. **Detection:** Continuous via ESLint, tests, code review
2. **Documentation:** Add entry to this file + create GitHub issue
3. **Prioritization:** Use Impact/Effort matrix
4. **Resolution:** Allocate 15-20% of sprint capacity
5. **Review:** Quarterly review with stakeholders
