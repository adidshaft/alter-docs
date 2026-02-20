---
title: "The Constraint Penalty System"
parent: Algorithm Reference
nav_order: 8
---

## Section 8: The Constraint Penalty System

The penalty system is the algorithm's mechanism for respecting hard preferences learned from user behaviour.

### 8.1 Penalty Values

| Constraint Type | Penalty | Trigger |
|---|---|---|
| Vibe-specific mismatch | **25%** | Thumbs-down on a specific vibe chip (e.g., "disliked Ethereal Vibe") |
| General type mismatch | **10%** | Thumbs-down without a specific vibe chip (`general_type_mismatch`) |

### 8.2 Caps and Floor

```
total_penalty = min(0.30, sum_of_applicable_penalties)
penalised_score = max(0.55, final_score × (1 − total_penalty))
```

- **Maximum penalty cap**: 30% — prevents a single constraint from being catastrophic
- **Score floor**: 0.55 — keeps every match above theoretical noise threshold

### 8.3 Constraint Storage

Constraints are stored in the `agent_constraints` table:

```sql
constraint_type: 'negative'
label:          'dislikes_ethereal_vibe'   -- snake_case
```

Each label is generated deterministically from the feedback reason chip. Multiple constraints compound (up to the 30% cap).

