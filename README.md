# hagen_pg - Planning Ground for Cross-Border Concept Marketplace

> **Purpose**: Documentation and data repository for the TikTok skit concept marketplace  
> **Related Codebase**: `hagen` (separate repository)  
> **Last Updated**: December 3, 2025

---

## What This Repository Contains

```
hagen_pg/
├── AI_HANDOFF.md                    # Original system documentation (170+ features)
├── AI_CONTEXT_HANDOFF.md            # Context for AI handoff
├── analysis_export_2025-12-03.json  # Training data export (51 videos)
├── analysis_export_2025-12-03.csv   # Flattened CSV version
├── docs/
│   ├── MVP_MASTER_SPECIFICATION.md  # Master spec (start here)
│   └── components/                  # 11 deep-dive documents
│       ├── 01_SYSTEM_ARCHITECTURE.md
│       ├── 02_DATABASE_SCHEMA.md
│       ├── 03_PRICING_LOGIC.md
│       ├── 04_CONCEPT_VIEWER.md
│       ├── 05_API_ENDPOINTS.md
│       ├── 06_MODEL_TRAINING.md
│       ├── 07_EVERGREEN_LOGIC.md
│       ├── 08_ROTATION_LOGIC.md
│       ├── 09_CASHBACK_FLOW.md
│       ├── 10_SUBTITLE_GENERATION.md
│       └── 11_IMPLEMENTATION_PHASES.md
└── INTEGRATION.md                   # How to use from hagen codespace
```

---

## How to Read and Understand the Data

### 1. Start with the Master Specification

```bash
# The master spec gives you the complete picture
cat docs/MVP_MASTER_SPECIFICATION.md
```

This document contains:
- System overview and core loop
- All database table definitions
- Pricing formula
- API endpoints summary
- Training pipeline overview
- Links to all component deep dives

### 2. Understand the Rating System

Videos are rated on **5+1 dimensions** (all 0-1 scale):

| Dimension | What It Measures |
|-----------|------------------|
| `hook` | Does the first 1-3 seconds grab attention? |
| `pacing` | Is the timing right? Does it hold attention? |
| `payoff` | Is the ending satisfying? Worth the watch? |
| `originality` | Is this fresh or just another copy? |
| `rewatchable` | Would you watch it again? Share? |
| `overall_score` | **TARGET VARIABLE** - holistic rating (0-1) |

### 3. Understand the Feature Schema

Each video has **170+ AI-extracted features** in the `visual_analysis` JSON. Key categories:

| Category | Example Features | Count |
|----------|-----------------|-------|
| `visual.*` | hookStrength, peopleCount, settingType | ~12 |
| `audio.*` | quality, hasVoiceover, soundEffects | ~10 |
| `content.*` | topic, style, format, duration | ~8 |
| `script.*` | conceptCore, transcript, replicability.score | ~33 |
| `casting.*` | minimumPeople, actingSkillRequired | ~6 |
| `production.*` | shotComplexity, timeToRecreate | ~5 |
| `flexibility.*` | industryLock, swappableCore | ~6 |
| `comedyStyle.*` | primaryTechnique, contrastMechanism | ~50+ |
| `standalone.*` | worksWithoutContext, memeDependent | ~5 |
| `trends.*` | memeDependent, usesPremadeSound | ~8 |

See `AI_HANDOFF.md` for the complete variable-by-variable schema.

### 4. Read the Training Data Export

```bash
# JSON format (full nested structure)
cat analysis_export_2025-12-03.json | jq '.[0] | keys'

# CSV format (flattened with dot notation)
head -1 analysis_export_2025-12-03.csv | tr ',' '\n' | head -20
```

**JSON Structure per video:**
```json
{
  "id": "uuid",
  "video_id": "uuid",
  "overall_score": 0.75,
  "dimensions": {
    "hook": 0.8,
    "pacing": 0.7,
    "payoff": 0.9,
    "originality": 0.6,
    "rewatchable": 0.7
  },
  "notes": "Owner's explanation of rating",
  "video": {
    "id": "uuid",
    "tiktok_id": "7xxx",
    "author": "@username",
    "visual_analysis": {
      "feature_count": 150,
      "visual": { ... },
      "audio": { ... },
      "script": { ... },
      // ... 170+ features
    }
  }
}
```

### 5. Understand Schema Versions

Not all videos have the same analysis depth:

| Version | Detection | Features | Training Use |
|---------|-----------|----------|--------------|
| v0 | `feature_count` missing | ~5 | ❌ Skip |
| v1 | `feature_count` < 50 | ~30 | ⚠️ Partial |
| v2 | `feature_count` 50-99 | ~80 | ⚠️ Partial |
| **v3** | `feature_count` >= 100 | 150-200 | ✅ Use for training |

**Only use v3 videos for training!**

```javascript
// Filter to v3 only
const trainingData = videos.filter(v => 
  v.visual_analysis?.feature_count >= 100
);
```

---

## Quick Reference: Key Numbers

| Metric | Value |
|--------|-------|
| Training-ready videos | 51 |
| Target for first model | 200 |
| Features per video | 170+ |
| Rating dimensions | 5+1 |
| Correlation milestones | 100, 200, 300, 500 |
| Budget constraint | $100-1000 |

---

## For AI Agents: Reading Order

If you're an AI agent trying to understand this system:

1. **First**: Read this README
2. **Then**: Read `docs/MVP_MASTER_SPECIFICATION.md` (complete picture)
3. **If needed**: Dive into specific component docs (01-11)
4. **For features**: Reference `AI_HANDOFF.md` section 4 (complete schema)
5. **For data**: Query or read `analysis_export_2025-12-03.json`

See `INTEGRATION.md` for how to fetch this from another codespace.

---

## Integration with hagen Codespace

See [INTEGRATION.md](./INTEGRATION.md) for:
- How to fetch documentation from this repo
- Chunked reading strategy for Claude Opus 4.5
- API patterns for querying training data