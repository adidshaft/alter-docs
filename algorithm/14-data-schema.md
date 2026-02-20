---
title: "Data Schema Reference"
parent: Algorithm
nav_order: 14
---

## Section 14: Data Schema Reference

### Key Tables

**`profiles`** — Core user record
- `agent_embedding` (vector 3072) — cosine similarity input
- `agent_dossier` (jsonb) — synthesised identity card
- `profile_analysis` (jsonb) — Gemini visual analysis output
- `voice_interview_done` (bool) — eligibility gate
- `dob`, `tob`, `lob` — Vedic astrology inputs
- `vibe_summary`, `occupation` — interview extraction outputs

**`agent_constraints`** — Learned hard preferences
- `constraint_type`: `negative` | `positive`
- `label`: snake_case reason chip (e.g., `dislikes_ethereal_vibe`)
- Powers the penalty system (Section 8)

**`agent_scouting_logs`** — Phase 12 live feed
- `action_type`: `rejected` | `shortlisted` | `analyzing`
- `log_message`: AI-narrated scouting update
- Published to Supabase Realtime

**`matches`** — Match records
- `score` (float) — final penalised score
- `negotiation_transcript` (text) — agent debate output; injected into chat
- `status`: `pending` | `accepted` | `rejected`

**`match_feedback`** — RL training signal
- `rating`: 1 (thumbs-down) | 5 (thumbs-up)
- `agent_action`: `accepted` | `rejected`

**`model_training_logs`** — Long-term learning corpus
- Every interaction signal for future fine-tuning

**`messages`** — Chat messages
- `vibe_check`, `vibe_score`, `vibe_reason`, `is_flagged` — safety layer output

