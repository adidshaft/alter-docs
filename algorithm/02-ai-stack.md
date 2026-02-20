---
title: "The AI Stack"
parent: Algorithm
nav_order: 2
---

## Section 2: The AI Stack

### 2.1 Model Inventory

| Model | Provider | Role in Alter |
|---|---|---|
| `whisper-1` | OpenAI | Audio-to-text transcription in the voice interview |
| `gpt-4o-mini` | OpenAI | Structured data extraction from interview transcripts |
| `gemini-2.0-flash` | Google | Visual profile analysis, dossier generation, scouting narration, negotiation transcripts, vibe-check safety |
| `gemini-embedding-001` | Google | 3072-dimensional embedding vectors for cosine similarity matching |

> **Why two providers?** Whisper + GPT-4o-mini give the highest-quality structured extraction from audio. Gemini's multimodal capabilities outperform all alternatives for interpreting aesthetic cues from photos. The `gemini-embedding-001` model produces the richest semantic vectors for dating-specific vibe matching.

### 2.2 Edge Function Inventory

All backend logic runs as Supabase Edge Functions (Deno runtime):

| Function | Trigger | Model used |
|---|---|---|
| `audio-interviewer` | User completes interview | Whisper-1 → GPT-4o-mini |
| `analyze-profile` | Photo upload complete | Gemini 2.0 Flash (multimodal) |
| `update-dossier` | Post-onboarding or admin call | Gemini 2.0 Flash |
| `generate-scouting-logs` | pg_cron (3× daily) | Gemini 2.0 Flash |
| `vibe-check` | Every message sent | Gemini 2.0 Flash |
| `update-agent-weights` | User feedback (thumbs up/down) | Rule-based (no LLM) |

The **matchmaker** runs as an AWS Lambda (`alter-matchmaker`) triggered by EventBridge at `cron(11 23 * * ? *)` — 11:11 PM UTC nightly.

