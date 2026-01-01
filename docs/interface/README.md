# letrend Interface Documentation

> **Purpose**: Complete specification for the customer-facing interface
> **Status**: Draft - Awaiting Owner Input
> **Created**: January 1, 2026

---

## Document Index

| Document | Purpose | Status |
|----------|---------|--------|
| [CUSTOMER_INTERFACE_OUTLINE.md](../CUSTOMER_INTERFACE_OUTLINE.md) | High-level overview and wireframes | Draft |
| [01_USER_FLOWS.md](./01_USER_FLOWS.md) | Step-by-step user journeys | Draft |
| [02_COMPONENTS.md](./02_COMPONENTS.md) | UI component specifications | Draft |
| [03_INFORMATION_ARCHITECTURE.md](./03_INFORMATION_ARCHITECTURE.md) | What info appears where and why | Draft |
| [04_EDGE_CASES.md](./04_EDGE_CASES.md) | Error handling and edge cases | Draft |

---

## Reading Order

1. **Start with** [CUSTOMER_INTERFACE_OUTLINE.md](../CUSTOMER_INTERFACE_OUTLINE.md) for the big picture
2. **Then** [01_USER_FLOWS.md](./01_USER_FLOWS.md) to understand user journeys
3. **Reference** other docs as needed for specific details

---

## Decision Points Summary

These documents contain decision points marked **[DECISION NEEDED]**. Summary of open decisions:

### From User Flows
| Decision | Options | Location |
|----------|---------|----------|
| Show prices to non-logged-in users? | Yes / No / "From $X" | Flow 1 |
| Infinite scroll vs pagination? | Infinite / Paginated | Flow 1 |
| Video preview type | 5-sec loop / Blurred / Static | Flow 2 |
| Checkout page vs modal? | Dedicated page / Modal | Flow 3 |
| Allow cashback resubmission after rejection? | Yes / No | Flow 6 |

### From Components
| Decision | Options | Location |
|----------|---------|----------|
| Video preview approach | 5-sec loop / Blurred / Static | VideoPreview |
| Component library | Tailwind / shadcn/ui / Radix | General |

### From Edge Cases
| Decision | Options | Location |
|----------|---------|----------|
| Unsupported market handling | Block / Default USD | EC-4 |
| Refund policy | No refunds / Time-limited / Manual | EC-11 |
| Resubmission after failed URL check? | Yes / No | EC-16 |
| Same video for multiple cashbacks? | Allow / Block / Review | EC-22 |
| Account deletion policy | Hard / Soft / Prevent | EC-23 |
| Minimum price floor | $0 / $1 / Configurable | EC-29 |

---

## Next Steps

1. **Owner Review**: Review all documents, make decisions on open points
2. **Owner Input**: Provide additional requirements and adjustments
3. **Wireframes**: Create visual wireframes based on finalized specs
4. **API Contracts**: Define API endpoints between frontend and backend
5. **Implementation**: Begin building

---

## Relationship to Other Documentation

```
hagen_pg (letrend)
├── docs/
│   ├── MVP_MASTER_SPECIFICATION.md    ← System architecture (backend)
│   ├── CUSTOMER_INTERFACE_OUTLINE.md  ← Interface overview
│   ├── interface/                      ← This folder
│   │   ├── README.md                   ← You are here
│   │   ├── 01_USER_FLOWS.md
│   │   ├── 02_COMPONENTS.md
│   │   ├── 03_INFORMATION_ARCHITECTURE.md
│   │   └── 04_EDGE_CASES.md
│   └── components/                     ← Backend component specs
│       └── 01-11_*.md

hagen-main
├── src/                               ← Actual codebase
└── planning_docs/                     ← Symlink to hagen_pg
```

---

*This is the living specification for the letrend customer interface.*
