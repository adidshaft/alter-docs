---
title: "Matchmaking Architecture"
description: "Complete technical reference — every model, weight, formula, and data table powering Alter's AI matchmaking."
---

# Alter Matchmaking Architecture
### A complete technical reference for engineers, researchers, and contributors

> **Philosophy**: Alter replaces swiping with autonomous agents. Your agent learns your taste, scouts the ecosystem, negotiates with other agents, and only brings you matches worth your attention. This document explains exactly how that works — every model, every weight, every formula.

---

## Table of Contents

1. [The User Journey & Data Taxonomy](#section-1-the-user-journey--data-taxonomy)
2. [The AI Stack](#section-2-the-ai-stack)
3. [Pipeline Stage 1: Audio Processing](#section-3-pipeline-stage-1-audio-processing)
4. [Pipeline Stage 2: Visual Processing](#section-4-pipeline-stage-2-visual-processing)
5. [Pipeline Stage 3: The Agent Dossier](#section-5-pipeline-stage-3-the-agent-dossier)
6. [The Mathematics of Matching](#section-6-the-mathematics-of-matching)
7. [Vedic Compatibility Scoring](#section-7-vedic-compatibility-scoring)
8. [The Constraint Penalty System](#section-8-the-constraint-penalty-system)
9. [The Agent Negotiation Protocol](#section-9-the-agent-negotiation-protocol)
10. [The Evolving Intelligence (RL Loop)](#section-10-the-evolving-intelligence-rl-loop)
11. [The Scouting Feed (Phase 12)](#section-11-the-scouting-feed-phase-12)
12. [The Vibe-Check Safety Layer](#section-12-the-vibe-check-safety-layer)
13. [Infrastructure & Scheduling](#section-13-infrastructure--scheduling)
14. [Data Schema Reference](#section-14-data-schema-reference)
15. [Contributing to the Algorithm](#section-15-contributing-to-the-algorithm)

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

---

## Section 4: Pipeline Stage 2 — Visual Processing

### 4.1 Multimodal Analysis (`analyze-profile`)

The 10 uploaded photos are passed to **Gemini 2.0 Flash** as multimodal inputs alongside a structured analysis prompt. The output is stored as `profiles.profile_analysis` (JSONB):

```json
{
  "aesthetic_vibe": "Dark Academia",
  "core_hobbies": ["reading", "café-hopping", "film photography"],
  "harmony_score": 78,
  "style_notes": "Consistent earth-tone wardrobe, books present in 4/10 photos"
}
```

- **`aesthetic_vibe`**: The dominant aesthetic archetype detected (e.g., "Cottagecore", "Streetwear", "Minimalist", "Dark Academia").
- **`harmony_score`**: A 0–100 score measuring visual coherence across all 10 photos. Higher scores indicate a clear, consistent self-presentation.
- **`core_hobbies`**: Activities and interests inferred from visual context clues.

### 4.2 Embedding Generation

After profile analysis, the enriched profile data (vibe summary + aesthetic + hobbies + occupation) is passed to **`gemini-embedding-001`** to produce a **3072-dimensional vector**. This is stored in `profiles.agent_embedding` and is the numerical backbone of all matching.

---

## Section 5: Pipeline Stage 3 — The Agent Dossier

### 5.1 What is the Dossier?

The `agent_dossier` is a Gemini-synthesised JSON identity card that consolidates everything known about a user into a single, human-readable + machine-readable object. It is generated by the `update-dossier` edge function.

### 5.2 Schema

```json
{
  "archetype_name": "The Quiet Architect",
  "archetype_emoji": "🏛️",
  "vibe_keywords": ["introspective", "precise", "warm"],
  "attracted_to": ["The Soulful Creator", "The Grounded Explorer"],
  "dealbreakers_learned": ["Avoids Superficial Conversations", "Avoids High Drama Energy"],
  "agent_strategy": "Prioritising depth-first matches — filtering for intellectual curiosity and measured emotional expressiveness before aesthetic alignment.",
  "astrological_compatibility_vibe": "Born under Capricorn energy with Mumbai's ambitious undertone, they seek a partner who balances ambition with tenderness — someone who plans for the future but knows how to savour the present.",
  "generated_at": "2025-11-14T23:11:00.000Z"
}
```

### 5.3 Generation Logic

1. Fetch `aesthetic_vibe`, `core_hobbies`, `harmony_score`, `occupation`, `vibe_summary`, `dob`, `lob` from `profiles`
2. Fetch up to 5 most-recent negative `agent_constraints` (see Section 8) and render as readable dealbreakers
3. Call Gemini 2.0 Flash (temperature 0.7, max 8192 tokens) with a structured prompt
4. Validate all 7 required fields are present; reject partial responses
5. Write to `profiles.agent_dossier`

The dossier is regenerated each time `update-dossier` is called — both at the end of onboarding and any time `agent_constraints` accumulate enough new signal.

---

## Section 6: The Mathematics of Matching

The matchmaker runs every night at **11:11 PM UTC** as an AWS Lambda function.

### 6.1 High-Level Flow

```
1. Fetch all eligible profiles
   (voice_interview_done = true AND agent_embedding IS NOT NULL)

2. Compute pairwise cosine similarity on 3072-dim agent_embedding vectors

3. Select top 20 pairs by vector score

4. For each pair: compute Vedic compatibility score

5. Apply constraint penalties

6. Compute final weighted score

7. Generate Gemini reasoning summaries for top 3 pairs

8. Generate negotiation transcript for pairs scoring > 0.80

9. Upsert results into matches table
```

### 6.2 Cosine Similarity

For two users **A** and **B** with embeddings **v_A** and **v_B** (each 3072-dimensional):

```
vector_score(A, B) = (v_A · v_B) / (|v_A| × |v_B|)
```

Range: [−1, 1], normalised to [0, 1] for scoring. This captures semantic proximity in latent personality space — users with similar aesthetic archetypes, communication styles, and life orientations cluster together.

### 6.3 Final Score Formula

```
final_score = (0.65 × vector_score) + (0.35 × vedic_score)
```

| Component | Weight | Rationale |
|---|---|---|
| Vector similarity | **65%** | Captures holistic personality + vibe alignment across all data signals |
| Vedic compatibility | **35%** | Integrates astrological and elemental compatibility layers |

After applying constraint penalties (Section 8):

```
penalised_score = max(0.55, final_score × (1 − total_penalty))
```

The floor of **0.55** ensures that even heavily penalised pairs retain a theoretical minimum score (preventing irreversible blacklisting from a single data point).

---

## Section 7: Vedic Compatibility Scoring

Alter uses classical Indian Vedic astrology as a structured compatibility layer. Birth details (date, time, place) are mapped using the `ephem` library to compute precise astronomical positions.

### 7.1 Vedic Sub-Score Formula

```
vedic_score = (0.50 × guna_milan_score)
            + (0.30 × nakshatra_score)
            + (0.20 × kuta_score)
```

| Sub-Component | Weight | Description |
|---|---|---|
| Guna Milan | **50%** | The classical Ashtakoot system (8 kutas, 36 total points) |
| Nakshatra Compatibility | **30%** | 27-lunar-mansion compatibility matrix |
| Kuta Points | **20%** | Rashi lord compatibility (planetary rulership of Moon signs) |

All three sub-scores are normalised to [0, 1] before weighting.

### 7.2 Guna Milan — The Ashtakoot System

The 8 kutas and their point values:

| Kuta | Max Points | What it measures |
|---|---|---|
| Varna | 1 | Spiritual evolution level |
| Vashya | 2 | Mutual attraction and control |
| Tara | 3 | Birth star compatibility and health |
| Yoni | 4 | Physical compatibility and intimacy |
| Graha Maitri | 5 | Intellectual compatibility and friendship |
| Gana | 6 | Temperament (Deva / Manav / Rakshasa) |
| Bhakoot | 7 | Emotional and financial compatibility |
| Nadi | 8 | Genetic / physiological compatibility |

**Maximum possible: 36 points.** Traditional threshold for compatibility: 18+ points (50%).

The system normalises the raw Guna score: `guna_milan_score = raw_points / 36`.

### 7.3 Nakshatra (Lunar Mansion) Compatibility

Each person's Moon Nakshatra is computed from their precise birth data. A 27×27 compatibility matrix maps all Nakshatra pair combinations to a compatibility score based on classical Taara chakra principles.

### 7.4 Kuta Points

The Moon signs (Rashis) of each user are used to identify ruling planets (Rashi lords). The inter-planetary relationship table (natural friendships, neutrality, enmity between planets) generates the final Kuta score.

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

---

## Section 9: The Agent Negotiation Protocol

Before any match reaches a user's screen, their agents must negotiate and agree.

### 9.1 Eligibility Gate

Only pairs with `final_score ≥ 0.80` after penalty application proceed to negotiation. This ensures only genuinely high-compatibility pairs incur the Gemini API cost for transcript generation.

### 9.2 The Debate

Two AI agents, each initialised with their respective user's `agent_dossier`, enter a simulated negotiation using Gemini 2.0 Flash. They evaluate:

- Shared vibe keywords and aesthetic resonance
- Mutual `attracted_to` archetypes (does Agent A's target match Agent B's archetype, and vice versa?)
- Potential dealbreaker conflicts
- Astrological compatibility vibe alignment

The negotiation explores friction points honestly — a good transcript surfaces tension, not just flattery.

### 9.3 The Human Icebreaker

The negotiation output is stored as `matches.negotiation_transcript`. When the match is surfaced to both users, the transcript is injected directly into their new chat room as the opening message. This serves as the **human icebreaker** — a highly personalised, agent-authored conversation starter that gives both users something specific and interesting to react to immediately.

> This is the single most differentiating UX feature of Alter: the moment of connection isn't a blank chat box, it's a window into how your agents saw each other.

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

---

## Section 13: Infrastructure & Scheduling

### 13.1 Nightly Matchmaker

| Component | Detail |
|---|---|
| Runtime | AWS Lambda (`alter-matchmaker`) |
| Schedule | EventBridge `cron(11 23 * * ? *)` = 11:11 PM UTC |
| Trigger | Nightly, no arguments |
| Output | Upserts into `matches` table |

### 13.2 Scouting Log Cron

Three pg_cron jobs call `generate-scouting-logs` via `pg_net.http_post`:

| Job name | Schedule (UTC) | Local time (IST) |
|---|---|---|
| `scouting-logs-noon` | 12:00 | 17:30 |
| `scouting-logs-afternoon` | 16:00 | 21:30 |
| `scouting-logs-evening` | 20:00 | 01:30 +1 |

### 13.3 Authentication Patterns

| Context | Auth method |
|---|---|
| User-triggered (onboarding, dossier refresh) | Bearer JWT (user's `access_token`) |
| Cron-triggered functions | `x-admin-secret` header |
| Matchmaker Lambda → Supabase | Service role key (env var) |
| iOS → Storage upload | Bearer JWT + `apikey` anon key |
| iOS → `analyze-profile` | Anon key (Edge Function uses service role internally) |

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

---

## Section 15: Contributing to the Algorithm

Alter's matching intelligence is an evolving system, and we welcome proposals from engineers, researchers, and data scientists. This section explains how to contribute.

### 15.1 What We're Looking For

We're particularly interested in proposals across these domains:

**Matching & Scoring**
- Alternative or additional embedding models for the personality vector
- New features to incorporate into the final score (behavioural signals, communication style vectors)
- Improvements to the Vedic scoring weights based on outcome data
- Graph-based or collaborative filtering approaches to complement cosine similarity

**Vibe Extraction**
- Better aesthetic archetype taxonomies (the current list was hand-crafted)
- Multi-modal fusion approaches (combining audio tone + visual + text in a single embedding)
- Cross-cultural compatibility of the current Vedic layer for non-South-Asian user bases

**Safety & Ethics**
- Bias audits of the embedding space (are certain demographics systematically clustered?)
- More nuanced ghosting-risk detection
- Privacy-preserving matching architectures (e.g., federated embeddings)

**Agent UX**
- Richer scouting log narrative styles
- Agent negotiation improvements — more structured debate formats
- Explanation quality for match reasoning

### 15.2 The RFC Process

All algorithmic changes go through a **Request for Comments (RFC)** process:

1. **Open a Discussion** in the GitHub repository under `Discussions > Algorithm RFC`
2. **Use the RFC template**:
   ```markdown
   ## Summary
   One paragraph describing the change.

   ## Motivation
   What problem does this solve? What outcome data or research supports it?

   ## Technical Proposal
   Specific model, weight, formula, or code changes. Include pseudocode or math.

   ## Expected Impact
   How would you measure success? What could go wrong?

   ## Alternatives Considered
   What else did you evaluate and why did you reject it?
   ```
3. **Discussion period**: minimum 7 days for community comment
4. **Implementation**: accepted RFCs get a tracking issue and are assigned a Phase number

### 15.3 Running the Algorithm Locally

The matchmaker can be run locally against a development Supabase instance:

```bash
# Clone the repo
git clone https://github.com/adidshaft/alter.git

# Install Python dependencies
cd backend/agent-service
pip install -r requirements.txt

# Set environment variables
export SUPABASE_URL=...
export SUPABASE_SERVICE_KEY=...
export GEMINI_API_KEY=...

# Run a single matchmaking cycle
python matchmaker.py
```

Edge Functions can be tested with the Supabase CLI:
```bash
supabase functions serve generate-scouting-logs --env-file .env.local
curl -X POST http://localhost:54321/functions/v1/generate-scouting-logs \
  -H "x-admin-secret: <your-local-secret>"
```

### 15.4 Key Invariants

Any proposal must preserve these invariants to be considered:

| Invariant | Reason |
|---|---|
| `voice_interview_done = true` gate | Audio data is the richest signal; matching without it degrades quality significantly |
| Score floor of 0.55 | Prevents irreversible blacklisting and preserves serendipity |
| Constraint penalty cap of 30% | A single data point should never fully determine a match outcome |
| Fail-open on vibe-check | Safety AI errors must never block message delivery |
| Per-user error isolation in batch jobs | One bad profile must never abort the entire nightly run |

### 15.5 Versioning

The algorithm is versioned by **Phase number**. The current phase is **Phase 12**. Each phase is documented in [CHANGELOG.md](CHANGELOG.md) (if it exists) with the specific changes made, the commit hash, and the date shipped.

---

*This document is maintained by the Alter engineering team and updated with each phase. Last updated: Phase 12 (Scouting Feed). If you find an inaccuracy, please open an issue or PR against this file.*
