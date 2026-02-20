---
title: "The Mathematics of Matching"
parent: Algorithm
nav_order: 6
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

