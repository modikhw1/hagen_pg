# Component 01: System Architecture Deep Dive

> **Parent Document**: [MVP Master Specification](../MVP_MASTER_SPECIFICATION.md)  
> **Component**: System Architecture  
> **Last Updated**: December 3, 2025

---

## Overview

This document provides exhaustive detail on the system architecture for the Cross-Border Concept Arbitrage Marketplace.

---

## 1. High-Level Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────────────────────────┐
│                                    DATA LAYER                                            │
├─────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                          │
│  ┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐                   │
│  │  TikTok/Instagram │    │  Google Cloud    │    │    Supabase      │                   │
│  │     (Source)      │───▶│    Storage       │◀──▶│   PostgreSQL     │                   │
│  └──────────────────┘    │  (Video Files)   │    │   (All Data)     │                   │
│                          └──────────────────┘    └──────────────────┘                   │
│                                                           │                              │
├───────────────────────────────────────────────────────────┼──────────────────────────────┤
│                                    AI LAYER               │                              │
├───────────────────────────────────────────────────────────┼──────────────────────────────┤
│                                                           ▼                              │
│  ┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐                   │
│  │   Gemini 2.0     │    │   Your Trained   │    │    OpenAI        │                   │
│  │   Flash Vertex   │    │   Preference     │    │   Embeddings     │                   │
│  │  (170+ features) │    │     Model        │    │  (Similarity)    │                   │
│  └──────────────────┘    └──────────────────┘    └──────────────────┘                   │
│           │                       │                       │                              │
│           └───────────────────────┼───────────────────────┘                              │
│                                   ▼                                                      │
├─────────────────────────────────────────────────────────────────────────────────────────┤
│                              APPLICATION LAYER                                           │
├─────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                          │
│  ┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐                   │
│  │   Next.js 14     │    │   Cron Jobs      │    │   Translation    │                   │
│  │   (Frontend +    │    │  (Rotation,      │    │      API         │                   │
│  │    API Routes)   │    │   Analytics)     │    │  (Subtitles)     │                   │
│  └──────────────────┘    └──────────────────┘    └──────────────────┘                   │
│           │                       │                       │                              │
│           └───────────────────────┼───────────────────────┘                              │
│                                   ▼                                                      │
├─────────────────────────────────────────────────────────────────────────────────────────┤
│                                 USER LAYER                                               │
├─────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                          │
│  ┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐                   │
│  │      Owner       │    │   Data Input     │    │     Buyers       │                   │
│  │   (Rates/Trains) │    │     Agents       │    │  (View/Purchase) │                   │
│  └──────────────────┘    └──────────────────┘    └──────────────────┘                   │
│                                                                                          │
└─────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Core Loop Explained

The system operates on a continuous loop with distinct phases:

### Phase 1: Discovery
```
Input:  TikTok URL (manual or scraped)
Output: Video stored in GCS, metadata in Supabase

Example:
  1. Owner finds interesting skit: https://www.tiktok.com/@creator/video/123456
  2. Calls POST /api/tiktok with URL
  3. System downloads video to gs://hagen-video-analysis/videos/{uuid}.mp4
  4. Extracts metadata via Supadata API
  5. Creates record in analyzed_videos table
```

### Phase 2: Analysis
```
Input:  Video file in GCS
Output: 170+ features in visual_analysis JSONB

Example:
  1. Call POST /api/videos/reanalyze with video_id
  2. Gemini 2.0 Flash analyzes video
  3. Extracts: visual, audio, script, casting, production, comedyStyle, trends, etc.
  4. Stores in analyzed_videos.visual_analysis
  5. Tags with schema_version: 'v3', feature_count: 172
```

### Phase 3: Rating (Training Data Collection)
```
Input:  Video with analysis
Output: Human rating (ground truth)

Example:
  1. Owner views video in rating interface
  2. AI provides prediction BEFORE owner rates (stored for comparison)
  3. Owner rates: overall=0.7, hook=0.8, pacing=0.6, payoff=0.7, originality=0.5, rewatchable=0.6
  4. Owner adds notes: "Strong concept, weak execution on timing"
  5. Stored in video_ratings with ai_prediction for later delta analysis
```

### Phase 4: Training
```
Input:  All rated videos with v3 analysis
Output: Trained model with feature weights

Example:
  1. Export 200+ rated videos as JSON
  2. Flatten 170+ features to dot notation
  3. Run Pearson/Spearman correlation with overall_score
  4. Top correlations:
     - script.replicability.score: +0.82
     - trends.memeDependent: -0.68
     - casting.actingSkillRequired: -0.55
  5. Train Ridge regression model
  6. Store in model_versions: { version_tag: 'v1.0', feature_weights: {...} }
```

### Phase 5: Scoring
```
Input:  New video with analysis
Output: Virality score (0-10)

Example:
  1. New video analyzed, has visual_analysis
  2. Call POST /api/model/predict with video features
  3. Model applies learned weights:
     virality = Σ(feature_value × feature_weight)
  4. Returns: { virality_score: 7.2, confidence: 0.85, top_factors: [...] }
```

### Phase 6: Listing
```
Input:  Scored concept
Output: Active listing in marketplace

Example:
  1. Concept has virality_score = 7.2
  2. Check evergreen_eligible = true (no memes, no trending sounds)
  3. Calculate price: base($5) + virality($36) = $41 USD
  4. Adjust for Indonesia: $41 × 0.25 PPP = $10.25
  5. Add premium: $10.25 × 1.12 = $11.48
  6. Create listing_window: 72 hours, cap 5/market, 15/global
```

### Phase 7: Purchase
```
Input:  Buyer selects listing
Output: Transaction record, access granted

Example:
  1. Indonesian buyer browses /api/listings?market=ID
  2. Selects concept, pays $11.48 IDR equivalent
  3. Transaction created: { buyer_id, concept_id, price_paid, cashback_amount: $1.38 }
  4. listing_window.sold_count incremented
  5. Buyer gains access to concept viewer
```

### Phase 8: Production
```
Input:  Buyer access
Output: Buyer creates their version

Example:
  1. Buyer views concept in viewer
  2. Sees: video playback, translated subtitles, script, production checklist
  3. Checklist: "☐ 2 people ☐ Smartphone ☐ Indoor setting ☐ 15min shoot"
  4. Buyer replicates concept in their language/context
  5. Posts to their TikTok
```

### Phase 9: Performance Feedback
```
Input:  Buyer submits produced content URL
Output: Cashback + performance data

Example:
  1. Buyer calls POST /api/transactions/{id}/submit-content
  2. Submits: { url: 'https://tiktok.com/@buyer/video/789' }
  3. System creates produced_content record
  4. Verifies (manual or automated) content matches concept
  5. Cashback processed: $1.38 refunded
  6. (Future) Scrape performance: { views: 50000, likes: 5000, shares: 200 }
  7. Performance data informs model: "This concept type performs well in Indonesia"
```

---

## 3. Data Flow Diagram

```
                    ┌─────────────┐
                    │  TikTok URL │
                    └──────┬──────┘
                           │
                           ▼
              ┌────────────────────────┐
              │   POST /api/tiktok     │
              │  (Import + Download)   │
              └────────────┬───────────┘
                           │
           ┌───────────────┼───────────────┐
           ▼               ▼               ▼
    ┌──────────┐    ┌──────────┐    ┌──────────┐
    │   GCS    │    │ Supabase │    │ Supadata │
    │ (Video)  │    │(Metadata)│    │  (Stats) │
    └────┬─────┘    └────┬─────┘    └──────────┘
         │               │
         ▼               │
┌──────────────────┐     │
│ POST /api/videos │     │
│   /reanalyze     │     │
└────────┬─────────┘     │
         │               │
         ▼               │
┌──────────────────┐     │
│  Gemini 2.0      │     │
│  (170+ features) │     │
└────────┬─────────┘     │
         │               │
         └───────┬───────┘
                 ▼
        ┌──────────────┐
        │analyzed_videos│
        │visual_analysis│
        └───────┬──────┘
                │
                ▼
        ┌──────────────┐
        │  Rating UI   │
        │ (Owner rates)│
        └───────┬──────┘
                │
                ▼
        ┌──────────────┐
        │video_ratings │
        │(Ground truth)│
        └───────┬──────┘
                │
    ┌───────────┴───────────┐
    │  At 200+ ratings      │
    ▼                       │
┌──────────────┐            │
│ Correlation  │            │
│  Analysis    │            │
└──────┬───────┘            │
       │                    │
       ▼                    │
┌──────────────┐            │
│model_versions│            │
│(Trained model)│           │
└──────┬───────┘            │
       │                    │
       ▼                    │
┌──────────────┐            │
│   concepts   │◀───────────┘
│(Abstracted)  │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│listing_windows│
│ (72h active) │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ transactions │
│  (Purchases) │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│produced_content│
│  (Feedback)  │
└──────────────┘
```

---

## 4. User Roles and Permissions

### Owner (You)
| Action | Access | Purpose |
|--------|--------|---------|
| Import videos | Full | Discover new concepts |
| Rate videos | Full | Train the model |
| View all data | Full | Monitor system |
| Adjust prices | Full | Override model suggestions |
| Review submissions | Full | Verify produced content |

### Data Input Agents (Future)
| Action | Access | Purpose |
|--------|--------|---------|
| Import videos | Limited (queue) | Scale discovery |
| Rate videos | None initially | Owner trains model |
| Adjust prices | Limited (±20%) | Local market knowledge |
| Flag content | Full | Quality control |

### Buyers
| Action | Access | Purpose |
|--------|--------|---------|
| Browse listings | By market | Find concepts |
| Purchase | Own market | Acquire concepts |
| View concepts | Purchased only | Learn and replicate |
| Submit content | Own transactions | Claim cashback |

---

## 5. Key Constraints Explained

### View-Only Platform (No Downloads)

**Why**: 
- Control distribution: Prevent unauthorized redistribution
- Enable overlays: Subtitles, scene markers rendered in-player
- Track engagement: Know how buyers interact with content
- Future upsells: Premium download tier possible later

**Implementation**:
```typescript
// Video URL is signed, short-lived, and logged
async function getViewerUrl(conceptId: string, buyerId: string): Promise<string> {
  // Verify buyer has purchased
  const transaction = await db.transactions.findFirst({
    where: { concept_id: conceptId, buyer_id: buyerId }
  });
  if (!transaction) throw new Error('Not purchased');
  
  // Generate signed URL valid for 4 hours
  const signedUrl = await gcs.getSignedUrl(concept.source_video.gcs_uri, {
    expires: Date.now() + 4 * 60 * 60 * 1000
  });
  
  // Log view
  await db.viewLogs.create({ buyer_id: buyerId, concept_id: conceptId });
  
  return signedUrl;
}
```

### 72-Hour Listing Windows

**Why**:
- Creates urgency: Buyers motivated to act fast
- Ensures freshness: Marketplace feels dynamic, not stale
- Prevents hoarding: Concepts cycle through
- Enables repricing: Failed listings can re-enter at different price

**Implementation**:
```typescript
// Cron: Every hour
async function checkExpirations() {
  const now = new Date();
  
  // Find expired active listings
  const expired = await db.listingWindows.findMany({
    where: {
      status: 'active',
      window_end: { lt: now }
    }
  });
  
  for (const listing of expired) {
    await db.listingWindows.update({
      where: { id: listing.id },
      data: { status: 'archived' } // Direct to archived, no re-activation
    });
    
    // Check if concept should be fully archived
    const otherActiveListings = await db.listingWindows.count({
      where: {
        concept_id: listing.concept_id,
        status: 'active'
      }
    });
    
    if (otherActiveListings === 0) {
      await db.concepts.update({
        where: { id: listing.concept_id },
        data: { archived: true }
      });
    }
  }
}
```

### Dual Purchase Caps (3-5 per market, 10-15 global)

**Why**:
- Prevents over-saturation: Same concept everywhere dilutes value
- Creates scarcity: Limited supply increases perceived value
- Enables arbitrage: Concept can succeed in multiple markets
- Protects creators: Original creator not overwhelmed by copies

**Implementation**:
```typescript
async function canPurchase(conceptId: string, marketId: string): Promise<boolean> {
  const listing = await db.listingWindows.findFirst({
    where: {
      concept_id: conceptId,
      market_context_id: marketId,
      status: 'active'
    }
  });
  
  if (!listing) return false;
  
  // Check per-market cap
  if (listing.sold_count >= listing.per_market_cap) return false;
  
  // Check global cap
  const globalSold = await db.transactions.count({
    where: { concept_id: conceptId }
  });
  
  if (globalSold >= listing.global_cap) return false;
  
  return true;
}
```

### No Exclusivity

**Why**:
- Maximizes revenue: Same concept sold multiple times
- Enables true arbitrage: US concept sold in Indonesia, Brazil, etc.
- Reduces pricing complexity: No exclusivity tiers
- Matches reality: TikTok concepts are already public

**What This Means**:
- Buyer in US purchases concept → Can still be sold in Indonesia
- Buyer does NOT get exclusive rights
- Buyer cannot resell or redistribute
- Multiple buyers in same market possible (up to cap)

### No Re-Activation

**Why**:
- Simplifies logic: Archived = done, forever
- Prevents gaming: Can't re-list failed concepts repeatedly
- Encourages fresh content: Focus on new discoveries
- Matches 72h freshness goal: Old concepts are old

**What This Means**:
- Once a listing expires → Moves to archived
- Once all listings for a concept archived → Concept archived
- Archived concepts never return to marketplace
- Historical data retained for analytics

---

## 6. Technology Stack Details

### Frontend: Next.js 14
```
- React 18 with Server Components
- App Router for file-based routing
- TailwindCSS for styling
- Shadcn/ui for components (if used)
```

### Backend: Next.js API Routes
```
- Serverless functions on Vercel (or self-hosted)
- TypeScript for type safety
- Zod for request validation
```

### Database: Supabase PostgreSQL
```
- PostgreSQL 15
- pgvector extension for embeddings
- Row-level security for multi-tenant (future)
- Realtime subscriptions (optional)
```

### Storage: Google Cloud Storage
```
- Bucket: gs://hagen-video-analysis/
- Structure: /videos/{uuid}.mp4
- Signed URLs for secure access
- Lifecycle policies for cost management
```

### AI: Google Gemini 2.0 Flash
```
- Vertex AI deployment
- Video understanding capability
- 170+ feature extraction prompt
- ~$0.001-0.005 per video analysis
```

### Embeddings: OpenAI text-embedding-3-small
```
- 1536 dimensions
- Used for similarity search
- Stored in content_embedding column
```

### Metadata: Supadata API
```
- TikTok metadata extraction
- Views, likes, shares, comments
- Author info, hashtags
```

---

## 7. Security Considerations

### Data Protection
```
- Videos stored in private GCS bucket
- Signed URLs expire after 4 hours
- No direct public access to videos
- Buyer access logged
```

### Authentication
```
- Supabase Auth (or custom)
- JWT tokens for API access
- Role-based access control
```

### Rate Limiting
```
- API routes rate limited
- Import limited to prevent abuse
- Purchase limited to prevent hoarding
```

### Content Moderation
```
- Owner reviews all content before listing
- Buyer-submitted URLs verified before cashback
- Brand safety from Gemini analysis (brand.riskLevel)
```

---

## 8. Scalability Considerations

### Current Scale (MVP)
```
- 100-500 videos
- 1 owner rating
- 10 initial markets
- 100s of transactions
```

### Future Scale
```
- 10,000+ videos
- Multiple data input agents
- 50+ markets
- 10,000s of transactions
```

### Bottlenecks and Solutions

| Bottleneck | Current | Scaled Solution |
|------------|---------|-----------------|
| Video storage | GCS (no limit) | Same |
| Gemini analysis | Sequential | Queue + parallel workers |
| Database queries | Single instance | Read replicas |
| Subtitle generation | On-demand | Pre-generate top 5 languages |
| Rotation cron | Hourly | Edge functions |

---

## Related Documents

- [Database Schema Deep Dive](./02_DATABASE_SCHEMA.md)
- [API Endpoints Deep Dive](./05_API_ENDPOINTS.md)
- [Model Training Deep Dive](./06_MODEL_TRAINING.md)

---

*This document provides exhaustive detail on system architecture. Refer to specific component documents for implementation details.*
