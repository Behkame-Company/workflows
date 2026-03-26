# Spec Sizing & Scope

> Right-sizing specifications for features, not for entire systems.

---

## The Goldilocks Problem

Specs fail when they're the wrong size:

```
Too Small:  "Build auth" → Agent makes wrong assumptions
Too Large:  500-line spec for everything → Agent loses context
Just Right: Feature-level spec with clear boundaries → Agent succeeds
```

---

## Sizing Guidelines

### By Feature Complexity

| Complexity | Spec Length | Time to Write | Example |
|-----------|------------|---------------|---------|
| **Trivial** | Skip spec | 0 min | Fix typo, rename variable |
| **Small** | 10–20 lines | 5–10 min | Add a button, change validation |
| **Medium** | 30–80 lines | 15–30 min | New API endpoint, form component |
| **Large** | 80–200 lines | 30–60 min | Complete feature (CRUD + UI) |
| **Epic** | Split into multiple specs | 1–2 hours | System redesign, new subsystem |

### When to Skip the Spec
- Bug fixes with obvious cause and solution
- Dependency updates
- Style/formatting changes
- Configuration tweaks
- Refactoring with no behavior change

### When You MUST Write a Spec
- New user-facing features
- Changes that affect multiple components
- Work that multiple developers will touch
- Features with compliance or audit requirements
- Changes that affect public APIs

---

## Scope Decomposition

### Breaking Down Epics

Large features should be decomposed into independently specifiable units:

```
Epic: "E-commerce Shopping Cart"
  ├── Spec 1: Cart data model and basic CRUD
  ├── Spec 2: Add/remove items from cart
  ├── Spec 3: Cart quantity updates and validation
  ├── Spec 4: Price calculation with discounts
  ├── Spec 5: Cart persistence (logged-in vs guest)
  └── Spec 6: Checkout initiation from cart
```

### The "Can It Ship Independently?" Test
Each spec should describe something that could theoretically be shipped alone:
```
✅  "Cart CRUD operations" — ships as API foundation
✅  "Add to cart button" — ships as a user-facing feature
❌  "Half of the checkout flow" — incomplete, can't verify
```

### Vertical Slices vs Horizontal Layers

```
❌  Horizontal (don't do this):
  Spec 1: "All database tables for cart"
  Spec 2: "All API endpoints for cart"
  Spec 3: "All UI components for cart"

✅  Vertical (do this):
  Spec 1: "Add item to cart (DB + API + UI)"
  Spec 2: "View cart contents (DB + API + UI)"
  Spec 3: "Update quantities (DB + API + UI)"
```

---

## The Scope Section

### Template

```markdown
## Scope

### In Scope
- [Specific capability 1]
- [Specific capability 2]
- [Specific capability 3]

### Out of Scope
- [Thing that might seem in scope but isn't] — reason/timeline
- [Related feature not included] — planned for [when]
- [Edge case not handled] — tracked as [issue]

### Assumptions
- [Infrastructure assumption]: We have [X] available
- [Data assumption]: [Y] already exists in the database
- [Timeline assumption]: This ships before [Z] feature
```

### Example

```markdown
## Scope

### In Scope
- Users can add products to cart from product detail page
- Cart shows item count in header badge
- Cart page displays all items with quantities and subtotals
- Users can update quantities (1-99) or remove items
- Cart persists in database for logged-in users

### Out of Scope
- Guest cart (localStorage) — Spec 5 in cart epic
- Discount/coupon codes — separate spec
- Saved for later / wishlist — v2 feature
- Cart sharing via link — not planned
- Real-time inventory checking — will use eventual consistency

### Assumptions
- Product catalog API exists and returns price data
- User authentication is in place (NextAuth)
- PostgreSQL database is provisioned with Prisma ORM
```

---

## Scope Creep Prevention

### The "Out of Scope" Power Move

Explicitly listing out-of-scope items prevents three problems:

1. **Developer assumption**: "They probably want X too" → builds unrequested features
2. **AI assumption**: "A cart system should include Y" → over-implements
3. **Stakeholder assumption**: "Surely Z was included" → disappointed at review

### During Implementation
When an AI agent or developer encounters something that *should* be done but is out of scope:

```markdown
## Discovered During Implementation
- [ ] Cart needs real-time stock validation — defer to inventory spec
- [ ] Price rounding rules need clarification — created QUESTION-1
```

Document it, don't implement it. Create a new spec or add to backlog.

---

## Context Window Considerations

AI agents have limited context windows. Spec sizing should account for this:

```
Agent context budget:
  System instructions:     ~2,000 tokens
  Codebase context:        ~4,000 tokens
  Spec + Plan + Tasks:     ~3,000 tokens  ← Your budget
  Available for reasoning: ~7,000 tokens
  ─────────────────────────────────────
  Total:                   ~16,000 tokens
```

### Implications
- A 200-line spec may use ~2,000 tokens alone
- Include only relevant spec sections for each task
- Break large specs into independently referenceable sections

---

*Next: [The Iterative Spec →](../04-best-practices/the-iterative-spec.md)*
