---
title: "Pipeline Stage 1 — Audio Processing"
parent: Algorithm Reference
nav_order: 3
---

## Section 3: Pipeline Stage 1 — Audio Processing

### 3.1 Transcription (`audio-interviewer`)

The voice interview is sent to **OpenAI Whisper-1** for transcription. The raw transcript is never stored — it is immediately fed to the extraction step.

### 3.2 Structured Extraction

The transcript is passed to **GPT-4o-mini** with a structured extraction prompt. The model outputs the following fields, which are written directly to `profiles`:

| Field | Description |
|---|---|
| `full_name` | User's preferred name |
| `dob` | Date of birth (YYYY-MM-DD) |
| `tob` | Time of birth (HH:MM, 24h) |
| `lob` | City/town of birth |
| `timezone` | User's current timezone |
| `occupation` | Current profession |
| `vibe_summary` | 2–3 sentence free-form personality summary in the user's own words |

On success, `profiles.voice_interview_done` is set to `true`, which gates the agent's eligibility for matching.

