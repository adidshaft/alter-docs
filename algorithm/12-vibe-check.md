---
title: "The Vibe-Check Safety Layer"
parent: Algorithm Reference
nav_order: 12
---

## Section 12: The Vibe-Check Safety Layer

Every message sent through Alter is asynchronously evaluated by the `vibe-check` edge function.

### 12.1 Classification Schema

| Class | `vibe_score` range | Meaning |
|---|---|---|
| `clean` | 0.9–1.0 | Normal, respectful conversation |
| `ghosting_risk` | 0.5–0.8 | Signals of impending disengagement |
| `flagged` | 0.5–0.8 | Ambiguous — passive aggression, boundary-pushing |
| `toxic` | 0.0–0.4 | Harassment, threats, hate speech, manipulation |

### 12.2 Processing

- Model: **Gemini 2.0 Flash** (temperature 0.1 — deterministic for safety)
- Called asynchronously after message delivery (does not block send)
- Results written back to `messages`: `vibe_check`, `vibe_score`, `vibe_reason`, `is_flagged`, `flagged_at`
- `is_flagged = true` for `toxic` and `flagged` classes
- Fail-open: if the AI call errors, the message defaults to `clean` (never block on AI failure)

### 12.3 Ghosting Detection

The `ghosting_risk` class catches low-intent disengagement patterns early (e.g., "whatever", "I don't care", one-word responses) so the system can surface the right nudge at the right time.

