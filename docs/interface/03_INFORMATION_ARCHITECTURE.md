# Information Architecture - letrend

> **Purpose**: Define what information appears where and why
> **Status**: Draft - Awaiting Owner Input
> **Created**: January 1, 2026

---

## Core Principle: Progressive Disclosure

Information is revealed progressively based on user commitment level:

```
VISITOR                    EVALUATOR                   BUYER                      PRODUCER
(browsing)                 (considering)               (purchased)                (claiming)
    │                          │                           │                          │
    ▼                          ▼                           ▼                          ▼
• Thumbnails               • Quick stats              • Full video               • Submit form
• Headlines                • Preview                  • Transcript               • Status tracking
• Scores                   • "Why it works"           • Structure                • Performance data
• Prices                   • Requirements             • Checklist
• Urgency                  • What you get             • All tabs
```

---

## Page-by-Page Information Architecture

### 1. Landing Page (`/`)

**Goal**: Convert visitors to browsers/registrations

| Section | Information | Why Here |
|---------|-------------|----------|
| Hero | Value prop, primary CTA | First impression, immediate clarity |
| How It Works | 4-step process | Reduce confusion, build understanding |
| Featured Concepts | 4-6 concept cards | Social proof, demonstrate product |
| Industries | Industry badges | Self-identification, relevance |
| Trust Signals | Stats, testimonials | Overcome skepticism |

**Information NOT shown:**
- Full concept details (→ browse)
- Pricing tables (→ individual concepts)
- Staff features (→ staff dashboard)

---

### 2. Browse Page (`/concepts`)

**Goal**: Help users find relevant concepts quickly

| Element | Information | Why |
|---------|-------------|-----|
| Filter bar | Industry, people, time, price, toggles | Self-service discovery |
| Result count | "X concepts match" | Feedback, expectation setting |
| Concept cards | Thumbnail, headline, score, stats, price, urgency | Quick evaluation |
| Sort | Ending Soon (default), Price, Score | Different user priorities |

**Card Information Hierarchy:**
```
┌─────────────────────────────────────────┐
│ 1. THUMBNAIL (visual hook)              │
│ 2. URGENCY BADGE (scarcity)             │
│ 3. HEADLINE (what is it)                │
│ 4. QUICK STATS (can I do this?)         │
│ 5. PRICE (what does it cost)            │
└─────────────────────────────────────────┘
```

**Why this order:**
1. Visual grabs attention
2. Urgency creates action pressure
3. Headline qualifies relevance
4. Stats filter out impossible concepts
5. Price is final decision factor

**Information NOT shown on cards:**
- Full description (→ detail page)
- Transcript (→ purchased)
- Why it works (→ detail page)

---

### 3. Concept Detail Page (`/concepts/[id]`)

**Goal**: Provide enough info to make purchase decision

| Section | Information | Why |
|---------|-------------|-----|
| Video Preview | Limited preview (decision needed) | Show quality without giving away concept |
| Purchase Panel | Price, urgency, stats, CTA | Clear path to purchase |
| Quick Stats | People, time, equipment, skill, flexibility, evergreen | "Can I actually do this?" |
| Why It Works | 2-3 sentences | Value justification |
| Industry Tags | Which businesses this fits | Self-qualification |
| What You Get | Bullet list | Set expectations |

**Information revealed vs. hidden:**

| SHOWN (Pre-Purchase) | HIDDEN (Post-Purchase) |
|---------------------|----------------------|
| Concept headline | Full transcript |
| Quick stats (people, time, etc.) | Scene-by-scene breakdown |
| Why it works (summary) | Visual transcript |
| Industry examples | Detailed production notes |
| Virality score | Casting notes |
| Limited video preview | Full video |
| Price | N/A |

**Rationale for hiding:**
- **Transcript**: The actual script is the product
- **Scene breakdown**: Implementation details
- **Full video**: Can be replicated without paying
- **Detailed notes**: Deep value reserved for buyers

---

### 4. Checkout Page (`/checkout/[id]`)

**Goal**: Complete purchase with confidence

| Section | Information | Why |
|---------|-------------|-----|
| Order Summary | Thumbnail, headline, key stats | Confirm what they're buying |
| Price Breakdown | Base + cashback premium = total | Transparency, explain cashback |
| Credits | Available balance, toggle | Incentivize return purchases |
| Payment | Card/wallet inputs | Transaction |
| Trust Signals | Security, instant access, cashback | Reduce friction |

**Information NOT shown:**
- Detailed concept info (→ already saw on detail page)
- Terms and conditions (linked, not inline)

**Price Breakdown Transparency:**
```
Concept access                    $10.71
Cashback premium (refundable)      $1.29
──────────────────────────────────────────
Total                             $12.00

💰 Produce your version → get $1.44 back
```

Why show this:
- Explains higher price
- Positions cashback as opportunity, not gimmick
- Sets expectation for post-purchase flow

---

### 5. Concept Viewer (`/viewer/[id]`)

**Goal**: Provide everything needed to replicate the concept

**This is the core product experience.**

| Section | Information | Why |
|---------|-------------|-----|
| Video Player | Full video, subtitles, scene markers | Source material |
| Script Tab | Concept core, structure, transcripts | The "script" |
| Production Tab | Time, equipment, complexity | Logistics |
| Casting Tab | People, skills, notes | Casting requirements |
| Adapt Tab | Industry examples, swappable elements | Customization |
| Checklist | Interactive production checklist | Actionable guidance |
| Cashback CTA | Deadline, amount, submit link | Drive next action |

**Tab Content Details:**

**Script Tab:**
| Element | Source | Purpose |
|---------|--------|---------|
| Concept Core | script.conceptCore | One-line summary |
| Hook | script.structure.hook | Opening seconds |
| Setup | script.structure.setup | Expectation established |
| Development | script.structure.development | Middle section |
| Payoff | script.structure.payoff | Resolution |
| Transcript | script.transcript | What is said |
| Visual Transcript | script.visualTranscript | Scene-by-scene with actions |

**Production Tab:**
| Element | Source | Purpose |
|---------|--------|---------|
| Time to Recreate | production.timeToRecreate | Planning |
| Equipment | production.equipmentNeeded | Shopping list |
| Shot Complexity | production.shotComplexity (1-10) | Skill assessment |
| Editing Dependency | production.editingDependency (1-10) | Post-production needs |
| Notes | production.productionNotes | Context |

**Casting Tab:**
| Element | Source | Purpose |
|---------|--------|---------|
| Minimum People | casting.minimumPeople | Crew planning |
| Requires Customer | casting.requiresCustomer | Special casting |
| Acting Skill | casting.actingSkillRequired (1-10) | Talent needs |
| Personality Dependency | casting.personalityDependency (1-10) | Persona requirements |
| Attractiveness Dependency | casting.attractivenessDependency (1-10) | Honest assessment |
| Notes | casting.castingNotes | Guidance |

**Adapt Tab:**
| Element | Source | Purpose |
|---------|--------|---------|
| Industry Examples | flexibility.industryExamples | Inspiration |
| Industry Lock | flexibility.industryLock (1-10) | Flexibility assessment |
| Swappable Core | flexibility.swappableCore | Can change premise? |
| Swap Examples | flexibility.swapExamples | How to adapt |
| Notes | flexibility.flexibilityNotes | Guidance |

---

### 6. My Purchases (`/purchases`)

**Goal**: Access owned concepts, manage cashback

| Section | Information | Why |
|---------|-------------|-----|
| Credits Bar | Balance, use CTA | Incentivize next purchase |
| Active Purchases | Concepts with cashback available | Priority action items |
| Pending | Submissions awaiting verification | Status awareness |
| Claimed | Successful cashbacks with amount | Satisfaction, tracking |
| Expired | Past deadline, no cashback | Complete history |

**Card Information by Status:**

| Status | Shown | Actions |
|--------|-------|---------|
| Active | Headline, purchase date, deadline countdown | View, Claim |
| Pending | Headline, submission date | View, Check Status |
| Claimed | Headline, amount earned, video performance | View |
| Expired | Headline, "Expired" badge | View |

---

### 7. Cashback Submit (`/cashback/submit/[tx-id]`)

**Goal**: Submit proof of production

| Section | Information | Why |
|---------|-------------|-----|
| Concept Reminder | Thumbnail, headline, amount available | Context |
| Form | Platform selector, URL input | Submission |
| Requirements | What qualifies, timeline | Set expectations |
| Bonus Info | Higher engagement = higher cashback | Incentivize quality |

---

## Data Visibility Matrix

Which data is visible at which stage:

| Data Field | Browse Card | Detail Page | Viewer | Staff |
|------------|-------------|-------------|--------|-------|
| thumbnail | ✓ | ✓ | ✓ | ✓ |
| concept_core | ✓ (headline) | ✓ | ✓ | ✓ |
| virality_score | ✓ | ✓ | ✓ | ✓ |
| price | ✓ | ✓ | N/A | ✓ |
| time_remaining | ✓ | ✓ | N/A | ✓ |
| market_availability | — | ✓ (logged in) | N/A | ✓ |
| casting.minimumPeople | ✓ | ✓ | ✓ | ✓ |
| production.timeToRecreate | ✓ | ✓ | ✓ | ✓ |
| production.equipmentNeeded | — | Summary | ✓ | ✓ |
| script.transcript | — | — | ✓ | ✓ |
| script.visualTranscript | — | — | ✓ | ✓ |
| script.structure.* | — | — | ✓ | ✓ |
| casting.actingSkillRequired | — | ✓ (summary) | ✓ | ✓ |
| flexibility.industryExamples | — | ✓ | ✓ | ✓ |
| full_video | — | Preview only | ✓ | ✓ |
| subtitles | — | — | ✓ | ✓ |
| human_rating.* | — | — | — | ✓ |
| notes (staff) | — | — | — | ✓ |

---

## URL Structure

```
/                               Landing page
/concepts                       Browse all
/concepts?industry=restaurant   Browse filtered
/concepts/[uuid]                Concept detail
/checkout/[uuid]                Purchase flow
/checkout/[uuid]/success        Purchase confirmation
/viewer/[uuid]                  Concept viewer
/purchases                      My purchases
/cashback/submit/[tx-uuid]      Submit cashback
/cashback/status/[tx-uuid]      Cashback status
/account                        Account settings
/account/credits                Credit history

/staff                          Staff dashboard
/staff/rate                     Rating queue
/staff/cashback                 Verification queue
/staff/listings                 Listing management
/staff/analytics                Analytics

/login                          Login page
/register                       Registration
/forgot-password                Password reset
```

**URL Design Principles:**
- UUIDs for resources (not sequential IDs)
- Filter params in query string (shareable)
- Clean paths (no `.html`, no trailing slashes)
- Concept ID consistent across detail/checkout/viewer

---

## Information Grouping Rationale

### Why group by "Can I do this?"

Users evaluating concepts ask:
1. Is this relevant to my business? → Industry tags, headline
2. Do I have the people? → minimumPeople
3. Do I have the time? → timeToRecreate
4. Do I have the equipment? → equipmentNeeded
5. Do I have the skill? → actingSkillRequired
6. Is this worth the price? → viralityScore, price

**Information architecture mirrors this mental model.**

### Why separate Script/Production/Casting/Adapt tabs?

Different user moments:
- **Script**: "What is this concept?"
- **Production**: "How do I shoot this?"
- **Casting**: "Who do I need?"
- **Adapt**: "How do I make it mine?"

Users don't need all info at once. Tabs reduce cognitive load.

### Why show virality score prominently?

- Differentiator from free TikTok browsing
- Justifies price variation
- Builds trust in curation
- Social proof equivalent

### Why show cashback throughout?

- Pre-purchase: Reduces effective price objection
- Viewer: Reminds of next step
- Purchases: Drives completion
- Deadline creates urgency

---

## Empty States

| Page | Empty State Message | Action |
|------|---------------------|--------|
| Browse (no results) | "No concepts match your filters." | "Try adjusting filters" or "Clear all" |
| Browse (no concepts) | "No concepts available right now. Check back soon!" | None |
| My Purchases | "You haven't purchased any concepts yet." | "Browse Concepts" |
| Cashback history | "No cashback claims yet." | Implied from purchases |

---

## Error States

| Error | Where | Message | Action |
|-------|-------|---------|--------|
| Concept not found | Detail page | "This concept doesn't exist or has been removed." | "Browse Concepts" |
| Concept expired | Detail page | "This concept is no longer available." | "Browse Concepts" |
| Concept sold out | Detail page | "This concept is sold out in your market." | "Browse Concepts" |
| Not purchased | Viewer | "You need to purchase this concept first." | Redirect to detail |
| Cashback expired | Submit | "The cashback window for this purchase has closed." | Back to purchases |
| Already submitted | Submit | "You've already submitted a cashback claim for this purchase." | Show status |
| Payment failed | Checkout | "Your payment could not be processed. [Reason]" | Retry |
| Network error | Any | "Something went wrong. Please try again." | Retry |

---

## Notification Architecture

### In-App Notifications

| Trigger | Message | Where |
|---------|---------|-------|
| Purchase complete | "Purchase complete! View your concept." | Toast + redirect |
| Cashback submitted | "Submission received. We'll verify within 24-48 hours." | Toast + redirect |
| Cashback approved | "Cashback approved! $X.XX credited." | Toast (on next visit) |
| Cashback rejected | "Cashback rejected: [reason]" | Toast (on next visit) |

### Email Notifications

| Trigger | Subject | Content |
|---------|---------|---------|
| Purchase | "Your concept is ready" | Receipt, viewer link, cashback info |
| 7 days before deadline | "7 days left to claim cashback" | Reminder, viewer link, submit link |
| 1 day before deadline | "Last day for cashback!" | Urgent reminder |
| Cashback approved | "Cashback approved - $X.XX credited" | Confirmation, balance, browse link |
| Cashback rejected | "Cashback claim update" | Reason, no resubmit |

---

## Search (Future)

Not in MVP, but architecture consideration:

**What would be searchable:**
- Concept headlines (concept_core)
- Industry tags
- NOT: Transcripts (paid content)

**Where:**
- Browse page: Search bar above filters
- Global: Header search

---

*This document defines information architecture. Awaiting owner input on priorities and changes.*
