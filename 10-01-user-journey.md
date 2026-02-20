---
title: "User Journey & Data Taxonomy"
parent: Algorithm Reference
nav_order: 1
---

## Section 1: The User Journey & Data Taxonomy

The Alter journey begins with three primary data capture events during onboarding. All three are required before the agent activates.

### 1.1 The Three Inputs

| Input | What we capture | Stored as |
|---|---|---|
| **10 Photos** | Visual presentation, aesthetic markers, body language, vibe | JPEG in Supabase Storage → signed URLs |
| **Audio Interview** | Personality, tone, communication style, life context | Audio file → transcript → structured fields |
| **Birth Details** | Date, time, and place of birth | `dob`, `tob`, `lob` on `profiles` |

### 1.2 The Transformation Pipeline

Raw inputs flow through a multi-stage transformation before the agent can use them:

```
Audio ──► Whisper-1 ──► GPT-4o-mini ──► profiles (name, dob, vibe_summary, occupation…)
                                               │
Photos ──► Gemini 2.0 Flash ──────────────► profiles.profile_analysis
           (multimodal vision)                 │
                                               ▼
                                    gemini-embedding-001
                                    (3072-dim vector)
                                               │
                                               ▼
                                    profiles.agent_embedding
```

After both pipelines complete, the **Dossier Generator** synthesises everything into `profiles.agent_dossier` — the unified identity representation that drives all downstream matching.

---
