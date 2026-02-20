---
title: "Pipeline Stage 2 — Visual Processing"
parent: Algorithm Reference
nav_order: 4
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

