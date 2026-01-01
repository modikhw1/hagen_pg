# Component Specifications - letrend

> **Purpose**: Define behavior of every UI component
> **Status**: Draft - Awaiting Owner Input
> **Created**: January 1, 2026

---

## Component Index

| Category | Components |
|----------|------------|
| [Navigation](#navigation) | Header, Footer, Breadcrumbs |
| [Cards](#cards) | ConceptCard, PurchaseCard, TransactionCard |
| [Video](#video) | VideoPreview, VideoPlayer, SubtitleOverlay |
| [Forms](#forms) | FilterBar, CheckoutForm, CashbackForm |
| [Feedback](#feedback) | Toast, Modal, Badge, Meter |
| [Layout](#layout) | Sidebar, TabPanel, Accordion, Checklist |

---

## Navigation

### Header

**Purpose**: Global navigation, auth status, key actions

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ LOGGED OUT:                                                                  │
│ [Logo]                                              [Login] [Get Started]   │
├─────────────────────────────────────────────────────────────────────────────┤
│ LOGGED IN:                                                                   │
│ [Logo]           [Browse]  [My Purchases]           [$4.32 credits] [👤 ▼] │
└─────────────────────────────────────────────────────────────────────────────┘
```

**States:**
| State | Shows |
|-------|-------|
| Logged out | Logo, Login, Get Started |
| Logged in | Logo, Browse, My Purchases, Credits (if any), User menu |
| Staff | + Staff Dashboard link |

**Behaviors:**
- Logo → Homepage
- Credits badge → /purchases (where they can be used)
- User menu dropdown: Account, Logout
- Sticky on scroll: Yes (desktop), No (mobile - takes too much space)

**Mobile:**
```
┌─────────────────────────────────────────┐
│ [☰]  [Logo]              [👤]          │
└─────────────────────────────────────────┘
```
- Hamburger opens slide-out menu
- Menu contains: Browse, My Purchases, Credits, Account, Logout

---

### Footer

**Purpose**: Secondary navigation, legal, trust signals

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                              │
│  letrend                    Links              Legal                        │
│  Proven skit concepts       Browse             Terms of Service             │
│  for your business          How It Works       Privacy Policy               │
│                             FAQ                Refund Policy                │
│                             Contact                                         │
│                                                                              │
│  © 2026 letrend                                                             │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

**Behaviors:**
- Present on all pages except Viewer (distraction-free)
- Links open in same tab
- Mobile: Stacked layout

---

### Breadcrumbs

**Purpose**: Show location in hierarchy, enable back navigation

**When to show:**
| Page | Breadcrumbs |
|------|-------------|
| Homepage | None |
| Browse | None (top level) |
| Concept Detail | Browse > Concept Name |
| Checkout | Browse > Concept Name > Checkout |
| Viewer | My Purchases > Concept Name |
| Cashback Submit | My Purchases > Concept Name > Claim Cashback |

**Behavior:**
- Each segment clickable
- Truncate long concept names: "POV: You have to tell the kit..." (max 40 chars)

---

## Cards

### ConceptCard

**Purpose**: Display concept in browse grid

```
┌─────────────────────────────┐
│ ┌─────────────────────────┐ │
│ │                         │ │
│ │    [Video Thumbnail]    │ │
│ │                         │ │
│ │    ⏰ 47h left          │ │  ← Urgency badge (overlay)
│ └─────────────────────────┘ │
│                             │
│ "POV: You have to tell the  │  ← Headline (2 lines max)
│  kitchen you messed up"     │
│                             │
│ ⭐ 8.2   👥 1   ⏱ 15min     │  ← Quick stats row
│                             │
│ $12.00                      │  ← Price
│                             │
└─────────────────────────────┘
```

**States:**
| State | Visual |
|-------|--------|
| Default | As shown |
| Hover | Slight lift (shadow), thumbnail zooms slightly |
| Sold out (market) | Grayed out, "Sold out in your market" badge |
| Expired | Removed from browse (not shown) |

**Data displayed:**
| Field | Source | Fallback |
|-------|--------|----------|
| Thumbnail | GCS signed URL (thumbnail) | Placeholder |
| Headline | concept_core | "Untitled Concept" |
| Score | virality_score | "—" |
| People | casting.minimumPeople | "—" |
| Time | production.timeToRecreate | "—" |
| Price | calculated (PPP-adjusted) | "$—" |
| Time left | listing_window.window_end - now | "—h left" |

**Click behavior:**
- Entire card clickable → /concepts/[id]

---

### PurchaseCard

**Purpose**: Display owned concept in My Purchases

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ ┌───────────┐                                                               │
│ │ [Thumb]   │  "POV: You have to tell the kitchen..."                      │
│ │           │  Purchased: Dec 28, 2025                                     │
│ │           │                                                               │
│ └───────────┘  Cashback: ⏰ 28 days left        [View] [Claim Cashback]    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

**States:**
| State | Cashback Area | Actions |
|-------|---------------|---------|
| Active (can claim) | "⏰ X days left" (yellow) | [View] [Claim Cashback] |
| Pending verification | "⏳ Verification pending" (blue) | [View] [Check Status] |
| Claimed | "✅ $1.44 earned" (green) | [View] |
| Expired | "❌ Expired" (gray) | [View] |
| Rejected | "❌ Rejected: reason" (red) | [View] |

**Grouping in list:**
1. Active (sorted by deadline, soonest first)
2. Pending verification
3. Claimed (sorted by claim date, newest first)
4. Expired/Rejected (sorted by purchase date, newest first)

---

## Video

### VideoPreview

**Purpose**: Show limited preview before purchase

**Options (decision needed):**

**Option A: 5-Second Loop**
```
┌─────────────────────────────────────────┐
│                                         │
│         [5-sec video loop]              │
│         (no audio, auto-plays)          │
│                                         │
│         🔒 Purchase to unlock           │
│            full video                   │
│                                         │
└─────────────────────────────────────────┘
```
- Pros: Shows actual content quality
- Cons: Might give away too much, bandwidth

**Option B: Blurred Video**
```
┌─────────────────────────────────────────┐
│                                         │
│         [Full video, blurred]           │
│         (can see movement, not detail)  │
│                                         │
│         ▶ Preview not available         │
│           Purchase to watch             │
│                                         │
└─────────────────────────────────────────┘
```
- Pros: Creates mystery
- Cons: User can't assess quality

**Option C: Static Thumbnail**
```
┌─────────────────────────────────────────┐
│                                         │
│         [Single frame thumbnail]        │
│                                         │
│              ▶ (play icon overlay)      │
│                                         │
│         🔒 Purchase to watch            │
│                                         │
└─────────────────────────────────────────┘
```
- Pros: Simple, fast loading
- Cons: Least engaging

**[DECISION NEEDED: Which approach?]**

---

### VideoPlayer

**Purpose**: Full video playback in concept viewer (post-purchase)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                              │
│                                                                              │
│                         [VIDEO FRAME]                                        │
│                                                                              │
│                                                                              │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │  "Hey everyone, um... Ah, I'm already crying."                        │  │  ← Subtitle
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
│  [SETUP]━━━━━━━━━━━━●━━━━━━━━━━━━━━━[DEV]━━━━━━━━━━━━━━[PAYOFF]            │  ← Timeline with markers
│                      ▲                                                       │
│                   current                                                    │
│                                                                              │
│  [⏪ 5s]  [▶️]  [⏩ 5s]      0:08 / 0:24      [CC]  [⚙]  [⛶]              │  ← Controls
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

**Controls:**
| Control | Function |
|---------|----------|
| ⏪ 5s | Skip back 5 seconds |
| ▶️ / ⏸️ | Play / Pause |
| ⏩ 5s | Skip forward 5 seconds |
| Timeline | Seek (click/drag) |
| Scene markers | Jump to scene (click) |
| CC | Toggle subtitles |
| ⚙ | Settings (playback speed) |
| ⛶ | Fullscreen |

**Playback speeds:** 0.5x, 0.75x, 1x (default), 1.25x, 1.5x, 2x

**Scene markers:**
- Visual dots on timeline at scene boundaries
- Labeled: HOOK, SETUP, DEVELOPMENT, PAYOFF
- Click to jump
- Tooltip on hover shows scene label

**No download enforcement:**
```html
<video
  controlsList="nodownload"
  disablePictureInPicture
  oncontextmenu="return false"
>
```

**URL expiry handling:**
- Monitor for 403 errors
- On expiry: Pause video, show modal "Session expired. [Refresh]"
- Refresh: Fetch new signed URL, resume from same position

---

### SubtitleOverlay

**Purpose**: Display translated subtitles over video

**Position:** Bottom of video, above controls

**Styling:**
```css
.subtitle {
  background: rgba(0, 0, 0, 0.75);
  color: white;
  padding: 8px 16px;
  border-radius: 4px;
  font-size: 16px;
  max-width: 80%;
  text-align: center;
}
```

**Behavior:**
- Sync with video time
- Multiple lines if text is long
- Fade in/out at boundaries
- Toggle via CC button

---

## Forms

### FilterBar

**Purpose**: Filter concepts in browse view

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ Industry [Any ▼]  People [Any ▼]  Time [Any ▼]  Price [Any ▼]             │
│ [☐ Evergreen only]  [☐ Low acting skill]                     [Clear all]  │
└─────────────────────────────────────────────────────────────────────────────┘
```

**Filter options:**

| Filter | Options |
|--------|---------|
| Industry | Any, Restaurant, Café, Bar, Retail, Salon, Gym |
| People | Any, 1, 2, 3+ |
| Time | Any, <15min, <30min, <1hr |
| Price | Any, <$10, $10-20, $20+ |
| Evergreen only | Toggle |
| Low acting skill | Toggle (skill < 4) |

**Behavior:**
- Dropdowns close on selection
- Filters apply immediately (no Apply button)
- URL updates with query params
- Result count updates: "X concepts match"
- [Clear all] resets to defaults

**Mobile:**
```
┌─────────────────────────────────────────┐
│ [Filter ▼]                    12 results│
└─────────────────────────────────────────┘
```
- Opens slide-up panel with all filters
- [Apply Filters] button to close and apply

---

### CheckoutForm

**Purpose**: Collect payment and process purchase

**Sections:**

1. **Order Summary** (read-only)
2. **Credits** (if available)
3. **Payment Method**
4. **Submit**

**Payment options:**
- Stripe Elements (card input)
- Saved cards (if returning customer)
- Apple Pay button (if available)
- Google Pay button (if available)

**Validation:**
| Field | Validation |
|-------|------------|
| Card number | Stripe validates |
| Expiry | Stripe validates |
| CVC | Stripe validates |
| ZIP/Postal | Optional (based on Stripe settings) |

**States:**
| State | UI |
|-------|-----|
| Default | Form enabled |
| Submitting | Button disabled + spinner, form disabled |
| Error | Error message, form re-enabled |
| Success | Redirect to success page |

---

### CashbackForm

**Purpose**: Submit produced content URL for cashback

**Fields:**
| Field | Type | Validation |
|-------|------|------------|
| Platform | Radio (TikTok, Instagram, YouTube) | Required |
| Video URL | Text input | Required, URL format, matches platform |

**URL validation patterns:**
```javascript
const patterns = {
  tiktok: /tiktok\.com\/@[\w.-]+\/video\/\d+/,
  instagram: /instagram\.com\/(reel|p)\/[\w-]+/,
  youtube: /youtube\.com\/shorts\/[\w-]+/
};
```

**Helper text (changes by platform):**
- TikTok: "Paste your TikTok video URL (e.g., tiktok.com/@user/video/123)"
- Instagram: "Paste your Reel URL (e.g., instagram.com/reel/ABC123)"
- YouTube: "Paste your Shorts URL (e.g., youtube.com/shorts/ABC123)"

**States:**
| State | UI |
|-------|-----|
| Default | Form enabled |
| Invalid URL | Inline error, submit disabled |
| Submitting | Button disabled + spinner |
| Success | Redirect to confirmation |
| Error | Error message (deadline passed, already submitted) |

---

## Feedback

### Toast

**Purpose**: Brief feedback messages

**Types:**
| Type | Color | Icon | Duration |
|------|-------|------|----------|
| Success | Green | ✓ | 3s |
| Error | Red | ✗ | 5s (or until dismissed) |
| Info | Blue | ℹ | 4s |
| Warning | Yellow | ⚠ | 4s |

**Position:** Top-right (desktop), Top-center (mobile)

**Behavior:**
- Auto-dismiss after duration
- Click to dismiss early
- Stack if multiple (newest on top)

---

### Modal

**Purpose**: Focused actions requiring attention

**Types:**
| Type | Use Case |
|------|----------|
| Confirmation | "Are you sure?" before destructive actions |
| Auth | Login/Register prompt |
| Alert | Session expired, error details |

**Structure:**
```
┌─────────────────────────────────────────┐
│ [Title]                           [✕]  │
├─────────────────────────────────────────┤
│                                         │
│ [Content]                               │
│                                         │
├─────────────────────────────────────────┤
│              [Cancel]  [Primary Action] │
└─────────────────────────────────────────┘
```

**Behavior:**
- Backdrop click → close (except for critical alerts)
- Escape key → close
- Focus trap inside modal
- Body scroll locked while open

---

### Badge

**Purpose**: Status indicators

**Variants:**
| Variant | Color | Use |
|---------|-------|-----|
| success | Green | Cashback approved |
| warning | Yellow | Pending, deadline approaching |
| error | Red | Rejected, expired |
| info | Blue | Neutral information |
| neutral | Gray | Default |

**Sizes:**
- Small: Inline with text
- Medium: Standalone labels

---

### Meter

**Purpose**: Show 1-10 scale values

```
Acting Skill Required:
[████████░░] 8/10
Low ───────────── High
```

**Color coding:**
| Range | Color | Label |
|-------|-------|-------|
| 1-3 | Green | Low / Easy |
| 4-6 | Yellow | Medium / Moderate |
| 7-10 | Red | High / Difficult |

**Used for:**
- Acting skill required
- Personality dependency
- Attractiveness dependency
- Shot complexity
- Editing dependency
- Industry lock
- Prop dependency

---

## Layout

### Sidebar (Viewer)

**Purpose**: Tabbed content panel in concept viewer

```
┌─────────────────────────────────────┐
│ [Script] [Production] [Cast] [Adapt]│
├─────────────────────────────────────┤
│                                     │
│ [Active tab content]                │
│                                     │
│                                     │
│                                     │
└─────────────────────────────────────┘
```

**Tabs:**
| Tab | Content |
|-----|---------|
| Script | Concept core, structure, transcript |
| Production | Time, equipment, complexity |
| Casting | People, skill requirements |
| Adapt | Industry flexibility, swaps |

**Default tab:** Script

**Mobile:** Converts to accordion (see below)

---

### TabPanel

**Purpose**: Generic tabbed interface

**Behavior:**
- Only one tab active at a time
- Tab indicator (underline) shows active
- Content area updates on tab change
- Keyboard: Arrow keys move between tabs

---

### Accordion

**Purpose**: Collapsible sections (mobile viewer, FAQ)

```
┌─────────────────────────────────────┐
│ ▼ Script                            │  ← Expanded
├─────────────────────────────────────┤
│ [Content visible]                   │
├─────────────────────────────────────┤
│ ▶ Production                        │  ← Collapsed
├─────────────────────────────────────┤
│ ▶ Casting                           │  ← Collapsed
├─────────────────────────────────────┤
│ ▶ Adapt                             │  ← Collapsed
└─────────────────────────────────────┘
```

**Behavior options:**
- Single: Only one section open at a time (default for mobile viewer)
- Multiple: Any combination can be open (for FAQ)

---

### Checklist (Production)

**Purpose**: Interactive production checklist in viewer

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ PRODUCTION CHECKLIST                                    [3/6 complete]     │
│ [████████████░░░░░░░░░░░░]                                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│ 👥 CASTING                                                                  │
│ [✓] 1 person minimum                                                       │
│ [ ] No customer/stranger needed                                            │
├─────────────────────────────────────────────────────────────────────────────┤
│ 📷 EQUIPMENT                                                                │
│ [✓] Smartphone only                                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│ 📍 LOCATION                                                                 │
│ [ ] Indoor setting (counter/service area)                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│ ⏱ TIME                                                                      │
│ [✓] 15 minutes to shoot                                                    │
│ [ ] Basic editing (2 cuts)                                                 │
└─────────────────────────────────────────────────────────────────────────────┘
```

**Behavior:**
- Click checkbox or row to toggle
- Progress bar updates
- State saved to localStorage: `checklist_[conceptId]_[userId]`
- Categories auto-generated from concept data

**Categories:**
| Category | Icon | Items From |
|----------|------|-----------|
| Casting | 👥 | casting.minimumPeople, casting.requiresCustomer |
| Equipment | 📷 | production.equipmentNeeded |
| Location | 📍 | Inferred from template description |
| Time | ⏱ | production.timeToRecreate, production.editingDependency |
| Skills | 🎭 | casting.actingSkillRequired (if high) |

---

## Responsive Breakpoints

| Breakpoint | Width | Layout Changes |
|------------|-------|----------------|
| Mobile | < 640px | Single column, hamburger menu, accordion |
| Tablet | 640-1024px | Two columns where appropriate |
| Desktop | > 1024px | Full layout, sidebar |

---

## Component Library Recommendation

**Option A: Build from scratch with Tailwind**
- Pros: Full control, no dependencies
- Cons: More work, consistency harder

**Option B: shadcn/ui + Tailwind**
- Pros: Pre-built accessible components, customizable, no package lock-in
- Cons: Learning curve

**Option C: Radix UI + Tailwind**
- Pros: Unstyled primitives, full accessibility
- Cons: Need to style everything

**[DECISION NEEDED: Which approach?]**

Recommendation: **shadcn/ui** - Good balance of pre-built and customizable

---

*This document specifies component behavior. Awaiting owner input on decision points.*
