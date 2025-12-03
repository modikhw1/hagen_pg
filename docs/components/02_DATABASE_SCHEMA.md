# Component 02: Database Schema Deep Dive

> **Parent Document**: [MVP Master Specification](../MVP_MASTER_SPECIFICATION.md)  
> **Component**: Database Schema  
> **Last Updated**: December 3, 2025

---

## Overview

This document provides exhaustive detail on every database table, column, relationship, and constraint in the system.

---

## 1. Existing Tables (From Current System)

### 1.1 `analyzed_videos`

Primary storage for imported videos with AI analysis.

```sql
CREATE TABLE analyzed_videos (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  video_url text NOT NULL,                    -- Original TikTok/Instagram URL
  video_id text,                              -- Platform's video ID (e.g., TikTok ID)
  platform text DEFAULT 'tiktok',             -- 'tiktok' | 'instagram' | 'youtube'
  metadata jsonb,                             -- Full platform metadata (author, stats)
  gcs_uri text,                               -- gs://hagen-video-analysis/videos/{id}.mp4
  visual_analysis jsonb,                      -- 170+ Gemini features (see schema below)
  content_embedding vector(1536),             -- OpenAI embedding for similarity
  created_at timestamp DEFAULT now(),         -- When imported
  analyzed_at timestamp,                      -- When Gemini analysis ran
  
  -- LEGACY COLUMNS (EMPTY - DO NOT USE)
  user_ratings jsonb,                         -- EMPTY: Use video_ratings table
  user_tags text[],                           -- EMPTY: Use video_ratings.tags
  user_notes text                             -- EMPTY: Use video_ratings.notes
);

-- Indexes
CREATE INDEX idx_analyzed_videos_platform ON analyzed_videos(platform);
CREATE INDEX idx_analyzed_videos_created ON analyzed_videos(created_at DESC);
CREATE INDEX idx_analyzed_videos_embedding ON analyzed_videos 
  USING ivfflat (content_embedding vector_cosine_ops);
```

**Example Record**:
```json
{
  "id": "695e6525-c41d-4152-96b4-931a3d75da29",
  "video_url": "https://www.tiktok.com/@kielekassidy/video/7263245062883085611",
  "video_id": "7263245062883085611",
  "platform": "tiktok",
  "metadata": {
    "author": {
      "uniqueId": "kielekassidy",
      "nickname": "Kiele Kassidy",
      "followerCount": 125000
    },
    "stats": {
      "playCount": 2500000,
      "diggCount": 180000,
      "shareCount": 5000,
      "commentCount": 2000
    },
    "desc": "POV: you have to tell the kitchen you messed up #restaurant #serverlife",
    "hashtags": ["restaurant", "serverlife", "pov"]
  },
  "gcs_uri": "gs://hagen-video-analysis/videos/695e6525-c41d-4152-96b4-931a3d75da29.mp4",
  "visual_analysis": { /* 170+ features - see section 3 */ },
  "created_at": "2025-12-03T10:00:00Z",
  "analyzed_at": "2025-12-03T12:59:40Z"
}
```

### 1.2 `video_ratings`

Human ratings (ground truth for model training).

```sql
CREATE TABLE video_ratings (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  video_id uuid REFERENCES analyzed_videos(id) NOT NULL,
  overall_score float NOT NULL,               -- 0-1 scale (primary target variable)
  dimensions jsonb NOT NULL,                  -- {hook, pacing, payoff, originality, rewatchable}
  notes text,                                 -- Free-form analysis
  tags text[],                                -- Classification tags
  ai_prediction jsonb,                        -- Model's pre-rating guess
  rated_at timestamp DEFAULT now(),
  
  -- NEW COLUMNS (for multi-rater future)
  rater_id uuid DEFAULT 'owner-uuid-here',    -- Who rated
  rater_type text DEFAULT 'owner',            -- 'owner' | 'agent'
  
  UNIQUE(video_id, rater_id)                  -- One rating per video per rater
);

-- Indexes
CREATE INDEX idx_video_ratings_video ON video_ratings(video_id);
CREATE INDEX idx_video_ratings_overall ON video_ratings(overall_score);
CREATE INDEX idx_video_ratings_rated ON video_ratings(rated_at DESC);
```

**Example Record**:
```json
{
  "id": "abc123",
  "video_id": "695e6525-c41d-4152-96b4-931a3d75da29",
  "overall_score": 0.3,
  "dimensions": {
    "hook": 0.5,
    "pacing": 0.56,
    "payoff": 0.43,
    "originality": 0.41,
    "rewatchable": 0.42
  },
  "notes": "The video is somewhat amusing... relies on TikTok sound which tells the script isn't strong...",
  "tags": ["restaurant", "gen-z-humor", "sound-dependent"],
  "ai_prediction": {
    "overall": 0.7,
    "modelUsed": "base",
    "reasoning": "Strong hook, good pacing...",
    "dimensions": {
      "hook": 0.8,
      "pacing": 0.9,
      "payoff": 0.6,
      "originality": 0.6,
      "rewatchable": 0.6
    },
    "user_disagreement": {
      "overall_delta": -0.4,
      "dimension_deltas": {
        "hook": -0.3,
        "pacing": -0.34,
        "payoff": -0.17,
        "originality": -0.19,
        "rewatchable": -0.18
      }
    }
  },
  "rated_at": "2025-12-03T14:00:00Z",
  "rater_id": "owner-uuid",
  "rater_type": "owner"
}
```

---

## 2. New Core Tables

### 2.1 `market_contexts`

Geographic and demographic segmentation for pricing and distribution.

```sql
CREATE TABLE market_contexts (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  country_code text NOT NULL UNIQUE,          -- ISO 3166-1 alpha-2 (e.g., 'US', 'ID')
  region_name text NOT NULL,                  -- Full name (e.g., 'Indonesia')
  primary_language text NOT NULL,             -- ISO 639-1 (e.g., 'en', 'id')
  purchasing_power_index float NOT NULL,      -- 1.0 = US baseline
  subtitle_priority int DEFAULT 5,            -- 1-10, higher = pre-generate subtitles
  currency_code text NOT NULL,                -- ISO 4217 (e.g., 'USD', 'IDR')
  notes text,                                 -- Admin notes
  created_at timestamp DEFAULT now()
);

-- Indexes
CREATE INDEX idx_market_contexts_country ON market_contexts(country_code);
CREATE INDEX idx_market_contexts_language ON market_contexts(primary_language);
```

**Example Records**:
```json
[
  {
    "id": "market-us",
    "country_code": "US",
    "region_name": "United States",
    "primary_language": "en",
    "purchasing_power_index": 1.0,
    "subtitle_priority": 1,
    "currency_code": "USD",
    "notes": "Primary market, baseline pricing"
  },
  {
    "id": "market-id",
    "country_code": "ID",
    "region_name": "Indonesia",
    "primary_language": "id",
    "purchasing_power_index": 0.25,
    "subtitle_priority": 3,
    "currency_code": "IDR",
    "notes": "High potential market, low PPP"
  }
]
```

**PPP Calculation Explanation**:
```
US concept worth $50 USD
Indonesia PPP = 0.25

Listed price in Indonesia = $50 × 0.25 = $12.50 USD equivalent
In IDR (at ~15,000 IDR/USD) = ~187,500 IDR

This makes concepts affordable relative to local income.
```

### 2.2 `model_versions`

Track trained preference models over time.

```sql
CREATE TABLE model_versions (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  version_tag text NOT NULL UNIQUE,           -- 'v1.0', 'v1.1', 'v2.0'
  trained_at timestamp NOT NULL,              -- When model was trained
  training_video_count int NOT NULL,          -- How many videos in training set
  feature_weights jsonb NOT NULL,             -- { "script.replicability.score": 0.82, ... }
  accuracy_metrics jsonb NOT NULL,            -- { "mae": 0.12, "rmse": 0.15, "r2": 0.68 }
  notes text,                                 -- Training notes, changes
  is_active boolean DEFAULT false             -- Only one active at a time
);

-- Indexes
CREATE INDEX idx_model_versions_active ON model_versions(is_active) WHERE is_active = true;
CREATE INDEX idx_model_versions_trained ON model_versions(trained_at DESC);

-- Constraint: Only one active model
CREATE UNIQUE INDEX idx_model_versions_single_active 
  ON model_versions(is_active) WHERE is_active = true;
```

**Example Record**:
```json
{
  "id": "model-v1",
  "version_tag": "v1.0",
  "trained_at": "2025-12-15T00:00:00Z",
  "training_video_count": 200,
  "feature_weights": {
    "script.replicability.score": 0.82,
    "script.humor.comedyTiming": 0.65,
    "casting.actingSkillRequired": -0.55,
    "trends.memeDependent": -0.68,
    "production.shotComplexity": -0.42,
    "audio.soundEffects_count": -0.38,
    "flexibility.industryLock": -0.35,
    "standalone.worksWithoutContext": 0.45,
    "comedyStyle.contrastMechanism.present": 0.52
    // ... 50+ more features
  },
  "accuracy_metrics": {
    "mae": 0.12,
    "rmse": 0.15,
    "r2": 0.68,
    "cross_validation_folds": 5,
    "test_set_size": 40
  },
  "notes": "First production model. Strong on replicability prediction.",
  "is_active": true
}
```

### 2.3 `concepts`

Abstracted concepts derived from source videos.

```sql
CREATE TABLE concepts (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  source_video_id uuid REFERENCES analyzed_videos(id) NOT NULL,
  concept_core text NOT NULL,                 -- One-line replicable concept
  template_description text,                  -- How to replicate
  required_elements text[],                   -- Must-have elements
  variable_elements text[],                   -- Swappable elements
  origin_market_id uuid REFERENCES market_contexts(id),
  virality_score float,                       -- 0-10 from model
  model_version_id uuid REFERENCES model_versions(id),
  evergreen_eligible boolean DEFAULT true,    -- Can be listed?
  archived boolean DEFAULT false,             -- Permanently unavailable?
  created_at timestamp DEFAULT now()
);

-- Indexes
CREATE INDEX idx_concepts_source ON concepts(source_video_id);
CREATE INDEX idx_concepts_virality ON concepts(virality_score DESC);
CREATE INDEX idx_concepts_archived ON concepts(archived);
CREATE INDEX idx_concepts_evergreen ON concepts(evergreen_eligible);
```

**Example Record**:
```json
{
  "id": "concept-123",
  "source_video_id": "695e6525-c41d-4152-96b4-931a3d75da29",
  "concept_core": "Employee dreads admitting mistake to kitchen, cut to deadpan reaction",
  "template_description": "Show person anxious about admitting mistake, cut to unexpected calm reaction from the people they're worried about",
  "required_elements": [
    "Person expressing anxiety",
    "Situation of admitting fault",
    "Cut to reaction"
  ],
  "variable_elements": [
    "Type of mistake",
    "Who they're telling",
    "The specific reaction (deadpan, supportive, etc.)"
  ],
  "origin_market_id": "market-us",
  "virality_score": 6.2,
  "model_version_id": "model-v1",
  "evergreen_eligible": true,
  "archived": false,
  "created_at": "2025-12-03T15:00:00Z"
}
```

---

## 3. New Marketplace Tables

### 3.1 `listing_windows`

Active marketplace listings with 72-hour windows.

```sql
CREATE TABLE listing_windows (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  concept_id uuid REFERENCES concepts(id) NOT NULL,
  market_context_id uuid REFERENCES market_contexts(id) NOT NULL,
  window_start timestamp NOT NULL,            -- When listing became active
  window_end timestamp NOT NULL,              -- window_start + 72 hours
  base_price_usd float NOT NULL,              -- Before any adjustments
  listed_price float NOT NULL,                -- After PPP + premium
  per_market_cap int DEFAULT 5,               -- Max sales in this market
  global_cap int DEFAULT 15,                  -- Max sales globally
  sold_count int DEFAULT 0,                   -- Current sales in this market
  status text DEFAULT 'active',               -- 'active' | 'sold_out' | 'expired' | 'archived'
  created_at timestamp DEFAULT now(),
  
  -- Prevent duplicate listings
  UNIQUE(concept_id, market_context_id, window_start)
);

-- Indexes
CREATE INDEX idx_listing_windows_concept ON listing_windows(concept_id);
CREATE INDEX idx_listing_windows_market ON listing_windows(market_context_id);
CREATE INDEX idx_listing_windows_status ON listing_windows(status);
CREATE INDEX idx_listing_windows_active ON listing_windows(status) WHERE status = 'active';
CREATE INDEX idx_listing_windows_end ON listing_windows(window_end);
```

**Status Flow**:
```
active → sold_out (when sold_count >= per_market_cap or global sales >= global_cap)
active → expired (when current_time > window_end)
expired → archived (via cron job)
sold_out → archived (via cron job after window_end)
```

**Example Record**:
```json
{
  "id": "listing-abc",
  "concept_id": "concept-123",
  "market_context_id": "market-id",
  "window_start": "2025-12-03T00:00:00Z",
  "window_end": "2025-12-06T00:00:00Z",
  "base_price_usd": 41.00,
  "listed_price": 11.48,
  "per_market_cap": 5,
  "global_cap": 15,
  "sold_count": 2,
  "status": "active",
  "created_at": "2025-12-03T00:00:00Z"
}
```

### 3.2 `transactions`

Purchase records linking buyers to concepts.

```sql
CREATE TABLE transactions (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  buyer_id uuid NOT NULL,                     -- From auth system
  concept_id uuid REFERENCES concepts(id) NOT NULL,
  listing_window_id uuid REFERENCES listing_windows(id) NOT NULL,
  market_context_id uuid REFERENCES market_contexts(id) NOT NULL,
  price_paid float NOT NULL,                  -- Actual amount paid
  currency_code text NOT NULL,                -- Currency of payment
  cashback_eligible boolean DEFAULT true,     -- Can claim cashback?
  cashback_amount float,                      -- price_paid × 0.12
  cashback_status text DEFAULT 'pending',     -- 'pending' | 'claimed' | 'expired'
  purchased_at timestamp DEFAULT now()
);

-- Indexes
CREATE INDEX idx_transactions_buyer ON transactions(buyer_id);
CREATE INDEX idx_transactions_concept ON transactions(concept_id);
CREATE INDEX idx_transactions_listing ON transactions(listing_window_id);
CREATE INDEX idx_transactions_cashback ON transactions(cashback_status);
```

**Cashback Status Flow**:
```
pending → claimed (when produced_content submitted and verified)
pending → expired (after 30 days with no submission)
```

**Example Record**:
```json
{
  "id": "txn-xyz",
  "buyer_id": "buyer-456",
  "concept_id": "concept-123",
  "listing_window_id": "listing-abc",
  "market_context_id": "market-id",
  "price_paid": 11.48,
  "currency_code": "USD",
  "cashback_eligible": true,
  "cashback_amount": 1.38,
  "cashback_status": "pending",
  "purchased_at": "2025-12-04T10:00:00Z"
}
```

### 3.3 `produced_content`

Buyer submissions for cashback + performance feedback.

```sql
CREATE TABLE produced_content (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  transaction_id uuid REFERENCES transactions(id) NOT NULL UNIQUE,
  submitted_url text NOT NULL,                -- Buyer's TikTok/Instagram URL
  platform text NOT NULL,                     -- 'tiktok' | 'instagram' | 'youtube'
  performance_metrics jsonb,                  -- { views, likes, shares, comments }
  verified boolean DEFAULT false,             -- Has been verified as matching concept
  verification_notes text,                    -- Admin notes on verification
  submitted_at timestamp DEFAULT now(),
  verified_at timestamp,
  metrics_updated_at timestamp
);

-- Indexes
CREATE INDEX idx_produced_content_transaction ON produced_content(transaction_id);
CREATE INDEX idx_produced_content_verified ON produced_content(verified);
CREATE INDEX idx_produced_content_platform ON produced_content(platform);
```

**Example Record**:
```json
{
  "id": "prod-789",
  "transaction_id": "txn-xyz",
  "submitted_url": "https://www.tiktok.com/@indonesian_creator/video/987654321",
  "platform": "tiktok",
  "performance_metrics": {
    "views": 52000,
    "likes": 4800,
    "shares": 320,
    "comments": 85,
    "scraped_at": "2025-12-10T00:00:00Z"
  },
  "verified": true,
  "verification_notes": "Content matches concept, good adaptation",
  "submitted_at": "2025-12-07T15:00:00Z",
  "verified_at": "2025-12-08T10:00:00Z",
  "metrics_updated_at": "2025-12-10T00:00:00Z"
}
```

### 3.4 `viewer_overlays`

Subtitles and scene markers for concept viewer.

```sql
CREATE TABLE viewer_overlays (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  concept_id uuid REFERENCES concepts(id) NOT NULL,
  overlay_type text NOT NULL,                 -- 'subtitle' | 'scene_marker' | 'timing_cue'
  language_code text NOT NULL,                -- ISO 639-1
  content text NOT NULL,                      -- The text to display
  timestamp_start float,                      -- Seconds from video start
  timestamp_end float,                        -- When to hide
  created_at timestamp DEFAULT now()
);

-- Indexes
CREATE INDEX idx_viewer_overlays_concept ON viewer_overlays(concept_id);
CREATE INDEX idx_viewer_overlays_language ON viewer_overlays(language_code);
CREATE INDEX idx_viewer_overlays_type ON viewer_overlays(overlay_type);
CREATE INDEX idx_viewer_overlays_concept_lang ON viewer_overlays(concept_id, language_code);
```

**Example Records**:
```json
[
  {
    "id": "overlay-1",
    "concept_id": "concept-123",
    "overlay_type": "scene_marker",
    "language_code": "en",
    "content": "SETUP",
    "timestamp_start": 0.0,
    "timestamp_end": 2.0
  },
  {
    "id": "overlay-2",
    "concept_id": "concept-123",
    "overlay_type": "subtitle",
    "language_code": "id",
    "content": "Hei semuanya, um...",
    "timestamp_start": 0.0,
    "timestamp_end": 1.5
  },
  {
    "id": "overlay-3",
    "concept_id": "concept-123",
    "overlay_type": "scene_marker",
    "language_code": "en",
    "content": "PAYOFF",
    "timestamp_start": 2.0,
    "timestamp_end": 5.0
  },
  {
    "id": "overlay-4",
    "concept_id": "concept-123",
    "overlay_type": "subtitle",
    "language_code": "id",
    "content": "Ah, aku sudah menangis.",
    "timestamp_start": 2.0,
    "timestamp_end": 3.5
  }
]
```

---

## 4. Analytics Tables

### 4.1 `feature_importance_snapshots`

Track which features matter over time.

```sql
CREATE TABLE feature_importance_snapshots (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  model_version_id uuid REFERENCES model_versions(id) NOT NULL,
  snapshot_at timestamp DEFAULT now(),
  training_count int NOT NULL,                -- Ratings at time of snapshot
  top_positive_features jsonb NOT NULL,       -- Features that predict HIGH ratings
  top_negative_features jsonb NOT NULL,       -- Features that predict LOW ratings
  pruned_features text[],                     -- Features removed (correlation < threshold)
  notes text
);

-- Indexes
CREATE INDEX idx_feature_snapshots_model ON feature_importance_snapshots(model_version_id);
CREATE INDEX idx_feature_snapshots_date ON feature_importance_snapshots(snapshot_at DESC);
```

**Example Record**:
```json
{
  "id": "snapshot-1",
  "model_version_id": "model-v1",
  "snapshot_at": "2025-12-15T00:00:00Z",
  "training_count": 200,
  "top_positive_features": [
    { "feature": "script.replicability.score", "correlation": 0.82, "rank": 1 },
    { "feature": "script.humor.comedyTiming", "correlation": 0.65, "rank": 2 },
    { "feature": "comedyStyle.contrastMechanism.present", "correlation": 0.52, "rank": 3 },
    { "feature": "standalone.worksWithoutContext", "correlation": 0.45, "rank": 4 },
    { "feature": "flexibility.swappableCore", "correlation": 0.42, "rank": 5 }
  ],
  "top_negative_features": [
    { "feature": "trends.memeDependent", "correlation": -0.68, "rank": 1 },
    { "feature": "casting.actingSkillRequired", "correlation": -0.55, "rank": 2 },
    { "feature": "production.shotComplexity", "correlation": -0.42, "rank": 3 },
    { "feature": "audio.soundEffects_count", "correlation": -0.38, "rank": 4 },
    { "feature": "flexibility.industryLock", "correlation": -0.35, "rank": 5 }
  ],
  "pruned_features": [
    "visual.colorDiversity",
    "audio.musicGenre",
    "content.emotionalTone"
  ],
  "notes": "Pruned 15 features with |correlation| < 0.15"
}
```

### 4.2 `arbitrage_metrics`

Track cross-border concept movement.

```sql
CREATE TABLE arbitrage_metrics (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  concept_id uuid REFERENCES concepts(id) NOT NULL,
  origin_market_id uuid REFERENCES market_contexts(id) NOT NULL,
  target_market_id uuid REFERENCES market_contexts(id) NOT NULL,
  time_to_first_sale interval,                -- How long until first sale
  total_sales int DEFAULT 0,                  -- Sales in this route
  conversion_rate float,                      -- Views / sales (if tracked)
  recorded_at timestamp DEFAULT now(),
  
  UNIQUE(concept_id, origin_market_id, target_market_id)
);

-- Indexes
CREATE INDEX idx_arbitrage_concept ON arbitrage_metrics(concept_id);
CREATE INDEX idx_arbitrage_origin ON arbitrage_metrics(origin_market_id);
CREATE INDEX idx_arbitrage_target ON arbitrage_metrics(target_market_id);
```

**Example Record**:
```json
{
  "id": "arb-1",
  "concept_id": "concept-123",
  "origin_market_id": "market-us",
  "target_market_id": "market-id",
  "time_to_first_sale": "4 hours",
  "total_sales": 3,
  "conversion_rate": 0.15,
  "recorded_at": "2025-12-05T00:00:00Z"
}
```

---

## 5. Visual Analysis Schema (Reference)

The `visual_analysis` JSONB column contains 170+ features. Full schema in [Feature Schema Reference](../reference/FEATURE_SCHEMA.md).

**Key Structure**:
```json
{
  "schema_version": "v3",
  "feature_count": 172,
  "analyzed_at": "2025-12-03T12:59:40Z",
  "analysis_model": "gemini-2.0-flash-vertex",
  
  "visual": { /* 13 fields */ },
  "audio": { /* 10 fields */ },
  "content": { /* 8 fields */ },
  "scenes": { /* 8 meta + N scene breakdowns */ },
  "script": {
    "conceptCore": "...",
    "humor": { /* 7 fields */ },
    "structure": { /* 10 fields */ },
    "emotional": { /* 4 fields */ },
    "replicability": { /* 6 fields */ },
    "originality": { /* 3 fields */ }
  },
  "casting": { /* 6 fields */ },
  "production": { /* 5 fields */ },
  "flexibility": { /* 6 fields */ },
  "comedyStyle": { /* 2 base + 8 sub-objects */ },
  "trends": { /* 9 fields */ },
  "brand": { /* 5 fields */ },
  "standalone": { /* 4 fields */ },
  "execution": { /* 4 fields */ },
  "technical": { /* 5 fields */ },
  "engagement": { /* 4 fields */ }
}
```

---

## 6. Entity Relationship Diagram

```
                                    ┌─────────────────┐
                                    │ market_contexts │
                                    └────────┬────────┘
                                             │
              ┌──────────────────────────────┼──────────────────────────────┐
              │                              │                              │
              ▼                              ▼                              ▼
    ┌─────────────────┐            ┌─────────────────┐            ┌─────────────────┐
    │    concepts     │◀───────────│analyzed_videos  │            │ listing_windows │
    └────────┬────────┘            └────────┬────────┘            └────────┬────────┘
             │                              │                              │
             │                              ▼                              │
             │                     ┌─────────────────┐                     │
             │                     │  video_ratings  │                     │
             │                     └─────────────────┘                     │
             │                                                             │
             │            ┌─────────────────┐                              │
             │            │ model_versions  │                              │
             │            └────────┬────────┘                              │
             │                     │                                       │
             │                     ▼                                       │
             │      ┌──────────────────────────┐                           │
             │      │feature_importance_snapshots│                         │
             │      └──────────────────────────┘                           │
             │                                                             │
             ├─────────────────────────────────────────────────────────────┤
             │                                                             │
             ▼                                                             ▼
    ┌─────────────────┐                                           ┌─────────────────┐
    │viewer_overlays  │                                           │  transactions   │
    └─────────────────┘                                           └────────┬────────┘
                                                                           │
                                                                           ▼
                                                                  ┌─────────────────┐
                                                                  │produced_content │
                                                                  └─────────────────┘
                                                                  
    ┌─────────────────┐
    │arbitrage_metrics│
    └─────────────────┘
```

---

## 7. Migration Scripts

### 7.1 Add Multi-Rater Support to Existing Table

```sql
-- Migration: Add rater columns to video_ratings
ALTER TABLE video_ratings 
ADD COLUMN IF NOT EXISTS rater_id uuid DEFAULT 'your-owner-uuid-here',
ADD COLUMN IF NOT EXISTS rater_type text DEFAULT 'owner';

-- Update existing records (all belong to owner)
UPDATE video_ratings 
SET rater_id = 'your-owner-uuid-here', rater_type = 'owner'
WHERE rater_id IS NULL;

-- Add constraint after data update
ALTER TABLE video_ratings 
ADD CONSTRAINT video_ratings_unique_rater UNIQUE(video_id, rater_id);
```

### 7.2 Create All New Tables

```sql
-- Run in order due to foreign key dependencies

-- 1. market_contexts (no dependencies)
CREATE TABLE IF NOT EXISTS market_contexts ( ... );

-- 2. model_versions (no dependencies)
CREATE TABLE IF NOT EXISTS model_versions ( ... );

-- 3. concepts (depends on analyzed_videos, market_contexts, model_versions)
CREATE TABLE IF NOT EXISTS concepts ( ... );

-- 4. listing_windows (depends on concepts, market_contexts)
CREATE TABLE IF NOT EXISTS listing_windows ( ... );

-- 5. transactions (depends on concepts, listing_windows, market_contexts)
CREATE TABLE IF NOT EXISTS transactions ( ... );

-- 6. produced_content (depends on transactions)
CREATE TABLE IF NOT EXISTS produced_content ( ... );

-- 7. viewer_overlays (depends on concepts)
CREATE TABLE IF NOT EXISTS viewer_overlays ( ... );

-- 8. feature_importance_snapshots (depends on model_versions)
CREATE TABLE IF NOT EXISTS feature_importance_snapshots ( ... );

-- 9. arbitrage_metrics (depends on concepts, market_contexts)
CREATE TABLE IF NOT EXISTS arbitrage_metrics ( ... );
```

---

## Related Documents

- [System Architecture Deep Dive](./01_SYSTEM_ARCHITECTURE.md)
- [API Endpoints Deep Dive](./05_API_ENDPOINTS.md)
- [Feature Schema Reference](../reference/FEATURE_SCHEMA.md)

---

*This document provides exhaustive detail on database schema. Refer to specific component documents for usage patterns.*
