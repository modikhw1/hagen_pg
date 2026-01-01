# Component 03: Pricing Logic Deep Dive

> **Parent Document**: [MVP Master Specification](../MVP_MASTER_SPECIFICATION.md)  
> **Component**: Pricing Logic  
> **Last Updated**: December 3, 2025

---

## Overview

This document provides exhaustive detail on how concept prices are calculated, including virality scoring, purchasing power adjustment, agent modifiers, and the cashback premium.

---

## 0. Pricing Design Rationale

### Budget Context

From chat discussion: The owner specified a **$100-1000 budget** range for MVP development, which influenced these pricing decisions:

- **Low base price ($5)**: Accessible entry point for emerging markets
- **Simple multiplier model**: Easy to implement without complex ML pricing
- **PPP adjustment**: Essential for cross-border marketplace viability
- **12% cashback premium**: Self-funding feedback loop without external capital

### Why These Specific Values?

| Decision | Value | Rationale |
|----------|-------|-----------|
| Base price | $5 | Low enough for any market after PPP adjustment; high enough to feel "worth something" |
| Virality multiplier | $5 | A "perfect 10" concept costs $55 USD base, which feels premium but not absurd |
| Cashback premium | 12% | Funds 10-15% cashback with margin for payment processing fees (~3%) |
| Max agent modifier | ±20% | Enough flexibility for local context without creating arbitrage opportunities |
| Price floor | $1 | Below this, transaction fees dominate |
| Price ceiling | $500 | Above this, buyers would expect more than a concept |

### Cashback Premium Math

```
Revenue needed per transaction to fund 10-15% cashback:

If buyer pays $11.20 (includes 12% premium on $10 base):
- Premium collected: $1.20
- Cashback paid: $1.00-$1.50 (10-15% of $10)
- Net margin: -$0.30 to +$0.20

At scale, high performers (who claim cashback) subsidized by low performers (who don't claim).
Typical cashback claim rate: 30-50% expected
Effective cost: 3-7.5% of transaction value
12% premium covers this with margin.
```

### Connection to Model Confidence

**Open question**: Should low-confidence predictions affect pricing?

Two approaches considered:
1. **Confidence discount**: If model confidence < 0.5, reduce virality multiplier by 50%
   - Pro: Honest pricing when model is uncertain
   - Con: Penalizes novel content that doesn't match training data

2. **Separate display**: Show virality score AND confidence separately, don't affect price
   - Pro: Full information for buyer to decide
   - Con: More cognitive load

**Current decision**: Approach 2 (display both, don't adjust price). Can revisit at 500+ ratings when model is more stable.

---

## 1. Pricing Formula

### Complete Formula

```
listed_price = ((base_price + (virality_score × virality_multiplier)) 
                × purchasing_power_index 
                × (1 + agent_modifier)) 
               × (1 + cashback_premium)
```

### Component Breakdown

| Component | Default Value | Range | Purpose |
|-----------|---------------|-------|---------|
| `base_price` | $5 USD | $5-$20 | Minimum viable price |
| `virality_score` | Model output | 0-10 | Predicted value of concept |
| `virality_multiplier` | $5 | $3-$10 | How much each virality point adds |
| `purchasing_power_index` | Market-specific | 0.1-1.0 | Regional affordability |
| `agent_modifier` | 0 | -0.2 to +0.2 | Local market knowledge |
| `cashback_premium` | 0.12 (12%) | 0.10-0.15 | Funds the cashback incentive |

---

## 2. Purchasing Power Parity (PPP) Reference Data

### Source: World Bank International Comparison Program

PPP values based on **2023 World Bank price level indices**, adjusted for digital goods:

| Country | ISO | PPP Index | Rationale |
|---------|-----|-----------|-----------|
| United States | US | 1.00 | Baseline |
| United Kingdom | GB | 0.92 | High income, strong currency |
| Germany | DE | 0.88 | High income EU |
| France | FR | 0.85 | High income EU |
| Spain | ES | 0.70 | Medium-high income EU |
| Mexico | MX | 0.40 | Medium income Americas |
| Brazil | BR | 0.35 | Medium income Americas |
| Indonesia | ID | 0.25 | Large medium income Asia |
| India | IN | 0.22 | Large lower-middle income Asia |
| Philippines | PH | 0.28 | Medium income Asia |
| Vietnam | VN | 0.24 | Lower-middle income Asia |
| Thailand | TH | 0.32 | Upper-middle income Asia |
| Nigeria | NG | 0.18 | Lower-middle income Africa |
| Egypt | EG | 0.20 | Lower-middle income MENA |
| Turkey | TR | 0.30 | Upper-middle income |
| Poland | PL | 0.55 | Medium income EU |
| Colombia | CO | 0.32 | Medium income Americas |
| Argentina | AR | 0.28 | Medium income Americas (volatile) |

### PPP Implementation

```typescript
// Stored in market_contexts table
const marketPPP: Record<string, number> = {
  US: 1.00, GB: 0.92, DE: 0.88, FR: 0.85, ES: 0.70,
  MX: 0.40, BR: 0.35, ID: 0.25, IN: 0.22, PH: 0.28,
  VN: 0.24, TH: 0.32, NG: 0.18, EG: 0.20, TR: 0.30,
  PL: 0.55, CO: 0.32, AR: 0.28
};

// Example: $55 USD concept price in different markets
// US: $55.00 × 1.00 = $55.00
// Mexico: $55.00 × 0.40 = $22.00
// Indonesia: $55.00 × 0.25 = $13.75
// India: $55.00 × 0.22 = $12.10
```

### Why Digital-Adjusted PPP?

Standard PPP is based on physical goods baskets. Digital goods have different considerations:
- No shipping costs
- Same production cost regardless of buyer location
- Local willingness to pay for digital content differs from physical goods

We use **0.8× the standard PPP ratio** for emerging markets to account for:
- Higher digital literacy among target buyers (skews more affluent)
- Content creators often have some disposable income
- Digital content valued more than PPP would suggest in these markets

---

## 3. Implementation

### TypeScript Interface

```typescript
interface PricingConfig {
  // Base configuration
  basePrice: number;           // Default: 5
  viralityMultiplier: number;  // Default: 5
  cashbackPremium: number;     // Default: 0.12
  
  // Constraints
  minPrice: number;            // Default: 1.00
  maxPrice: number;            // Default: 500.00
  maxAgentModifier: number;    // Default: 0.20
}

interface PricingInput {
  viralityScore: number;          // 0-10 from model
  purchasingPowerIndex: number;   // From market_contexts
  agentModifier?: number;         // Optional ±20% adjustment
  modelConfidence?: number;       // 0-1, displayed but doesn't affect price
  config?: Partial<PricingConfig>;
}

interface PricingOutput {
  basePrice: number;              // Before any adjustments
  viralityAdjusted: number;       // After virality
  pppAdjusted: number;            // After PPP
  agentAdjusted: number;          // After agent modifier
  listedPrice: number;            // Final price with premium
  cashbackAmount: number;         // What buyer gets back (10-15%)
  cashbackMinimum: number;        // 10% of pre-premium price
  cashbackMaximum: number;        // 15% of pre-premium price
  modelConfidence: number;        // Pass-through for display
  breakdown: PricingBreakdown;    // Detailed calculation
}

interface PricingBreakdown {
  steps: {
    step: string;
    value: number;
    explanation: string;
  }[];
}
```

### Complete Implementation

```typescript
const DEFAULT_CONFIG: PricingConfig = {
  basePrice: 5,
  viralityMultiplier: 5,
  cashbackPremium: 0.12,
  minPrice: 1.00,
  maxPrice: 500.00,
  maxAgentModifier: 0.20
};

function calculatePrice(input: PricingInput): PricingOutput {
  const config = { ...DEFAULT_CONFIG, ...input.config };
  const { viralityScore, purchasingPowerIndex, agentModifier = 0, modelConfidence = 0.5 } = input;
  
  const steps: PricingBreakdown['steps'] = [];
  
  // Step 1: Base price
  const basePrice = config.basePrice;
  steps.push({
    step: 'Base Price',
    value: basePrice,
    explanation: `Starting point: $${basePrice.toFixed(2)}`
  });
  
  // Step 2: Virality adjustment
  const viralityBonus = viralityScore * config.viralityMultiplier;
  const viralityAdjusted = basePrice + viralityBonus;
  steps.push({
    step: 'Virality Adjustment',
    value: viralityAdjusted,
    explanation: `${viralityScore.toFixed(1)} × $${config.viralityMultiplier} = +$${viralityBonus.toFixed(2)}`
  });
  
  // Step 3: PPP adjustment
  const pppAdjusted = viralityAdjusted * purchasingPowerIndex;
  steps.push({
    step: 'PPP Adjustment',
    value: pppAdjusted,
    explanation: `$${viralityAdjusted.toFixed(2)} × ${purchasingPowerIndex.toFixed(2)} PPP = $${pppAdjusted.toFixed(2)}`
  });
  
  // Step 4: Agent modifier (clamped)
  const clampedModifier = Math.max(
    -config.maxAgentModifier, 
    Math.min(config.maxAgentModifier, agentModifier)
  );
  const agentAdjusted = pppAdjusted * (1 + clampedModifier);
  steps.push({
    step: 'Agent Modifier',
    value: agentAdjusted,
    explanation: `$${pppAdjusted.toFixed(2)} × ${(1 + clampedModifier).toFixed(2)} = $${agentAdjusted.toFixed(2)}`
  });
  
  // Step 5: Cashback premium
  const withPremium = agentAdjusted * (1 + config.cashbackPremium);
  steps.push({
    step: 'Cashback Premium',
    value: withPremium,
    explanation: `$${agentAdjusted.toFixed(2)} × ${(1 + config.cashbackPremium).toFixed(2)} = $${withPremium.toFixed(2)}`
  });
  
  // Step 6: Clamp to min/max
  const listedPrice = Math.max(
    config.minPrice,
    Math.min(config.maxPrice, Math.round(withPremium * 100) / 100)
  );
  
  if (listedPrice !== withPremium) {
    steps.push({
      step: 'Price Clamping',
      value: listedPrice,
      explanation: `Clamped to $${config.minPrice}-$${config.maxPrice} range`
    });
  }
  
  // Calculate cashback
  const cashbackAmount = Math.round(listedPrice * (config.cashbackPremium / (1 + config.cashbackPremium)) * 100) / 100;
  
  return {
    basePrice,
    viralityAdjusted,
    pppAdjusted,
    agentAdjusted,
    listedPrice,
    cashbackAmount,
    breakdown: { steps }
  };
}

function calculateCashback(pricePaid: number, premium: number = 0.12): number {
  // Cashback is the premium portion of the price
  // If price = base × 1.12, then cashback = price × (0.12/1.12) = price × 0.107
  return Math.round(pricePaid * (premium / (1 + premium)) * 100) / 100;
}
```

---

## 3. Example Calculations

### Example 1: High-Virality Concept in US Market

```
Input:
  - virality_score: 8.5
  - purchasing_power_index: 1.0 (US)
  - agent_modifier: 0 (no adjustment)

Calculation:
  1. Base: $5.00
  2. Virality: $5 + (8.5 × $5) = $5 + $42.50 = $47.50
  3. PPP: $47.50 × 1.0 = $47.50
  4. Agent: $47.50 × 1.0 = $47.50
  5. Premium: $47.50 × 1.12 = $53.20

Output:
  - listed_price: $53.20
  - cashback_amount: $5.70
```

### Example 2: Same Concept in Indonesia

```
Input:
  - virality_score: 8.5
  - purchasing_power_index: 0.25 (Indonesia)
  - agent_modifier: 0

Calculation:
  1. Base: $5.00
  2. Virality: $5 + (8.5 × $5) = $47.50
  3. PPP: $47.50 × 0.25 = $11.88
  4. Agent: $11.88 × 1.0 = $11.88
  5. Premium: $11.88 × 1.12 = $13.30

Output:
  - listed_price: $13.30
  - cashback_amount: $1.42
```

### Example 3: Agent Increases Price (Local Trend Signal)

```
Input:
  - virality_score: 6.0
  - purchasing_power_index: 0.40 (Mexico)
  - agent_modifier: +0.15 (agent believes concept is trending locally)

Calculation:
  1. Base: $5.00
  2. Virality: $5 + (6.0 × $5) = $35.00
  3. PPP: $35.00 × 0.40 = $14.00
  4. Agent: $14.00 × 1.15 = $16.10
  5. Premium: $16.10 × 1.12 = $18.03

Output:
  - listed_price: $18.03
  - cashback_amount: $1.93
```

### Example 4: Low-Virality Minimum Price

```
Input:
  - virality_score: 1.5
  - purchasing_power_index: 0.18 (India)
  - agent_modifier: 0

Calculation:
  1. Base: $5.00
  2. Virality: $5 + (1.5 × $5) = $12.50
  3. PPP: $12.50 × 0.18 = $2.25
  4. Agent: $2.25 × 1.0 = $2.25
  5. Premium: $2.25 × 1.12 = $2.52

Output:
  - listed_price: $2.52
  - cashback_amount: $0.27

Note: Above minimum price ($1.00), so no clamping needed.
```

---

## 4. Virality Score Calculation

The virality score (0-10) comes from the trained preference model.

### How It's Calculated

```typescript
interface ViralityCalculation {
  // From model
  rawScore: number;             // Direct model output (0-1 scale)
  confidence: number;           // Model confidence (0-1)
  
  // Converted
  viralityScore: number;        // rawScore × 10 (0-10 scale)
  
  // Explanation
  topPositiveFactors: string[]; // Features pushing score up
  topNegativeFactors: string[]; // Features pushing score down
}

function calculateViralityScore(features: DeepAnalysis, model: ModelVersion): ViralityCalculation {
  const weights = model.feature_weights;
  let rawScore = 0;
  const positiveFactors: { feature: string; contribution: number }[] = [];
  const negativeFactors: { feature: string; contribution: number }[] = [];
  
  // Apply each weight
  for (const [featurePath, weight] of Object.entries(weights)) {
    const featureValue = getNestedValue(features, featurePath);
    
    if (featureValue === undefined) continue;
    
    // Normalize feature value to 0-1 range
    const normalizedValue = normalizeFeature(featurePath, featureValue);
    const contribution = normalizedValue * weight;
    
    rawScore += contribution;
    
    if (contribution > 0.05) {
      positiveFactors.push({ feature: featurePath, contribution });
    } else if (contribution < -0.05) {
      negativeFactors.push({ feature: featurePath, contribution });
    }
  }
  
  // Clamp to 0-1
  rawScore = Math.max(0, Math.min(1, rawScore));
  
  return {
    rawScore,
    confidence: calculateConfidence(features, model),
    viralityScore: rawScore * 10,
    topPositiveFactors: positiveFactors
      .sort((a, b) => b.contribution - a.contribution)
      .slice(0, 5)
      .map(f => f.feature),
    topNegativeFactors: negativeFactors
      .sort((a, b) => a.contribution - b.contribution)
      .slice(0, 5)
      .map(f => f.feature)
  };
}
```

### Example Virality Calculation

```
Video Features:
  - script.replicability.score: 9 (normalized: 0.9)
  - trends.memeDependent: false (normalized: 0.0)
  - casting.actingSkillRequired: 3 (normalized: 0.3)
  - production.shotComplexity: 2 (normalized: 0.2)
  - comedyStyle.contrastMechanism.present: true (normalized: 1.0)

Model Weights:
  - script.replicability.score: +0.25
  - trends.memeDependent: -0.20
  - casting.actingSkillRequired: -0.15
  - production.shotComplexity: -0.10
  - comedyStyle.contrastMechanism.present: +0.12

Calculation:
  (0.9 × 0.25) + (0.0 × -0.20) + (0.3 × -0.15) + (0.2 × -0.10) + (1.0 × 0.12)
  = 0.225 + 0 - 0.045 - 0.02 + 0.12
  = 0.28 (base contribution from these 5 features)
  + contributions from other 45+ features
  = 0.72 (final raw score)

virality_score = 0.72 × 10 = 7.2
```

---

## 5. Purchasing Power Index

### Data Sources

The PPP index can come from:
1. **World Bank Data**: Official PPP conversion factors
2. **Big Mac Index**: Intuitive, updated regularly
3. **Manual Tiers**: Simplified categorization

### Recommended Values (Based on World Bank 2024)

| Market | Country Code | PPP Index | Rationale |
|--------|--------------|-----------|-----------|
| United States | US | 1.00 | Baseline |
| United Kingdom | GB | 0.85 | Slightly lower |
| Germany | DE | 0.90 | Strong economy |
| France | FR | 0.85 | Similar to UK |
| Spain | ES | 0.70 | Lower cost of living |
| Mexico | MX | 0.40 | Emerging market |
| Brazil | BR | 0.35 | Emerging market |
| Indonesia | ID | 0.25 | Large gap |
| Philippines | PH | 0.20 | Similar to Indonesia |
| India | IN | 0.18 | Largest gap |

### Updating PPP Values

```typescript
// Quarterly update recommended
async function updatePPPValues() {
  const worldBankData = await fetch('https://api.worldbank.org/v2/country/all/indicator/PA.NUS.PPP');
  
  // US PPP factor as baseline
  const usPPP = worldBankData.find(d => d.country.id === 'US').value;
  
  for (const country of worldBankData) {
    const pppIndex = country.value / usPPP;
    
    await db.marketContexts.update({
      where: { country_code: country.country.id },
      data: { purchasing_power_index: pppIndex }
    });
  }
}
```

---

## 6. Agent Modifier

### When Agents Adjust Price

Agents (data input agents, not buyers) can adjust price ±20% based on:

1. **Local Trend Signal**: Concept matches current local trend
2. **Cultural Fit**: Concept particularly resonates with local culture
3. **Competition**: Similar concepts already popular locally
4. **Seasonality**: Holiday-specific content
5. **Market Saturation**: Many similar concepts already sold

### Implementation

```typescript
interface AgentPriceAdjustment {
  modifier: number;        // -0.20 to +0.20
  reason: AgentAdjustmentReason;
  notes?: string;
}

enum AgentAdjustmentReason {
  LOCAL_TREND = 'local_trend',
  CULTURAL_FIT = 'cultural_fit',
  HIGH_COMPETITION = 'high_competition',
  LOW_COMPETITION = 'low_competition',
  SEASONAL = 'seasonal',
  MARKET_SATURATION = 'market_saturation'
}

function applyAgentModifier(
  basePrice: number, 
  adjustment: AgentPriceAdjustment
): number {
  const clampedModifier = Math.max(-0.20, Math.min(0.20, adjustment.modifier));
  
  // Log adjustment for analysis
  logAgentAdjustment({
    basePrice,
    modifier: clampedModifier,
    reason: adjustment.reason,
    notes: adjustment.notes,
    adjustedPrice: basePrice * (1 + clampedModifier)
  });
  
  return basePrice * (1 + clampedModifier);
}
```

### Adjustment Guidelines

| Reason | Typical Modifier | Example |
|--------|------------------|---------|
| Strong local trend match | +15% to +20% | Concept matches trending challenge |
| Good cultural fit | +5% to +10% | Humor style matches local preference |
| High local competition | -10% to -15% | Many similar concepts available |
| Low local competition | +5% to +10% | Unique concept for market |
| Seasonal relevance | +10% to +20% | Holiday-specific at right time |
| Market saturation | -15% to -20% | Market has seen too many similar |

---

## 7. Cashback Premium

### Why 12%?

```
Goal: Incentivize buyers to submit produced content
Trade-off: Higher premium = higher initial price = fewer sales

Analysis:
- 10% premium: Cashback feels minimal, low incentive
- 12% premium: Meaningful cashback, acceptable price increase
- 15% premium: Strong incentive, but price may deter buyers

Decision: 12% balances incentive with conversion
```

### How It Works

```
Scenario:
  - Base price after all adjustments: $10.00
  - With 12% premium: $10.00 × 1.12 = $11.20
  
  Buyer pays: $11.20
  If buyer submits produced content: Gets back $1.20
  Net price for buyer: $10.00
  
  Revenue breakdown:
  - If cashback claimed: Platform keeps $10.00
  - If cashback not claimed: Platform keeps $11.20
```

### Cashback Expiration

```typescript
const CASHBACK_EXPIRATION_DAYS = 30;

async function checkCashbackExpiration() {
  const expirationDate = new Date();
  expirationDate.setDate(expirationDate.getDate() - CASHBACK_EXPIRATION_DAYS);
  
  await db.transactions.updateMany({
    where: {
      cashback_status: 'pending',
      purchased_at: { lt: expirationDate },
      // No produced_content submitted
      produced_content: null
    },
    data: { cashback_status: 'expired' }
  });
}
```

---

## 8. Price Display Considerations

### Currency Conversion

```typescript
interface DisplayPrice {
  amount: number;
  currency: string;
  formatted: string;
  usdEquivalent: number;
}

async function getDisplayPrice(
  listedPriceUsd: number, 
  marketContext: MarketContext
): Promise<DisplayPrice> {
  const exchangeRate = await getExchangeRate('USD', marketContext.currency_code);
  const localAmount = listedPriceUsd * exchangeRate;
  
  return {
    amount: localAmount,
    currency: marketContext.currency_code,
    formatted: formatCurrency(localAmount, marketContext.currency_code),
    usdEquivalent: listedPriceUsd
  };
}

function formatCurrency(amount: number, currency: string): string {
  const formatters: Record<string, Intl.NumberFormat> = {
    USD: new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD' }),
    EUR: new Intl.NumberFormat('de-DE', { style: 'currency', currency: 'EUR' }),
    GBP: new Intl.NumberFormat('en-GB', { style: 'currency', currency: 'GBP' }),
    IDR: new Intl.NumberFormat('id-ID', { style: 'currency', currency: 'IDR' }),
    BRL: new Intl.NumberFormat('pt-BR', { style: 'currency', currency: 'BRL' }),
    MXN: new Intl.NumberFormat('es-MX', { style: 'currency', currency: 'MXN' }),
    INR: new Intl.NumberFormat('en-IN', { style: 'currency', currency: 'INR' }),
  };
  
  return formatters[currency]?.format(amount) || `${currency} ${amount.toFixed(2)}`;
}
```

### Example Display Prices

| Market | USD Price | Local Currency | Formatted |
|--------|-----------|----------------|-----------|
| US | $13.30 | $13.30 USD | $13.30 |
| UK | $11.31 | £9.05 GBP | £9.05 |
| Indonesia | $3.33 | 51,500 IDR | Rp 51.500 |
| Brazil | $4.66 | R$23.30 BRL | R$ 23,30 |
| Mexico | $5.32 | $92.00 MXN | $92.00 |
| India | $2.39 | ₹200 INR | ₹200.00 |

---

## 9. Pricing API

### Endpoint: Calculate Price

```typescript
// POST /api/pricing/calculate
interface CalculatePriceRequest {
  concept_id: string;
  market_id: string;
  agent_modifier?: number;
}

interface CalculatePriceResponse {
  concept_id: string;
  market_id: string;
  pricing: PricingOutput;
  display: DisplayPrice;
  cashback: {
    amount: number;
    percentage: number;
    expiration_days: number;
  };
}
```

### Endpoint: Bulk Calculate

```typescript
// POST /api/pricing/bulk
interface BulkCalculateRequest {
  concept_ids: string[];
  market_id: string;
}

interface BulkCalculateResponse {
  results: {
    concept_id: string;
    virality_score: number;
    listed_price: number;
    display_price: DisplayPrice;
  }[];
}
```

---

## 10. Pricing Analytics

### Track Conversion by Price Point

```typescript
interface PricingAnalytics {
  priceRange: string;         // '$0-5', '$5-10', etc.
  listingCount: number;
  salesCount: number;
  conversionRate: number;
  averageTimeToSale: number;  // Hours
}

async function getPricingAnalytics(marketId: string): Promise<PricingAnalytics[]> {
  const ranges = [
    { min: 0, max: 5, label: '$0-5' },
    { min: 5, max: 10, label: '$5-10' },
    { min: 10, max: 20, label: '$10-20' },
    { min: 20, max: 50, label: '$20-50' },
    { min: 50, max: 999, label: '$50+' }
  ];
  
  return Promise.all(ranges.map(async range => {
    const listings = await db.listingWindows.findMany({
      where: {
        market_context_id: marketId,
        listed_price: { gte: range.min, lt: range.max }
      },
      include: { transactions: true }
    });
    
    const salesCount = listings.reduce((sum, l) => sum + l.sold_count, 0);
    
    return {
      priceRange: range.label,
      listingCount: listings.length,
      salesCount,
      conversionRate: listings.length > 0 ? salesCount / listings.length : 0,
      averageTimeToSale: calculateAverageTimeToSale(listings)
    };
  }));
}
```

---

## Related Documents

- [Model Training Deep Dive](./06_MODEL_TRAINING.md) - How virality score is calculated
- [Market Contexts Deep Dive](./11_MARKET_CONTEXTS.md) - PPP values and market data
- [Cashback Flow Deep Dive](./10_CASHBACK_FLOW.md) - How cashback is processed

---

*This document provides exhaustive detail on pricing logic. Refer to specific component documents for related systems.*
