---
title: "The Scouting Feed (Phase 12)"
parent: Algorithm
nav_order: 11
---

## Section 11: The Scouting Feed (Phase 12)

The scouting feed makes the agent visible. Instead of a black-box "Your agent is working" state, users see a live terminal feed of the agent's internal monologue.

### 11.1 Architecture

```
pg_cron (12:00, 16:00, 20:00 UTC)
    │
    ▼
generate-scouting-logs (Edge Function)
    │
    ├── Fetch eligible users (voice_interview_done + agent_dossier not null)
    │
    └── For each user:
        ├── Fetch negative constraints
        ├── Call Gemini 2.0 Flash (temp: 0.85, max: 512 tokens)
        └── Insert 2 rows into agent_scouting_logs
```

### 11.2 Log Types

| `action_type` | Icon | Colour | Meaning |
|---|---|---|---|
| `rejected` | `xmark.circle` | Red | Agent dismissed a candidate |
| `analyzing` | `magnifyingglass` | Amber | Agent is evaluating a candidate |
| `shortlisted` | `checkmark.circle` | Green | Agent has flagged a high-potential candidate |

### 11.3 Gemini Scouting Prompt Design

The scouting logs are generated in **mission-log style** — first-person, terse, past-tense, quantitative:

> "Reviewed 14 profiles against dark-academia aesthetic filter. Disqualified 13 on vibe mismatch. One candidate — 87% Vedic alignment, introspective archetype — flagged for deeper review."

Rules enforced in the prompt:
- Must cite specific numbers
- Must reference the user's dossier subtly (not verbatim)
- No emojis in log text
- Maximum 1–2 sentences per entry
- One negative entry + one positive-leaning entry per batch

### 11.4 Realtime Delivery

`agent_scouting_logs` is published to `supabase_realtime`. The iOS app subscribes on channel `scouting:<userId>`. New entries appear in the terminal feed with a letter-by-letter typing animation (`TypingTextModifier`, 1.5s duration).

