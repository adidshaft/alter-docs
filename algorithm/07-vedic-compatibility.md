---
title: "Vedic Compatibility Scoring"
parent: Algorithm Reference
nav_order: 7
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

