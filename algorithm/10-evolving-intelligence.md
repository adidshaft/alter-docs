---
title: "The Evolving Intelligence (RL Loop)"
parent: Algorithm Reference
nav_order: 10
---

## Section 10: The Evolving Intelligence (RL Loop)

Alter gets smarter with every interaction. The RL loop turns user behaviour into agent constraints that sharpen future matching.

### 10.1 Feedback Collection

Every match outcome is recorded via `update-agent-weights`. Two feedback pathways:

**Thumbs Up (Accepted)**
- Writes to `match_feedback`: `rating = 5`, `agent_action = "accepted"`
- No constraint changes — positive signal reinforces existing strategy

**Thumbs Down (Rejected)**
- Writes to `match_feedback`: `rating = 1`, `agent_action = "rejected"`
- Triggers Reason Chip analysis
- Creates a new `agent_constraints` record

### 10.2 Reason Chips

When a user thumbs-down a match, they select a reason chip:
- **Beauty** → vibe/aesthetic mismatch → creates `dislikes_<vibe>_vibe` constraint (25% penalty)
- **Vibe** → same as Beauty path
- **Type** → creates `general_type_mismatch` constraint (10% penalty)

The chip label is converted to a snake_case constraint label that the matchmaker evaluates on every future candidate pair involving that user.

### 10.3 Dossier Refresh

When a user accumulates new constraints, their `agent_dossier` is regenerated to reflect the updated dealbreakers. The new dossier's `dealbreakers_learned` array incorporates the top 5 most-recent negative constraints, creating a feedback loop where the dossier accurately represents the agent's evolved taste.

### 10.4 `model_training_logs`

All interactions (viewed, messaged, ignored, exited early) are recorded in `model_training_logs`. This dataset is the foundation for future supervised fine-tuning of the embedding model — training the vector space itself to better reflect what "compatible" means for Alter's specific user population.

