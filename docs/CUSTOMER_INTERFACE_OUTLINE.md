# letrend - Customer Interface Outline

> **Purpose**: Define the customer-facing interface for the concept marketplace
> **Status**: Draft
> **Created**: January 1, 2026

---

## Service Summary

**letrend** is a marketplace where small businesses (restaurants, cafés, bars, etc.) discover and purchase viral TikTok skit concepts they can replicate for their own social media. Concepts are sourced cross-border—ideas that worked in one country, sold to businesses in another.

### Value Proposition

"Stop scrolling for ideas. Get proven skit concepts delivered to you, with everything you need to recreate them."

---

## User Types

### 1. Buyers (Primary Customers)
- Small business owners/managers
- Social media managers for local businesses
- Marketing freelancers serving SMBs
- **Goal**: Find easy-to-execute, proven skit formats for their business's TikTok/Reels

### 2. Staff (Internal)
- Curators who rate and approve concepts
- Support for buyer issues
- **Goal**: Maintain quality, manage the pipeline
- **Note**: Likely same app with elevated permissions, not a separate app

---

## Customer Journey & Pages

### Phase 1: Discovery

#### Landing Page (`/`)
**Purpose**: Explain the service, convert visitors to signups

```
┌─────────────────────────────────────────────────────────────────┐
│  [Logo]                              [Login] [Get Started]       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│     Stop Scrolling. Start Creating.                              │
│     Proven skit concepts for your business—                      │
│     ready to film in 15 minutes.                                 │
│                                                                  │
│     [Browse Concepts]  [How It Works]                            │
│                                                                  │
├─────────────────────────────────────────────────────────────────┤
│  HOW IT WORKS                                                    │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐        │
│  │ Browse   │→ │ Purchase │→ │  Study   │→ │ Produce  │        │
│  │ concepts │  │ (from $5)│  │  & learn │  │ your own │        │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘        │
│                                                                  │
├─────────────────────────────────────────────────────────────────┤
│  FEATURED CONCEPTS (preview - requires account to purchase)      │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐               │
│  │ [thumb] │ │ [thumb] │ │ [thumb] │ │ [thumb] │               │
│  │ 8.2/10  │ │ 7.5/10  │ │ 9.1/10  │ │ 7.8/10  │               │
│  │ $12     │ │ $8      │ │ $18     │ │ $10     │               │
│  └─────────┘ └─────────┘ └─────────┘ └─────────┘               │
│                                                                  │
├─────────────────────────────────────────────────────────────────┤
│  PERFECT FOR:                                                    │
│  Restaurants • Cafés • Bars • Retail • Salons • Gyms            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

**Key Elements:**
- Clear value prop above the fold
- Social proof (if available): "500+ concepts sold", "Used by 200+ businesses"
- Preview of concepts (teaser thumbnails, not full access)
- Trust signals: "View-only access", "No subscription required"

---

#### Browse/Marketplace (`/concepts`)
**Purpose**: Discover available concepts, filter by needs

```
┌─────────────────────────────────────────────────────────────────┐
│  [Logo]                    [Search...]     [My Purchases] [👤]  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  BROWSE CONCEPTS                                                 │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ Filters:                                                     ││
│  │ [Industry ▼] [People: 1-2 ▼] [Time: <30min ▼] [Price ▼]    ││
│  │ [Evergreen only ☐] [Low acting skill ☐]                     ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                  │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐               │
│  │ [Video     ]│ │ [Video     ]│ │ [Video     ]│               │
│  │ [Thumbnail ]│ │ [Thumbnail ]│ │ [Thumbnail ]│               │
│  │             │ │             │ │             │               │
│  │ "POV: The   │ │ "When the   │ │ "Customer   │               │
│  │ customer..."│ │ order is..."│ │ asks for..."│               │
│  │             │ │             │ │             │               │
│  │ ⭐ 8.2/10   │ │ ⭐ 7.5/10   │ │ ⭐ 9.1/10   │               │
│  │ 👥 1 person │ │ 👥 2 people │ │ 👥 1 person │               │
│  │ ⏱ 15 min   │ │ ⏱ 30 min   │ │ ⏱ 15 min   │               │
│  │             │ │             │ │             │               │
│  │ $12         │ │ $8          │ │ $18         │               │
│  │ ⏰ 47h left │ │ ⏰ 23h left │ │ ⏰ 71h left │               │
│  │             │ │             │ │             │               │
│  │ [Preview]   │ │ [Preview]   │ │ [Preview]   │               │
│  └─────────────┘ └─────────────┘ └─────────────┘               │
│                                                                  │
│  [Load More]                                                     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

**Key Filters:**
- **Industry**: Restaurant, Café, Bar, Retail, Salon, Gym, Any
- **People Required**: 1, 2, 3+
- **Time to Produce**: <15min, <30min, <1hr, Any
- **Price Range**: <$10, $10-20, $20+
- **Evergreen Only**: Exclude trend-dependent concepts
- **Acting Skill**: Low (<4), Medium (4-7), Any

**Card Information:**
- Thumbnail (blurred or partial for non-logged-in users)
- Concept headline (the "concept core")
- Virality score (0-10)
- Production quick stats: people, time
- Price (PPP-adjusted for user's market)
- Time remaining in listing window
- [Preview] button → Concept Detail page

---

### Phase 2: Evaluation

#### Concept Detail (`/concepts/[id]`)
**Purpose**: Evaluate before purchase—show enough to decide, not enough to replicate

```
┌─────────────────────────────────────────────────────────────────┐
│  [← Back to Browse]                              [My Purchases]  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────────────────┐  ┌─────────────────────────────┐   │
│  │                         │  │                              │   │
│  │   [VIDEO PREVIEW]       │  │  "POV: You have to tell     │   │
│  │   (5-second loop OR     │  │   the kitchen you messed    │   │
│  │    blurred full video)  │  │   up an order"              │   │
│  │                         │  │                              │   │
│  │   🔒 Full video after   │  │  Virality Score: ⭐ 8.2/10  │   │
│  │      purchase           │  │                              │   │
│  │                         │  │  ┌────────────────────────┐  │   │
│  └─────────────────────────┘  │  │ QUICK STATS            │  │   │
│                               │  │ 👥 1 person            │  │   │
│                               │  │ ⏱ 15 minutes          │  │   │
│                               │  │ 📷 Smartphone only     │  │   │
│                               │  │ 🎭 Low acting (2/10)   │  │   │
│                               │  │ 🔄 Works for: Any      │  │   │
│                               │  │ 🌲 Evergreen: Yes      │  │   │
│                               │  └────────────────────────┘  │   │
│                               │                              │   │
│                               │  $12.00                      │   │
│                               │  ⏰ 47 hours left            │   │
│                               │  📊 3 of 5 left in market   │   │
│                               │                              │   │
│                               │  [Purchase Now]              │   │
│                               │                              │   │
│                               │  ✓ 30-day access             │   │
│                               │  ✓ Full script + breakdown   │   │
│                               │  ✓ Production checklist      │   │
│                               │  ✓ 12% cashback available    │   │
│                               └─────────────────────────────┘   │
│                                                                  │
├─────────────────────────────────────────────────────────────────┤
│  WHY THIS WORKS                                                  │
│  "Self-deprecating humor where the employee is the butt of      │
│   the joke. Relatable to anyone who's worked service. Low       │
│   production value feels authentic, not like an ad."            │
│                                                                  │
├─────────────────────────────────────────────────────────────────┤
│  WORKS FOR THESE BUSINESSES                                      │
│  [Restaurant] [Café] [Bar] [Retail] [Any service business]      │
│                                                                  │
├─────────────────────────────────────────────────────────────────┤
│  WHAT YOU GET AFTER PURCHASE                                     │
│  • Full video with subtitles in your language                   │
│  • Scene-by-scene breakdown (hook → setup → payoff)             │
│  • Exact transcript                                              │
│  • Production checklist                                          │
│  • Casting & equipment requirements                              │
│  • Flexibility notes (what you can swap out)                    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

**Pre-Purchase Shows:**
- Limited preview (5-sec loop, blurred, or just thumbnail)
- Concept headline and "why it works" summary
- Quick stats (people, time, skill, equipment)
- Virality score
- Price and urgency (time left, market availability)
- What's included after purchase

**Pre-Purchase Hides:**
- Full video
- Transcript
- Scene breakdown
- Production details

---

### Phase 3: Purchase

#### Checkout (`/checkout/[concept-id]`)
**Purpose**: Complete purchase, simple and fast

```
┌─────────────────────────────────────────────────────────────────┐
│  CHECKOUT                                                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ [Thumb] "POV: You have to tell the kitchen..."             ││
│  │         ⭐ 8.2/10 • 1 person • 15 min                       ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                  │
│  Price breakdown:                                                │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ Concept access                              $10.71          ││
│  │ Cashback premium (refundable if produced)    $1.29          ││
│  │ ─────────────────────────────────────────────────────       ││
│  │ Total                                       $12.00          ││
│  │                                                              ││
│  │ 💰 Produce your version → get $1.44 back (12%)              ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                  │
│  Account credits: $0.00  [+ Add credits]                        │
│                                                                  │
│  Payment Method:                                                 │
│  ○ Credit Card                                                  │
│  ○ PayPal                                                       │
│                                                                  │
│  [Pay $12.00]                                                   │
│                                                                  │
│  ✓ Secure checkout                                              │
│  ✓ Instant access after payment                                 │
│  ✓ 30 days to claim cashback                                    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

**Elements:**
- Concept summary (what they're buying)
- Price breakdown showing cashback premium
- Account credits (from previous cashbacks)
- Payment methods
- Trust signals

---

### Phase 4: Learning (Post-Purchase)

#### Concept Viewer (`/viewer/[concept-id]`)
**Purpose**: Study the concept with everything needed to replicate

This is the core product experience. Based on `04_CONCEPT_VIEWER.md`:

```
┌─────────────────────────────────────────────────────────────────────────┐
│  [← My Purchases]                    [Claim Cashback] [Share feedback]  │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  ┌─────────────────────────────────┐  ┌──────────────────────────────┐  │
│  │                                 │  │  TABS: [Script] [Production] │  │
│  │        VIDEO PLAYER             │  │        [Casting] [Adapt]     │  │
│  │                                 │  │                              │  │
│  │   [Full video with subtitles]   │  │  ┌──────────────────────┐   │  │
│  │                                 │  │  │ CONCEPT CORE         │   │  │
│  │   🔒 View only (no download)    │  │  │ "Employee dreads     │   │  │
│  │                                 │  │  │  telling kitchen     │   │  │
│  │  ┌──────────────────────────┐   │  │  │  about mistake—      │   │  │
│  │  │ [SETUP]──[DEV]──[PAYOFF] │   │  │  │  cuts to their       │   │  │
│  │  │     ▲                    │   │  │  │  stone-faced         │   │  │
│  │  │  current                 │   │  │  │  reaction"           │   │  │
│  │  └──────────────────────────┘   │  │  └──────────────────────┘   │  │
│  │                                 │  │                              │  │
│  │  [⏪ 5s] [▶️ Play] [5s ⏩]       │  │  STRUCTURE BREAKDOWN         │  │
│  │                                 │  │  ┌──────────────────────┐   │  │
│  └─────────────────────────────────┘  │  │ HOOK (0-3s)          │   │  │
│                                       │  │ Text overlay + nerves │   │  │
│                                       │  ├──────────────────────┤   │  │
│                                       │  │ SETUP (3-8s)         │   │  │
│                                       │  │ "Hey everyone, um..."|   │  │
│                                       │  ├──────────────────────┤   │  │
│                                       │  │ PAYOFF (8-12s)       │   │  │
│                                       │  │ Cut to deadpan face  │   │  │
│                                       │  └──────────────────────┘   │  │
│                                       │                              │  │
│                                       │  FULL TRANSCRIPT             │  │
│                                       │  "Hey everyone, um... Ah,   │  │
│                                       │   I'm already crying."       │  │
│                                       └──────────────────────────────┘  │
│                                                                          │
│  ┌───────────────────────────────────────────────────────────────────┐  │
│  │ PRODUCTION CHECKLIST                                               │  │
│  │ ☐ 1 person (you or employee)                                      │  │
│  │ ☐ Smartphone                                                       │  │
│  │ ☐ Counter/service area setting                                    │  │
│  │ ☐ 15 minutes to shoot                                             │  │
│  │ ☐ Basic editing (2 cuts)                                          │  │
│  └───────────────────────────────────────────────────────────────────┘  │
│                                                                          │
│  ┌───────────────────────────────────────────────────────────────────┐  │
│  │ 💰 CLAIM YOUR CASHBACK                                             │  │
│  │ Produce your version, submit the link, get $1.44 back.            │  │
│  │ ⏰ 28 days remaining                    [Submit Production →]     │  │
│  └───────────────────────────────────────────────────────────────────┘  │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

**Sidebar Tabs:**

1. **Script Tab**
   - Concept core (one-sentence summary)
   - Structure breakdown (hook → setup → payoff)
   - Full transcript
   - Visual transcript (scene-by-scene with stage directions)

2. **Production Tab**
   - Time to recreate
   - Equipment needed
   - Shot complexity (1-10 meter)
   - Editing dependency (1-10 meter)
   - Production notes

3. **Casting Tab**
   - Minimum people
   - Acting skill required (1-10)
   - Personality dependency
   - Requires customer/stranger?
   - Casting notes

4. **Adapt Tab** (Flexibility)
   - Industry examples where this works
   - What elements are swappable
   - Swap suggestions
   - Industry lock score

**Bottom Section:**
- Production checklist (interactive, checkable)
- Cashback CTA with deadline countdown

---

### Phase 5: Production Support

#### My Purchases (`/purchases`)
**Purpose**: Access purchased concepts, track cashback

```
┌─────────────────────────────────────────────────────────────────┐
│  MY PURCHASES                                                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Account Credits: $4.32                     [Use at Checkout]   │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ ACTIVE (can still claim cashback)                           ││
│  ├─────────────────────────────────────────────────────────────┤│
│  │ [Thumb] "POV: Kitchen mistake"                              ││
│  │         Purchased: Dec 28, 2025                             ││
│  │         Cashback: ⏰ 28 days left                           ││
│  │         [View Concept] [Claim Cashback]                     ││
│  ├─────────────────────────────────────────────────────────────┤│
│  │ [Thumb] "When the regular customer..."                      ││
│  │         Purchased: Dec 15, 2025                             ││
│  │         Cashback: ⏰ 15 days left                           ││
│  │         [View Concept] [Claim Cashback]                     ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                  │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ CASHBACK CLAIMED                                            ││
│  ├─────────────────────────────────────────────────────────────┤│
│  │ [Thumb] "Impossible order"                                  ││
│  │         Purchased: Nov 20, 2025                             ││
│  │         Cashback: ✅ $1.80 earned                           ││
│  │         Your video: 12.4K views                             ││
│  │         [View Concept]                                      ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                  │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ EXPIRED (cashback window closed)                            ││
│  ├─────────────────────────────────────────────────────────────┤│
│  │ [Thumb] "Behind the scenes"                                 ││
│  │         Purchased: Oct 5, 2025                              ││
│  │         Cashback: ❌ Expired                                ││
│  │         [View Concept]                                      ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

#### Cashback Submission (`/cashback/submit/[transaction-id]`)
**Purpose**: Submit proof of production for cashback

```
┌─────────────────────────────────────────────────────────────────┐
│  CLAIM CASHBACK                                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ [Thumb] "POV: Kitchen mistake"                              ││
│  │         You paid: $12.00                                    ││
│  │         Cashback available: $1.44 (12%)                     ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                  │
│  Submit a link to your produced video:                          │
│                                                                  │
│  Platform:                                                       │
│  ○ TikTok                                                       │
│  ○ Instagram Reels                                              │
│  ○ YouTube Shorts                                               │
│                                                                  │
│  Video URL:                                                      │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ https://www.tiktok.com/@yourbusiness/video/123...          ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                  │
│  [Submit for Review]                                            │
│                                                                  │
│  ℹ️ Requirements:                                                │
│  • Video must be public                                         │
│  • Posted after your purchase date                              │
│  • Based on this concept (your interpretation)                  │
│  • Verification takes 24-48 hours                               │
│                                                                  │
│  💡 Higher engagement = higher cashback (up to 15%)             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Staff Interface (Same App, Elevated Permissions)

### Staff Dashboard (`/staff`)
```
┌─────────────────────────────────────────────────────────────────┐
│  STAFF DASHBOARD                                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐           │
│  │ Pending  │ │ Active   │ │ Cashback │ │ Revenue  │           │
│  │ Review   │ │ Listings │ │ Queue    │ │ Today    │           │
│  │   12     │ │   45     │ │    8     │ │  $234    │           │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘           │
│                                                                  │
│  [Rate Videos] [Manage Listings] [Verify Cashbacks] [Analytics] │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Video Rating Interface (`/staff/rate`)
- Queue of imported videos needing rating
- 5+1 dimension rating (hook, pacing, payoff, originality, rewatchable, overall)
- Notes field
- Approve/reject for marketplace

### Cashback Verification (`/staff/cashback`)
- Queue of pending cashback submissions
- View original concept + submitted video
- Approve/reject with reason
- Override cashback percentage

---

## Information Architecture

```
/                           Landing page
/concepts                   Browse marketplace
/concepts/[id]              Concept detail (pre-purchase)
/checkout/[id]              Purchase flow
/viewer/[id]                Concept viewer (post-purchase)
/purchases                  My purchased concepts
/cashback/submit/[tx-id]    Submit cashback claim
/account                    Account settings
/account/credits            Credit balance & history

/staff                      Staff dashboard (elevated role)
/staff/rate                 Video rating queue
/staff/listings             Manage active listings
/staff/cashback             Verify cashback submissions
/staff/analytics            Sales & performance data
```

---

## Key UX Principles

### 1. Urgency Without Pressure
- 72-hour windows create natural urgency
- "X left in your market" shows scarcity
- No fake countdown timers or dark patterns

### 2. Value Before Payment
- Show enough to evaluate (stats, preview, "why it works")
- Hide enough to require purchase (full video, transcript, breakdown)

### 3. Cashback as Feature, Not Afterthought
- Prominent throughout (pre-purchase, viewer, purchases)
- Clear deadline tracking
- Credits usable on next purchase

### 4. Mobile-First
- Target users (SMB owners) often on mobile
- Video viewer must work well on phones
- Quick purchase flow (Apple Pay, Google Pay)

### 5. Low Friction Onboarding
- Browse without account
- Account required only for purchase
- Social login (Google, Apple)

---

## MVP Feature Prioritization

### Must Have (Launch)
- [ ] Browse/filter concepts
- [ ] Concept detail page
- [ ] Purchase flow (Stripe)
- [ ] Concept viewer (video + script + checklist)
- [ ] My purchases page
- [ ] Basic cashback submission
- [ ] Staff rating interface

### Should Have (Soon After)
- [ ] Subtitles in buyer's language
- [ ] Email notifications (purchase, cashback reminders)
- [ ] Account credits system
- [ ] Cashback verification workflow
- [ ] Basic analytics for staff

### Could Have (Later)
- [ ] Performance metrics on cashback submissions
- [ ] "Similar concepts" recommendations
- [ ] Favorites/wishlist
- [ ] Bulk purchase discounts
- [ ] API for agencies

---

## Technical Considerations

### Authentication
- Supabase Auth (existing in hagen)
- Social logins: Google, Apple
- Role-based access: buyer, staff, admin

### Payments
- Stripe for purchases
- Stripe credits/balance for cashback
- PPP pricing via market_contexts table

### Video Delivery
- GCS signed URLs (4-hour expiry)
- No download enforcement (HTML5 controls disabled)
- HLS streaming for longer videos (future)

### Internationalization
- UI in English initially
- Subtitles generated per market (Google Translate API)
- Prices in local currency (via Stripe)

---

## Next Steps

1. **Validate this outline** with stakeholders
2. **Create wireframes** for key flows (Figma)
3. **Define API contracts** between letrend frontend and hagen backend
4. **Decide**: Separate Next.js app or extension of hagen?
5. **Set up project** (letrend repo or folder in hagen)

---

*This outline defines the customer-facing interface for letrend. The actual codebase (API, AI analysis, model training) remains in hagen-main.*
