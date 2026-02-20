---
title: "Contributing to the Algorithm"
parent: Algorithm
nav_order: 15
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
