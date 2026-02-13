# CONDITIONAL Generator - Logic-Based Values

## What It Does

Uses if-then-else logic to choose different generators based on conditions. Most complex generator type - implements branching logic.

---

## When to Use CONDITIONAL

Use CONDITIONAL when:
- Value depends on another field
- Different values for different scenarios
- Optional fields based on conditions
- Status-based variations
- Type-based routing

**Common Scenarios:**
- Different values based on ticket status
- Assign teams based on ticket type
- Escalation based on priority
- Optional fields based on conditions
- Complex nested branching logic

---
## When to Use CONDITIONAL

| Scenario | Use CONDITIONAL | Why |
|----------|---|---|
| Value depends on field | Yes | Logical branching |
| Multiple scenarios | Yes | More concise |
| Optional field | Yes | Clearer than null |
| Fixed value | No | Use STATIC |
| API fetch | No | Use DYNAMIC |
| Timestamp | No | Use REMOTE |
| Request data | No | Use REFERENCE |

---

## Reference

- **Fetch Method:** Conditional logic
- **Data Type:** Any (from branch generator)
- **Scope:** Depends on condition
- **Updates:** Dynamic based on condition

---

*Last Updated: 11 February 2026*
