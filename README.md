# letrend

> Proven video concepts for your business's social media.

**letrend** is a personalized dashboard that helps small businesses discover and recreate viral TikTok concepts. We watch hundreds of videos and pick the ones that fit your business—so you don't have to.

---

## What is letrend?

Most small business owners know they should be on TikTok, but don't know what to post. letrend solves this by:

1. **Analyzing viral concepts** - We identify replicable video formats from around the world
2. **Matching to your business** - AI-powered matching based on your profile, tone, and constraints
3. **Making it easy to execute** - Plain-language guides, scripts, and checklists

**This is not a marketplace you browse.** It's a personalized dashboard that shows you what fits your business.

---

## Key Features

| Feature | Description |
|---------|-------------|
| **Social Sync** | Connect TikTok/Instagram for automatic brand profiling |
| **Match Percentage** | Every concept shows how well it fits your business (0-100%) |
| **Profile Completeness** | "How well we know you" meter encourages profile growth |
| **Curated Rows** | Top Matches, Fresh This Week, Easy Wins, Local Favorites |
| **Mini-Chat** | Quick refinement without leaving the dashboard |
| **Plain Language** | "Easy to film" not "Shot complexity: 3/10" |

---

## Documentation

This repository contains the complete specification for letrend.

```
docs/
├── MVP_MASTER_SPECIFICATION.md      # Start here - complete system overview
├── CUSTOMER_INTERFACE_OUTLINE.md    # Customer-facing UX summary
│
├── interface/                       # User experience documentation
│   ├── 01_USER_FLOWS.md             # Onboarding, dashboard, purchase flows
│   ├── 02_COMPONENTS.md             # UI component specifications
│   ├── 03_INFORMATION_ARCHITECTURE.md # What info appears where
│   └── 04_EDGE_CASES.md             # Error states, edge cases
│
└── components/                      # Technical deep dives
    ├── 01_SYSTEM_ARCHITECTURE.md    # Overall system design
    ├── 02_DATABASE_SCHEMA.md        # Database tables and relationships
    ├── 03_PRICING_LOGIC.md          # PPP-adjusted pricing ($20-30 US)
    ├── 04_CONCEPT_VIEWER.md         # Post-purchase viewer specs
    ├── 05_API_ENDPOINTS.md          # API specifications
    ├── 06_MODEL_TRAINING.md         # ML model for scoring concepts
    ├── 07_EVERGREEN_LOGIC.md        # Evergreen vs trending detection
    ├── 08_ROTATION_LOGIC.md         # 72-hour listing windows
    ├── 09_CASHBACK_FLOW.md          # Feedback loop incentives
    ├── 10_SUBTITLE_GENERATION.md    # Multi-language support
    ├── 11_IMPLEMENTATION_PHASES.md  # Development roadmap
    └── 12_PROFILE_AND_MATCHING.md   # User profiles and match % calculation
```

---

## Core Concepts

### Match Percentage (0-100%)

Every concept shows a personalized match score:

```
matchPercentage = (conceptScore × 0.6) + (profileFitScore × 0.4)
```

- **Concept Score (60%)**: Intrinsic quality—replicability, engagement potential, trend independence
- **Profile Fit (40%)**: How well it matches your business, goals, and constraints

### User Profile

Built through AI chat or social sync:

| Data Point | How Collected | Why Needed |
|------------|---------------|------------|
| Business type | AI chat | Match to industry-appropriate concepts |
| Team size | Social sync inference | Filter by people required |
| Tone preference | Social analysis | Match humor style |
| Goals | AI chat | Align recommendations to outcomes |
| Constraints | AI chat | Filter by difficulty/resources |

### Pricing

- **US base price**: $20-30 per concept
- **PPP-adjusted**: Lower prices for lower-income markets
- **Scarcity**: Limited sales per market (3-5) and globally (10-15)

---

## Target User

**Primary**: Small business owners (cafés, restaurants, salons, retail) who:
- Know they should be on TikTok but don't know what to post
- Have mid/low tech comfort
- Are time-constrained (can't watch hundreds of videos)
- Want proven formats, not creative risk

**Not for**: Agencies, influencers, or people who already know what works.

---

## Tech Stack

| Layer | Technology |
|-------|------------|
| Frontend | Next.js, React |
| Backend | Node.js, PostgreSQL |
| AI Analysis | Gemini 2.0 Flash (170+ features per video) |
| Matching | Custom ML model trained on owner ratings |
| Video | GCS storage, view-only (no downloads) |
| Payments | Stripe with PPP pricing |

---

## Getting Started

### For Developers

1. Read `docs/MVP_MASTER_SPECIFICATION.md` for the complete picture
2. Review `docs/interface/` for user-facing flows
3. Check `docs/components/05_API_ENDPOINTS.md` for API contracts

### For AI Agents

Reading order:
1. This README
2. `docs/MVP_MASTER_SPECIFICATION.md`
3. `docs/interface/03_INFORMATION_ARCHITECTURE.md`
4. Specific component docs as needed

---

## Data Assets

| File | Description |
|------|-------------|
| `analysis_export_2025-12-03.json` | Training data (51 rated videos) |
| `analysis_export_2025-12-03.csv` | Flattened CSV version |
| `AI_HANDOFF.md` | Complete 170+ feature schema |

---

## Related

- **hagen** (separate repo): Implementation codebase
- Training data and feature extraction system is operational
- MVP specification is complete and ready for implementation

---

*Built with the belief that small businesses deserve access to proven content strategies, not just guesswork.*
