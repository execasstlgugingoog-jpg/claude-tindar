# TINDAR — Master Architecture Document
## Single Source of Truth (SOT) — Version 3.0

**Status:** LOCKED — All decisions final. Ready for Implementation Plan.
**Date:** March 2026
**Build Scope:** Free Tier + Pro Tier | Forward-compatible to all upper tiers
**Teams:** Windsurf (Cloud/Backend/Dashboard) · Antigravity (PWA/Offline/Hardware)
**Gatekeeper:** Agent 1 / Master AI — sole authority over schema, shared package, and all deviation sign-offs

> This document consolidates and supersedes:
> TINDAR Final Architecture Documentation v1 (ADRs #1–12)
> Architecture Supplement v1 (Consultant 1 — 13 gaps resolved)
> Architecture Supplement v1.1 (Consultant 1 follow-up — 6 second-order risks resolved)
> Consultant 2 findings (4 gaps resolved — paged sync, privacy scrimming, hybrid PIN, jittered backoff)
> ADR #13 — Capital Withdrawal, Structured Expenses, Provisional P&L
> ADR #14 — Consignment Domain
> ADR #15 — Discounts & Promos
> ADR #16 — Suki Card Loyalty System
>
> No prior document has any authority. This document alone is the SOT.

---

## PART I — PRODUCT CONTEXT & STRATEGIC POSITIONING

---

### 1. Product Vision

TINDAR is the **operating system for Philippine micro-retail** — a mobile-first, offline-capable, GCash-native POS SaaS platform purpose-built for the 1.2 million sari-sari stores across the Philippines.

It is not a generic POS adapted for the Philippine market. It is infrastructure built from the ground up around Filipino sari-sari workflows, connectivity realities, and the economic structures unique to this market segment.

### 2. The Problem

Philippine sari-sari stores operate with zero software infrastructure. Sales are recorded in notebooks, GCash transactions are manually reconciled, utang (credit) is tracked informally, shrinkage and clerk leakage go undetected, and owners have no real-time visibility into their operations. Consignment goods from local suppliers are accepted and settled without records. Expenses like wages and utilities are drawn from undefined pools. Existing POS solutions (Loyverse, Square, StoreHub) are either too complex, not localized for the Philippines, or do not understand the GCash, utang, and consignment workflows central to sari-sari economics.

### 3. The Solution — Five Core Pillars

**Pillar 1 — Offline-First:** Guaranteed reliability in low-connectivity areas. The app is fully operational without internet. Local truth is always operational truth. The server never unilaterally rolls back local operations.

**Pillar 2 — Mobile-First (Android Priority):** Built for the hardware sari-sari owners already have. Low-end Android optimization. Minimum 48dp touch targets. High-contrast outdoor support. Initial JS bundle under 150KB gzipped.

**Pillar 3 — Tier-Unlock Architecture:** No migration required on upgrade. Data captured from day one across all tiers. Instant feature access upon tier change. Upper tier features are schema-ready hooks, never afterthoughts.

**Pillar 4 — GCash-Native:** GCash is treated as a separate micro-business inside the store, not merely a payment method. Separate ledger, separate P&L, separate reconciliation. This is TINDAR's primary competitive moat — no competitor models GCash this way.

**Pillar 5 — Scale-Ready:** Designed for multi-branch from day one. Every store document includes `organizationId`. Branch manager role is schema-ready. Upper tier unlock is always additive, never structural.

### 4. Target Market

**Primary:** Sari-sari store owners and neighborhood convenience shops (1.2M stores nationwide).
**Launch Region:** Northern Mindanao + Caraga (regional pilot — 15,700 target stores).
**Expansion:** 1% PH = 12,000 stores; 3% = 36,000 stores; then SEA (20M+ micro-retailers).

### 5. Competitive Positioning

| Category | Competitor | Gap TINDAR Fills |
|---|---|---|
| Generic POS | Loyverse | No GCash ledger, no utang intelligence, no consignment |
| Global Fintech | Square | Not Philippines-localized |
| SEA POS | StoreHub | Not sari-sari focused |
| Manual | Notebook | No reporting, no accountability, no control |

**TINDAR Exclusive Advantages:**
- Native GCash 4-pool ledger (protected IP — Gatekeeper Rule 7)
- Native utang intelligence with credit limits and aging
- Full consignment lifecycle with receipts and supplier ledger
- Capital withdrawal and structured expense tracking with per-domain P&L
- Provisional P&L per business unit (Store, GCash, Consolidated)
- Discount and promo engine with policy-controlled limits
- Suki Card loyalty system (upper tier)
- Hybrid offline reliability with deterministic conflict resolution

### 6. Business Model — Subscription SaaS

| Tier | Price (PHP/mo) | Target Segment |
|---|---|---|
| Free | 0 | Micro stores — feeder funnel |
| Pro | 300 | Growing stores |
| Business | 600 | Higher volume |
| Advanced | 900 | 2-branch owners |
| Multi-Branch | 1,500 | 3 branches |
| Enterprise AI | 2,500 | 3–9 branches |

Weighted ARPU target: ₱500–₱600/month. Future: supplier marketplace, embedded lending, analytics.

---

## PART II — TECHNOLOGY STACK & MONOREPO STRUCTURE

---

### 7. Technology Stack

| Layer | Technology | Rationale |
|---|---|---|
| PWA Frontend | React + Vite | Antigravity team. Fast builds, modern DX. |
| Backend | Node.js REST Cloud Functions | Windsurf. Single Express function — shared warm instances. |
| Cloud Database | Firestore | Scalable, real-time, Firebase ecosystem. |
| Local Database | Dexie.js (IndexedDB) | Offline-first local persistence. |
| Auth | Firebase Auth | Google Sign-In for owner only. |
| Language | TypeScript — entire stack | Shared types across all packages. |
| Styling | Tailwind CSS | Utility-first, PurgeCSS at build. |
| Reactive State | Dexie `useLiveQuery` | Persisted data. Reactive queries on local DB. |
| UI State | Zustand | Cart, clerk session, non-persisted UI state. |
| Error Tracking | Sentry | Both PWA and Functions. |
| Date Library | date-fns | Tree-shakeable. NOT moment.js — ever. |
| i18n | react-i18next | Lazy-loaded language packs. |

### 8. Monorepo Structure

```
/tindar
  /packages
    /shared     ← Business logic, validators, TS types, abstract interfaces (Gatekeeper-owned)
    /ui         ← Design tokens, shared React components, Tailwind config, i18n locales
    /pwa        ← Antigravity: Dexie implementations, React UI, hardware integration
    /functions  ← Windsurf: Firestore Admin SDK, REST API, server-side logic
    /web        ← FUTURE: Upper tier web dashboard (same React + Vite stack)
```

### 9. Package Dependency Rules

**`/packages/shared`** — Imports NOTHING from Dexie, Firebase, or any runtime SDK. Zero external dependencies except TypeScript. Contains: validators, TS types, abstract repository interfaces, business logic, event type definitions, policy engine logic.

**`/packages/ui`** — Design tokens, shared React components, Tailwind config, i18n locale JSON files. Both `/pwa` and `/web` import from `/ui`. No raw Tailwind in team code — teams consume components only.

**`/packages/pwa`** — Owns ALL hardware integration code (printer, scanner, cash drawer). Dexie repository implementations. No hardware logic in `/shared` or `/functions`.

**`/packages/functions`** — Firestore Admin SDK implementations. REST API endpoints. Server-side-only logic (bcrypt hashing, JWT verification, audit writes, projection updates).

**Import rule:** `/pwa` and `/web` import from `/shared` and `/ui`. Neither `/shared` nor `/ui` imports from `/pwa`, `/functions`, or `/web`. Circular imports are a build-time error.

---

## PART III — SHARED BUSINESS LAYER & REPOSITORY PATTERN

---

### 10. Shared Business Layer

Every write operation passes through shared validation before touching Dexie or Firestore.

```
PWA input → /shared validator → PASS → Dexie write + sync queue → Cloud Function → Firestore
                              → FAIL → Rejected before any write
```

**What lives in `/packages/shared`:**
- Sale transaction validation (items, quantities, payment totals, discount validation)
- Inventory deduction logic and low stock checks
- GCash entry validation (4-pool float impact computation)
- Utang entry validation (balance, credit limit checks)
- Shift open/close validation and reconciliation math
- Expense validation (category, source pool, business domain, split ratio)
- Capital withdrawal validation (pool balance checks)
- Consignment lifecycle validation (batch integrity, settlement math)
- Discount and promo engine (promo condition evaluation)
- Provisional P&L computation logic
- Pricing and margin computation, PHP currency formatting
- Tier feature gate checks (UI-level only — not enforcement)
- All TypeScript entity types and event type definitions
- Abstract repository interfaces
- Policy engine authorization checks

**What stays backend-only (Cloud Functions):**
- JWT verification and authorization enforcement
- Sensitive operation execution (void, grant utang, apply discounts)
- Subscription and tier management
- Audit log writes
- PIN hashing (bcrypt — server-side only, never client-side)
- Projection updates (server-authoritative)
- Fraud baseline rule evaluation
- sortKey computation

**Golden Rule:** No business rule exists in only one place. Cloud Functions use the same validators the PWA used before sending the request.

### 11. Abstract Repository Interfaces

```typescript
IProductRepository          → getById, search, save, deduct, getConsignmentBatch
ITransactionRepository      → append, getByShift, getByDate
IShiftRepository            → open, close, getCurrent
ICustomerRepository         → getById, save, getUtangBalance, getBySukiCard
IGCashRepository            → append, getFloat, getByDate
IClerkRepository            → getByPin, save
ISyncQueueRepository        → enqueue, dequeue, markSynced, getByPriority
IConsignmentRepository      → receiveBatch, recordSale, returnBatch, settle
IExpenseRepository          → append, getByPeriod, getByDomain
ISupplierRepository         → getById, save, getActiveBatches
IPromoRepository            → getActive, evaluate, getById
```

Implementations: `/packages/pwa` → Dexie | `/packages/functions` → Firestore Admin SDK

---

## PART IV — AUTHENTICATION, RBAC & SECURITY

---

### 12. Authentication Model

#### Owner Authentication
- Method: Google Sign-In via Firebase Auth
- Trigger: One-time onboarding only
- Firebase Auth user is ALWAYS the owner — never a clerk
- Full logout: `auth.signOut()` + `clearCredentialState()` + `prompt: 'select_account'` (prevents Android Credential Manager auto-login bug)
- **Offline rule:** Never call `getIdToken()` in offline code paths. Cache `ownerUID` + auth state in Dexie. Full offline operation from cached credentials. Silent re-auth on reconnection only.

#### Clerk Authentication
- Method: Store PIN (4–6 digits minimum; 6 digits required for financial operations)
- Scope: Local app-layer session switch only — NOT Firebase Auth
- No Firebase Auth UID for clerks — ever
- Clerk switch: PIN entry closes current shift, opens new shift

#### Hybrid PIN Provisioning (Offline Clerk Add)
When a new clerk is added while the device is offline, the owner cannot wait for server-side bcrypt hashing:

```
OFFLINE CLERK PROVISIONING:

1. Owner adds new clerk offline
2. Client generates a temporary local PIN hash:
   tempHash = SHA-256(ownerUID + ":" + clerkId + ":" + PIN + ":" + localSalt)
   localSalt = crypto.getRandomValues(8 bytes) — stored in Dexie clerk record
3. Clerk can log in immediately using tempHash validation
4. CLERK_CREATED event queued for sync with raw status: 'TEMP_HASH'
5. On reconnection:
   Cloud Function receives CLERK_CREATED event
   Generates proper bcrypt hash server-side
   Writes bcrypt hash to /clerks/{clerkId}/private/auth
   Sets pinVersion = 1
   Returns confirmation
6. On next device sync:
   Dexie clerk record updated: pinHash = bcrypt hash, status = 'CONFIRMED'
   localSalt cleared
   tempHash discarded
7. From this point: standard PIN verification flow applies

Security note: tempHash is only valid for the offline window. It is replaced on first sync.
Compromise risk is bounded: attacker needs ownerUID + clerkId + localSalt + PIN — all local to device.
```

#### Auth Flow States

| State | Behavior |
|---|---|
| Cold start (Dexie cached) | Splash 3s → Offline Owner Mode |
| Cold start (no cache) | Splash → Google Sign-In onboarding |
| Token refresh | Invisible to active shift. Claims update at next shift start. |
| Device transfer | Empty Dexie + existing UID → full re-sync |
| Onboarding | 5-step atomic flow. Dexie progress flag by step ID. |

### 13. Role-Based Access Control (RBAC)

```
OWNER          → full access, all stores, all branches, all policies
BRANCH_MANAGER → full access to assigned branch (upper tier unlock, schema-ready now)
CLERK          → operational access per owner-configured permissions
```

Free and Pro: `BRANCH_MANAGER` exists in schema, never assigned. Owner is the effective branch manager.

**Enforcement:**
- Clerks have no Firebase Auth UID
- Owner JWT validates store ownership in Firestore rules + Cloud Functions
- Clerk RBAC lives in app-layer UI + Cloud Functions only — NOT Firestore rules
- Clerk permissions in `/stores/{storeId}/clerks/{clerkId}` (permission flags only — NO PIN hash)
- Sensitive writes route through Cloud Functions only — never direct Firestore client writes

**Offline clerk permissions:**
- Last synced permissions valid offline for standard operations
- Sensitive ops require connection (verified server-side): void sales, grant utang, apply discounts, capital withdrawal, consignment settlement

### 14. PIN Storage & Security

- Hashing: bcrypt — server-side only. Raw PIN never stored anywhere.
- Storage: Firestore `/stores/{storeId}/clerks/{clerkId}/private/auth` — deny-all client reads, Admin SDK only
- Local: Dexie stores `pinHash` + `pinVersion` (integer)
- Version logic: If `Firestore.pinVersion > Dexie.pinVersion` → local hash invalidated → sync required
- Offline: Last synced version valid
- Rate limiting: 5 attempts/minute, lockout after 10 failures (Cloud Functions middleware)

### 15. Full Security Stack

Every API request passes through seven layers:

```
1. HTTPS                → channel encryption
2. Firebase JWT         → owner identity verification
3. HMAC-SHA256          → request integrity + client authenticity
4. Timestamp window     → replay attack prevention (±5 minutes standard)
5. Body hash            → payload tamper detection
6. Rate limiting        → per store, per endpoint
7. clerkId in payload   → clerk action attribution
```

#### HMAC Signing (Per-Device Key)

```
signingKey = HMAC-SHA256(serverPepper, ownerUID + ":" + deviceId)
```

- Per-device key: compromise of one device does not expose others
- Signing token: short-lived, memory-only on PWA — never stored in Dexie
- Server pepper never leaves the server
- deviceId included in HMAC-signed payload (verified server-side)

#### Additional Security
- CSP: `script-src 'self'`, no `unsafe-inline`, restricted `connect-src`
- CORS: Restricted to PWA domain only — never `Access-Control-Allow-Origin: *`

---

## PART V — EVENT SOURCING, CQRS & DATA MODEL

---

### 16. Event Sourcing & CQRS Pattern

- Append-only event log (source of truth)
- CQRS projected state tables (fast read layer)
- Write path: new event + projection update in a single atomic Dexie transaction
- Projection is rebuildable from the event log at any time
- Firestore event log: **immutable, never pruned, never TTL'd** — policy is absolute

#### Complete Event Envelope

```typescript
{
  eventId: string,               // UUID, client-generated
  storeId: string,               // owner Firebase UID
  shiftId: string,
  clerkId: string,
  deviceId: string,              // registered device UUID
  localSequenceNumber: number,   // monotonic, per-device, never resets
  type: EventType,
  payload: object,               // domain-specific, typed per EventType
  timestamp: string,             // ISO 8601, device local time
  serverTime: string | null,     // set by Cloud Function at ingestion — authoritative
  sortKey: string | null,        // computed server-side at ingestion — ordering key
  syncStatus: 'PENDING' | 'SYNCED' | 'FAILED' | 'DEAD_LETTER',
  syncMethod: 'STANDARD' | 'EMERGENCY',  // EMERGENCY = corruption recovery upload
  syncAttempts: number,
  schemaVersion: number
}
```

#### sortKey Construction (Server-Side Only)

```typescript
function buildSortKey(event: TindarEvent): string {
  const t = event.serverTime ?? event.timestamp;
  const d = event.deviceId.padStart(36, '0');
  const s = String(event.localSequenceNumber).padStart(10, '0');
  return `${t}__${d}__${s}`;
}

// Ordering priority:
// 1. serverTime ASC      — authoritative wall clock
// 2. deviceId ASC        — deterministic tie-break across devices
// 3. localSequenceNumber ASC — causal order within same device
```

`sortKey` is set server-side only, never by client. Pending local events are sorted by `timestamp + localSequenceNumber` as ephemeral operational truth until synced.

#### Complete Event Type Taxonomy

```
Sales:
  SALE_COMPLETED          → standard sale, supports discounts and split payment
  SALE_VOIDED             → same-shift full reversal (compensating event)
  SALE_RETURNED           → any-shift partial or full return (compensating event)

Inventory:
  INVENTORY_ADJUSTED      → manual stock correction (positive or negative)
  PRODUCT_ADDED           → new product created
  PRODUCT_UPDATED         → product fields modified

GCash:
  GCASH_CASHIN            → customer deposits cash → wallet↓ cash↑
  GCASH_CASHOUT           → customer withdraws → wallet↑ cash↓
  GCASH_LOAD              → customer buys load → wallet↓ commission earned
  GCASH_TOPUP_STATION     → owner tops up at station → cash↓ wallet↑ fee→EXPENSE
  GCASH_TOPUP_DIRECT      → owner tops up wallet directly → wallet↑ (no cash move)

Financial:
  CASH_TRANSFER           → moves cash between store/GCash drawer allocations
  ALLOCATION_TRANSFER     → logical rebalance between allocations (no money moves)
  CAPITAL_INJECTION       → fresh capital into either business, either pool
  CAPITAL_WITHDRAWAL      → owner extracts profit/capital from a pool (equity event)
  CASH_DROP               → removes cash from drawer (policy-controlled)
  CASH_PICKUP             → removes cash from drawer (senior role minimum)
  CASH_VARIANCE           → auto-generated at shift close on discrepancy
  EXPENSE                 → structured operational expense with category + domain

Utang:
  UTANG_GRANTED           → credit extended to registered customer
  UTANG_PAYMENT           → customer pays outstanding balance
  UTANG_WRITTEN_OFF       → bad debt write-off (owner authorization)

Consignment:
  CONSIGNMENT_RECEIVED    → supplier drops batch — opens consignment ledger
  CONSIGNMENT_SALE        → unit sold from consignment batch (batchId-tagged)
  CONSIGNMENT_RETURNED    → supplier retrieves unsold units — clears ledger, no financial effect
  CONSIGNMENT_SETTLED     → store pays supplier for sold units — COGS recognized

Discounts:
  PROMO_ACTIVATED         → owner activates a promo configuration
  PROMO_DEACTIVATED       → promo deactivated (expired or manual)

Shifts:
  SHIFT_OPENED
  SHIFT_CLOSED

Clerks:
  CLERK_CREATED
  CLERK_PIN_CHANGED
  CLERK_DEACTIVATED

Store:
  STORE_ONBOARDED
  TIER_CHANGED
  POLICY_CHANGED          → logs old policy, new policy, changedBy, timestamp

Loyalty (upper tier):
  SUKI_ENROLLED           → customer enrolled in suki card program
  SUKI_POINTS_EARNED      → points credited on qualifying sale
  SUKI_POINTS_REDEEMED    → points redeemed as discount

System:
  DEVICE_REVOKED          → device revocation audit event
  PROJECTION_DIVERGENCE   → server/local projection mismatch detected
  PROJECTION_REBASED      → local projection overwritten from server snapshot
  EMERGENCY_SYNC          → marks batch uploaded via corruption recovery
```

#### Event Upcasting
`schemaVersion` field + migration map in `/packages/shared`. Old events remain valid at their original schema version. Migration is applied at read time, not at write time.

### 17. Complete Dexie Schema (v1)

Forward-compatible from day one. Version starts at **1**. Schema increments require Gatekeeper sign-off and a tested migration script. Goal: never need one after launch.

```
events
  eventId, storeId, shiftId, clerkId, deviceId, localSequenceNumber,
  type, payload, timestamp, serverTime, sortKey, syncStatus, syncMethod,
  syncAttempts, schemaVersion

products
  id, storeId, name, barcode, price, cost, stock, isInNegativeStock,
  lowStockThreshold, category, unitsPerBox, isActive, schemaVersion

customers
  id, storeId, name, phone, utangBalance, creditLimit,
  consentGiven, isActive, sukiCardNumber, pointsBalance,
  schemaVersion
  NOTE: Record created ONLY for utang customers (all tiers)
        OR suki enrollees (upper tier). Walk-in buyers = anonymous.

shifts
  id, storeId, clerkId, openTime, closeTime, openingBalance,
  closingBalance, totalSales, totalGCash, totalUtang,
  totalExpenses, totalWithdrawals, totalDiscounts,
  reconciliationStatus, schemaVersion

gcash_state
  id, storeId, shiftId,
  cashStore, cashGCash,       ← four logical pools
  walletStore, walletGCash,
  lastUpdated, schemaVersion

sync_queue
  id, eventId, priority, attempts, lastAttempt, nextAttempt,
  status, schemaVersion
  priority: 'HIGH' | 'NORMAL' | 'LOW'

dead_letters
  id, eventId, failureReason, createdAt, resolvedAt, schemaVersion

snapshots
  shiftId, timestamp, productsSnapshot, gcashSnapshot,
  customersSnapshot, schemaVersion

consignment_batches
  id, storeId, supplierId, productName, productId,
  receivedQty, soldQty, returnedQty, settledQty,
  supplierPrice, storePrice, status,
  receivedAt, settledAt, returnedAt,
  receivedBy, settledBy, returnedBy,
  sourcePool, schemaVersion
  status: 'OPEN' | 'PARTIALLY_SOLD' | 'SETTLED' | 'RETURNED'

suppliers
  id, storeId, name, phone, address, notes, isActive, schemaVersion

promos
  id, storeId, name, type, conditions, discountType, discountValue,
  productIds, minimumCartTotal, buyQty, getQty,
  startDate, endDate, activeDays, isActive, schemaVersion

expenses
  id, storeId, shiftId, amount, category, sourcePool,
  businessDomain, splitRatio, description, referenceDate,
  authorizedBy, eventId, schemaVersion

store_policies
  storeId, voidPolicy, returnPolicy, cashDropPolicy,
  cashPickupPolicy, discountPolicy, utangPolicy,
  expensePolicies, consignmentPolicy, shiftGuardEnabled,
  shiftGuardThreshold, schemaVersion

device_meta
  deviceId, storeId, role, status, registeredAt, lastSeenAt, schemaVersion
```

### 18. Firestore Collections (Complete)

```
/stores/{storeId}/
  ├── events/{eventId}                    ← IMMUTABLE, never pruned
  ├── registeredDevices/{deviceId}
  ├── revokedDevices/{deviceId}
  ├── policies/
  ├── clerks/{clerkId}/
  │     └── private/auth                  ← PIN hash, Admin SDK only
  ├── shiftFlags/{shiftId}
  ├── snapshots/{timestamp}
  ├── emergencyAuditLog/{callId}          ← Admin SDK only, deny-all client reads
  ├── suppliers/{supplierId}
  ├── consignmentBatches/{batchId}
  ├── promos/{promoId}
  └── projections/
        ├── products/{productId}
        ├── customers/{customerId}
        ├── gcash_state/current
        ├── shifts/{shiftId}
        ├── store_summary/current
        └── divergence_log/{timestamp}
```

**Firestore Security Rules principle:** Default deny-all. Owner reads own store projections directly. Cloud Functions (Admin SDK) write all transactional data. Monitor device reads `/projections/*` only (deviceRole custom claim enforced).

---

## PART VI — SYNC ARCHITECTURE & CONFLICT RESOLUTION

---

### 19. Conflict Resolution Policy

TINDAR's launch architecture (Free: 1 device, Pro: 1 primary + 1 read-only monitor) means true concurrent write conflict is not possible at launch. The monitor device is read-only by architecture and Cloud Function enforcement.

**Declared policy for all tiers including future upper:**

```
Server-Time Ordered, Last-Event-Wins, Delta-Based

Inventory domain:
  Overdraw ALLOWED — stock goes negative, surfaces as STOCK_ALERT, never blocks sale

Financial pools:
  All events are additive deltas, never absolute overwrites
  Delta events commute — order does not change final pool balance
  Overdraft ALLOWED — surfaces as POOL_ALERT, never blocks operation

Utang domain:
  Delta-based, commutative, never rejected

Tie-break (same serverTime — handled deterministically):
  1. serverTime ASC
  2. deviceId ASC (lexicographic)
  3. localSequenceNumber ASC
```

### 20. Sync Queue

**Strategy:** FIFO with priority tiers, exponential backoff, jittered reconnection, Dead Letter queue.

#### Priority Tiers

```
HIGH:   SHIFT_CLOSED, SHIFT_OPENED, CASH_VARIANCE, CONSIGNMENT_SETTLED
        Retry interval: 30 seconds
NORMAL: SALE_COMPLETED, GCASH_*, UTANG_*, EXPENSE, CAPITAL_WITHDRAWAL
        Retry interval: 4 minutes
LOW:    PRODUCT_UPDATED, CLERK_CREATED, PROMO_ACTIVATED
        Retry interval: 15 minutes
```

#### Retry Policy with Jittered Backoff

```
Standard retry (exponential):
  Attempt 1 → wait 1s
  Attempt 2 → wait 2s
  Attempt 3 → wait 4s
  Attempt 4 → wait 8s
  Attempt 5 → wait 16s → DEAD LETTER

Jittered reconnection (after regional outage):
  When connectivity restores after >5 min offline:
  Initial delay = random(0, 30) seconds before first sync attempt
  Rationale: prevents thundering herd when all devices in a region reconnect simultaneously
  This staggers Cloud Function load across a 30-second window
```

#### Paged Sync Upload

```
Batch upload policy:
  Maximum 50 events per sync request
  If sync_queue > 50 pending events: upload in sequential pages of 50
  Each page: full HMAC signing, idempotency check, projection update
  On partial failure: failed page retried independently; successful pages not repeated
  Page order: always sorted by priority DESC, then sortKey ASC within priority
  Rationale: prevents payload overflow on weak 3G/Edge after extended offline periods
             (e.g., store offline for a week — could accumulate 500+ events)
```

#### Dead Letter Handling
- Failed events → `dead_letters` table in Dexie
- Surfaced to owner as reconciliation task: "This transaction could not be synced. Confirm or discard."
- Projection rolled back only on explicit owner discard — never auto-rolled back by server
- Local truth = operational truth

#### Sync Triggers
- `visibilitychange` (app resume)
- `navigator.onLine` event
- Shift close (mandatory gate — see §21)
- 4-hour heartbeat (Free tier long-running sessions)
- No Web Background Sync API

### 21. Mandatory Shift Close Sync Gate

```
At shift close initiation (ALL TIERS):

1. App checks connectivity
   ONLINE:
     Execute sync attempt (blocks close UI, 30s timeout)
     On success → proceed to closing balance entry → confirm close
     On failure → warn owner → proceed anyway → sync retried on reconnect
   OFFLINE:
     Inform owner → proceed to closing balance entry → confirm close
     SHIFT_CLOSED event queued as HIGH priority

2. SHIFT_CLOSED event always written to Dexie regardless of sync outcome
3. Shift close is NEVER blocked by sync failure — only warned
```

### 22. Projection Rebase Policy

When server projection diverges from local projection beyond threshold:

```
REBASE_THRESHOLD (triggers full rebase):
  products.stock:          ±10 units OR ±20% (whichever smaller)
  customers.utangBalance:  ±₱100
  gcash_state any pool:    ±₱500

Below threshold: flag logged, operational continuity, no rebase

Rebase sequence (server always authoritative):
  STEP 1: Write PROJECTION_DIVERGENCE event to Firestore event log
  STEP 2: Pull server projections/*
  STEP 3: Overwrite Dexie projection tables field-by-field
          (events table and sync_queue NEVER touched)
  STEP 4: Notify owner (non-blocking badge)
  STEP 5: Write PROJECTION_REBASED event to event log
```

### 23. Corruption Recovery Protocol

```
STEP 1 — DETECT
  Startup: canary record check + event count vs localStorage counter
  On mismatch: set corruptionFlag = true in memory (NOT in Dexie)

STEP 2 — TRIAGE
  Read sync_queue from Dexie — identify PENDING and FAILED events

STEP 3 — EMERGENCY UPLOAD (if online)
  POST /api/v1/sync/emergency (±30min HMAC window, 5 calls/hour limit)
  syncMethod: 'EMERGENCY' tagged on all events
  All fraud baseline rules still apply — no bypass
  On success: mark events SYNCED

STEP 4 — SNAPSHOT
  Server generates /stores/{storeId}/snapshots/{timestamp}

STEP 5 — WIPE
  dexie.delete() + localStorage counters cleared

STEP 6 — REHYDRATE
  Pull products, customers, gcash_state, shifts (last 7) from projections/*
  Rebuild Dexie. New canary + reset counters.

STEP 7 — RESUME
  Owner notified: "Your device data was restored from cloud backup."

OFFLINE CORRUPTION:
  Steps 3–4 skipped. App → READ-ONLY until reconnect + rehydrate.
  POS operations BLOCKED. This is the only scenario that blocks POS.
```

---

## PART VII — DOMAIN MODELS

---

### 24. Shift Management

Shifts are first-class data entities captured from day one.

```typescript
{
  id, storeId, clerkId,
  openTime, closeTime,
  openingCashBalance, closingCashBalance,
  totalSales, totalGCash, totalUtang,
  totalExpenses, totalWithdrawals, totalDiscounts,
  reconciliationStatus: 'balanced' | 'discrepancy' | 'pending',
  schemaVersion
}
```

**Rules:**
- Shift ends on: explicit close, another clerk PIN entry, or owner device resume
- Prompt for closing cash balance before session closes
- Offline shifts written to Dexie, synced on reconnection

**Shift Close Reconciliation Formula:**

```
Expected closing balance =
    openingCashBalance
  + cash sales
  + GCash cashout received (physical cash in)
  - GCash cashin paid (physical cash out)
  - cash refunds issued
  - cash drops
  - cash pickups
  - cash expenses (sourcePool = CASH_STORE or CASH_GCASH)
  - cash withdrawals (sourcePool = CASH_STORE or CASH_GCASH)

Variance = actual counted − expected
```

**Tier Visibility:**

| Tier | Shift Data |
|---|---|
| Free | Captured silently — daily summary only |
| Pro | Per-shift summaries + discrepancy flags |
| Upper | Full shift reports, multi-clerk comparison, branch reconciliation |

### 25. Tingi / Stock Tracking

Stock always tracked at individual sellable unit level. No parent-child SKU relationship.

- `unitsPerBox` field: convenience multiplier for restocking entry only
- Bulk add: owner inputs quantity × unitsPerBox → system adds result to stock
- Box concept disappears after addition — only units exist in the ledger
- **Inventory overdraw:** ALLOWED. Stock goes negative. STOCK_ALERT flag set. Never blocks a sale.
- Upper tiers: Purchase Order domain fires `INVENTORY_ADJUSTED` on receipt

### 26. Split Payment Model

```typescript
payment: [
  { method: 'CASH',  amount: 100 },
  { method: 'GCASH', amount: 50  }
]
```

Total must equal or exceed transaction amount. Change computed from cash leg only. Forward-compatible: new payment methods = new enum value only.

### 27. GCash Domain Model — Protected Core IP

GCash is NOT integrated via API. All entries are manual. GCash operates as two completely separate domains:

**Domain 1 — GCash as Payment Method:** Customer pays for paninda using GCash. Recorded as payment leg on sale. Flows into store's wallet allocation.

**Domain 2 — GCash Subledger Business:** Separate cash-in/cash-out/load business with its own float, its own P&L, its own reconciliation.

#### Four Logical Financial Pools

```
CASH DRAWER
  CASH_STORE    → cash from paninda sales
  CASH_GCASH    → physical cash for GCash business operations

GCASH WALLET
  WALLET_STORE  → accumulated from customers paying sales via GCash
  WALLET_GCASH  → float for cash-in/cash-out/load operations
```

**Rules:**
- Single drawer mode (default) or split drawer mode — owner configures at onboarding
- Overdraw allowed on ALL four pools — never blocked
- Deficit surfaced immediately as POOL_ALERT flag
- GCash wallet total must always equal GCash app balance — the reconciliation anchor
- Owner rebalances via explicit `ALLOCATION_TRANSFER` (logical only, no money moves)

**GCash Event Pool Effects:**

```
GCASH_CASHIN:         WALLET_GCASH↓  CASH_GCASH↑
GCASH_CASHOUT:        WALLET_GCASH↑  CASH_GCASH↓
GCASH_LOAD:           WALLET_GCASH↓  commission→GCash P&L revenue
GCASH_TOPUP_STATION:  CASH_GCASH↓   WALLET_GCASH↑  fee→EXPENSE(GCASH_TOPUP_FEE)
GCASH_TOPUP_DIRECT:   WALLET_GCASH↑  (no cash movement)
CAPITAL_INJECTION:    target pool↑
CAPITAL_WITHDRAWAL:   source pool↓  (equity event — not P&L)
EXPENSE:              source pool↓  (P&L expense)
```

### 28. Capital Withdrawal

Owner extracts realized profit or capital from any pool. This is an **equity event** — it reduces pool balance but does NOT appear as a P&L expense. It is the mirror of `CAPITAL_INJECTION`.

```typescript
// CAPITAL_WITHDRAWAL payload
{
  amount: number,                   // always positive
  sourcePool: PoolIdentifier,       // CASH_STORE | CASH_GCASH | WALLET_STORE | WALLET_GCASH
  withdrawalType: 'CASH' | 'WALLET',
  note: string | null,
  authorizedBy: 'owner_pin'         // always owner-only, never delegatable
}
```

**Rules:**
- Overdraw allowed — POOL_ALERT flag if pool goes negative
- Owner PIN always required — cannot be delegated to clerk
- Included in shift close reconciliation formula
- Does NOT appear in P&L income statement

### 29. Structured Expense Domain

#### Expense Category Taxonomy

```typescript
type ExpenseCategory =
  | 'WAGES'           // salaries, daily wages, helpers
  | 'UTILITIES'       // electricity, water, internet, load
  | 'RENT'            // store space, stall, display
  | 'MAINTENANCE'     // repairs, servicing, upkeep
  | 'SUPPLIES'        // non-inventory operating supplies
  | 'MARKETING'       // flyers, signage, promotional materials
  | 'TRANSPORT'       // delivery, logistics, restocking travel
  | 'GCASH_TOPUP_FEE' // fee at loading station topup
  | 'TAX_FEES'        // barangay permits, government charges
  | 'OTHER'           // catch-all; requires non-empty description
```

#### Expense Event Payload

```typescript
{
  amount: number,
  category: ExpenseCategory,
  sourcePool: PoolIdentifier,       // which pool the money physically leaves
  businessDomain: 'STORE' | 'GCASH' | 'SHARED',
  splitRatio: { storePct: number, gcashPct: number } | null,  // only if SHARED
  description: string | null,       // required for 'OTHER'
  referenceDate: string,            // ISO date — may be after-the-fact
  authorizedBy: 'owner_pin' | 'manager_or_owner'
}
```

#### Expense Policy Engine

Owner configures default behavior per category. Stored in Dexie `store_policies`. Applied at policy engine level — clerk sees pre-filled defaults, owner can override per transaction.

```
Default policies (owner-customizable):
WAGES:           sourcePool=CASH_STORE  domain=SHARED    split=50/50
UTILITIES:       sourcePool=CASH_STORE  domain=SHARED    split=50/50
RENT:            sourcePool=CASH_STORE  domain=SHARED    split=50/50
MAINTENANCE:     sourcePool=CASH_STORE  domain=STORE     split=null
SUPPLIES:        sourcePool=CASH_STORE  domain=STORE     split=null
MARKETING:       sourcePool=CASH_STORE  domain=STORE     split=null
TRANSPORT:       sourcePool=CASH_STORE  domain=STORE     split=null
GCASH_TOPUP_FEE: sourcePool=CASH_GCASH  domain=GCASH     split=null
TAX_FEES:        sourcePool=CASH_STORE  domain=SHARED    split=50/50  ownerAuth=true
OTHER:           sourcePool=CASH_STORE  domain=STORE     split=null   ownerAuth=true
```

Policy changes take effect at next shift open (same rule as all policies).

### 30. Provisional P&L Engine

#### Design Philosophy
P&L is "provisional" because: (1) not all products may have `cost` populated — COGS is partial; (2) it reflects period-to-date state, not a closed accounting period. Missing cost data is flagged, never silently zeroed.

**Period options:** Shift | Day | Week | Month — owner selects.

#### P&L Structure

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
STORE BUSINESS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Revenue
  Gross Sales                      ₱ X
  Less: Returns & Voids           (₱ X)
  Less: Discounts Given           (₱ X)  ← revenue reduction, not expense
  Net Revenue                      ₱ X

Cost of Goods Sold
  COGS — tracked products         (₱ X)
  ⚠ N products have no cost set   [flag — COGS understated]
  Consignment Cost of Sales       (₱ X)  ← recognized at settlement only

Gross Profit                       ₱ X
Gross Margin %                     XX%

Operating Expenses
  [Category lines per domain attribution]
  Total Expenses                  (₱ X)

Net Provisional Profit             ₱ X

━━━━━━━━━━━━━━━━━━━━━━━━━━━━
GCASH BUSINESS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Revenue
  Cash-in Commission               ₱ X
  Cash-out Commission              ₱ X
  Load Commission                  ₱ X
  Total GCash Revenue              ₱ X

Operating Expenses
  GCash Topup Fees                (₱ X)
  [Shared expense allocations]    (₱ X)
  Total Expenses                  (₱ X)

Net Provisional Profit             ₱ X

━━━━━━━━━━━━━━━━━━━━━━━━━━━━
CONSOLIDATED
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Store Net Profit                   ₱ X
GCash Net Profit                   ₱ X
Total Net Provisional Profit       ₱ X

Notes:
  ⚠ Unsettled consignment liability: ₱ X  (future cost obligation)
  ⚠ Outstanding suki points liability: ₱ X  (upper tier only)
  ⚠ COGS data incomplete for N products
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**Tier access:**
- Free: "Did I make money today?" — single daily net figure
- Pro: Full 3-panel P&L per shift/day/week/month
- Upper: Cross-period trend, branch comparison, export

### 31. Customer & Utang System

**Customer record creation rule:**
A customer record is created ONLY when a trackable relationship exists:
- Customer has utang (all tiers), OR
- Customer enrolled in suki card (upper tier only), OR
- Both

Walk-in buyers who pay cash or GCash with no utang and no suki card are **anonymous**. No record. No PII collected. No RA 10173 obligation for walk-in buyers.

**Customer schema:**
```typescript
{
  id, storeId, name, phone,
  utangBalance, creditLimit,     // utang fields — all tiers
  consentGiven,                  // RA 10173 compliance
  isActive,
  sukiCardNumber,                // null at Free/Pro — populated at upper tier on enrollment
  pointsBalance,                 // null at Free/Pro — computed projection at upper tier
  schemaVersion
}
```

**Suki-only customer** (upper tier): `utangBalance: 0`, `creditLimit: null`, `sukiCardNumber: [populated]`.

**Utang features:**
- Credit limits: Pro tier (field schema-ready at Free)
- Aging reports: Pro tier (0–7 days current, 8–30 short overdue, 30+ long overdue)
- Clerk credit restrictions: Pro tier (policy-controlled per clerk)
- Soft-delete anonymization: `name → "Customer [ID]"`, `phone → null` (RA 10173)
- One-tap export per customer (data subject access request fulfillment)

### 32. Consignment Domain

**Design principle:** Every physical movement of consignment stock generates an event. No silent inventory changes. Full audit trail including actor identity on every event.

#### Consignment Lifecycle

```
CONSIGNMENT_RECEIVED
  Supplier drops batch. Opens consignment batch record.
  payload: { supplierId, productName, productId, receivedQty,
             supplierPrice, storePrice, receivedBy, note }
  Effect: consignment_batches record created, status='OPEN'
  NO effect on owned inventory — separate ledger
  Receipt: generated immediately (acceptance receipt)

CONSIGNMENT_SALE
  Unit sold from consignment batch at POS.
  payload: { batchId, qty, saleEventId }
  Effect: batch.soldQty + qty
  Tagged to sale line item: { consignmentBatchId }
  Owned inventory: not touched
  NO financial effect yet — settlement pending

CONSIGNMENT_RETURNED
  Supplier retrieves unsold units before or at settlement.
  payload: { batchId, returnedQty, returnedBy, note }
  Effect: batch.returnedQty + returnedQty
         If (soldQty + returnedQty == receivedQty): batch.status = 'RETURNED'
  NO financial effect — clears consignment ledger for returned units
  Receipt: generated immediately (return receipt)
  CRITICAL: this event is NOT optional — physically departed goods
            MUST be recorded or inventory appears on record but is absent

CONSIGNMENT_SETTLED
  Store pays supplier for sold units only.
  Settlement amount = supplierPrice × soldQty
  payload: { batchId, settledQty, totalPaid, sourcePool, settledBy, note }
  Effect: COGS recognized (totalPaid → Store P&L cost line)
         Source pool↓ by totalPaid
         batch.status = 'SETTLED'
  Owner PIN required
  Receipt: generated immediately (settlement receipt)
```

#### Consignment Receipts (All Three Types)

Every consignment event generates a printable/shareable receipt containing:
- Receipt type (Acceptance / Return / Settlement)
- Supplier name and contact
- Product name, batch reference
- Quantities (received / returned / sold as applicable)
- Prices and total amounts
- Actor: who performed the action (actorId → clerk or owner name)
- Timestamp
- Store name

Delivered via: ESC/POS thermal print (same printer transport as sale receipts) or digital share via OS share sheet.

#### Consignment Policy

```typescript
consignmentPolicy: {
  allowConsignment: boolean,
  defaultSettlementPeriod: 'SAME_DAY' | 'NEXT_DAY' | 'ON_DEMAND',
  defaultSourcePool: PoolIdentifier,
  allowPartialSettlement: boolean,
  requiresOwnerAuth: boolean       // for acceptance and settlement
}
```

#### Provisional P&L — Consignment Impact

- `CONSIGNMENT_SALE` does NOT affect P&L at time of sale
- `CONSIGNMENT_SETTLED` recognizes COGS → `Consignment Cost of Sales` line
- Consignment margin = `(storePrice − supplierPrice) × soldQty` → Store Gross Profit
- Unsettled open batches at period close → liability note in P&L footer

#### Suppliers Entity

```typescript
{
  id, storeId, name, phone, address, notes, isActive, schemaVersion
}
```

New Dexie table `suppliers`. New Firestore collection `/stores/{storeId}/suppliers/{supplierId}`.

### 33. Discounts & Promos

#### Two Distinct Types

**Ad-hoc Discount** — applied at POS by clerk, policy-controlled.
**Promo** — pre-configured by owner, auto-applied at POS when conditions met.

#### P&L Treatment
Discounts are **revenue reductions**, not expenses.
```
Gross Sales
Less: Discounts Given    ← distinct visible line
Net Revenue
```

#### Sale Line Item — Updated Fields

```typescript
// Per line item in SALE_COMPLETED payload:
{
  productId, qty, originalPrice, finalPrice,
  discountType: 'NONE' | 'AD_HOC' | 'PROMO' | 'SUKI_POINTS',
  discountValue: number,          // absolute amount discounted
  discountReason: string | null,  // promoId or owner note for ad-hoc
}
```

#### Ad-hoc Discount Policy

```typescript
discountPolicy: {
  allowAdHocDiscount: boolean,
  maxDiscountAmount: number | null,     // absolute ₱ limit
  maxDiscountPercent: number | null,    // percentage limit
  requiresOwnerAuth: boolean,           // if true: owner PIN required for any discount
  authorizedBy: 'owner_pin' | 'manager_or_owner' | 'any_clerk'
}
```

#### Promo Engine

```typescript
type PromoType = 'PRODUCT_DISCOUNT' | 'CART_THRESHOLD' | 'BUY_X_GET_Y' | 'POINTS_MULTIPLIER'

{
  id, storeId, name, type,
  isActive, startDate, endDate, activeDays,
  productIds: string[],           // which products qualify (empty = all)
  minimumCartTotal: number | null, // for CART_THRESHOLD type
  buyQty: number | null,          // for BUY_X_GET_Y
  getQty: number | null,
  discountType: 'PERCENT' | 'FIXED' | 'FREE_ITEM',
  discountValue: number,
  pointsMultiplier: number | null, // for POINTS_MULTIPLIER (upper tier)
  schemaVersion
}
```

**Promo evaluation:** At POS cart computation, the promo engine checks all active promos against current cart. Fully offline — promos pre-loaded in Dexie. Promo auto-deactivates when `endDate` passes (device time check).

### 34. Void & Return Workflows

**Void:**
- Same shift only
- Full transaction reversal (no partial voids)
- `SALE_VOIDED` compensating event with `originalEventId`
- Policy-controlled authorization

**Return:**
- Any shift, any time
- Partial or full quantities supported
- `SALE_RETURNED` compensating event with `originalEventId`
- Policy-controlled authorization

### 35. Policy Engine (Expanded)

```typescript
{
  voidPolicy: {
    requiresAuthorization: boolean,
    allowedWindow: 'same_shift',     // voids: same shift only
    authorizedBy: 'owner_pin' | 'manager_or_owner' | 'any_clerk'
  },
  returnPolicy: {
    requiresAuthorization: boolean,
    allowedWindow: 'same_shift' | 'same_day' | '7_days' | 'any',
    authorizedBy: 'owner_pin' | 'manager_or_owner' | 'any_clerk',
    allowPartial: boolean,
    refundMethod: 'cash_only' | 'original_method'
  },
  cashDropPolicy: {
    authorizedBy: 'owner_pin' | 'manager_or_owner' | 'any_clerk'
  },
  cashPickupPolicy: {
    authorizedBy: 'owner_pin' | 'manager_or_owner'  // never any_clerk
  },
  discountPolicy: { ... },           // see §33
  utangPolicy: {
    allowUtang: boolean,
    requiresOwnerAuth: boolean,
    maxUtangPerTransaction: number | null
  },
  expensePolicies: { [category]: ExpensePolicyEntry },  // see §29
  consignmentPolicy: { ... },        // see §32
  shiftGuardEnabled: boolean,        // Pro tier opt-in
  shiftGuardCashVarianceThreshold: number
}
```

**Rules:**
- Stored in Firestore `/stores/{storeId}/policies`, cached in Dexie
- Applied at shift open — mid-shift changes take effect next shift only
- `POLICY_CHANGED` event logs: old policy, new policy, changedBy, timestamp
- `CASH_PICKUP` always requires senior role — never delegatable to `any_clerk`

### 36. Cash Float Management

**CASH_DROP:**
```
amount, performedBy, reason, destinationNote (optional)
Authorization: policy-controlled
Effect: expected drawer balance↓
```

**CASH_PICKUP:**
```
amount, performedBy, reason, destinationNote (optional, recommended)
Authorization: always senior role minimum
Effect: expected drawer balance↓
```

**CASH_VARIANCE:**
```
Auto-generated at shift close when expected ≠ actual
expectedAmount, actualAmount, variance, shiftId, resolvedBy, resolution
Owner must acknowledge before shift closes
```

### 37. Suki Card Loyalty (Upper Tier Only)

**Free and Pro tiers:** `sukiCardNumber` field exists in schema, populated as `null`. No UI. No logic executed. Forward-compatible.

**Upper tier unlock — full feature:**

```
Points earn:
  On every qualifying SALE_COMPLETED:
  pointsEarned = floor(netSaleAmount / earnRate)
  earnRate = owner-configured (e.g., ₱10 = 1 point)
  SUKI_POINTS_EARNED event generated
  customer.pointsBalance projection updated

Points multiplier (via promo):
  PROMO type POINTS_MULTIPLIER: earnRate × multiplier during promo window

Points redemption:
  Customer presents suki card at POS (scan QR, enter phone, or name search)
  Clerk selects "Redeem Points"
  discountType: 'SUKI_POINTS'
  Redemption rate: owner-configured (e.g., 100 points = ₱10 off)
  SUKI_POINTS_REDEEMED event generated
  customer.pointsBalance↓

Suki Card:
  Card number = UUID (auto-generated on enrollment)
  QR code generated from sukiCardNumber
  Printed on receipt after enrollment
  Shareable digitally via OS share sheet
  Lookup: QR scan | phone number | name search — all offline-capable

P&L impact:
  SUKI_POINTS_REDEEMED = revenue reduction (same as any discount)
  Outstanding unredeemed points = liability note in Consolidated P&L footer:
  "Outstanding suki points liability: ₱X equivalent"
```

---

## PART VIII — FRAUD BASELINE & SHIFT GUARD

---

### 38. Fraud Baseline Detection (All Tiers)

Server-side, rule-based. Evaluated at `SHIFT_CLOSED` sync ingestion.

```
RULE-F1: Refund Ratio
  Trigger: refunds > 20% of shift gross sales
  Flag: SHIFT_HIGH_REFUND_RATIO (MEDIUM)

RULE-F2: Void Spike
  Trigger: void count > 5 OR void value > 30% of gross
  Flag: SHIFT_HIGH_VOID_RATE (MEDIUM)

RULE-F3: Cash Variance (existing)
  Trigger: |actual − expected| > ₱500
  Flag: CASH_VARIANCE (HIGH)

RULE-F4: Large Transaction
  Trigger: single sale > ₱2,000 (configurable)
  Flag: LARGE_TRANSACTION (LOW — informational)

RULE-F5: Extended Offline Shift
  Trigger: SHIFT_CLOSED sync > 48 hours after SHIFT_OPENED
  Flag: EXTENDED_OFFLINE_SHIFT (LOW — informational)
  Also auto-flagged for EMERGENCY sync uploads
```

**Tier visibility:**
- Free: flag summary badge (no drill-down)
- Pro: full flag detail per shift
- Upper: cross-shift trend, clerkId pattern analysis, weekly digest

### 39. Shift Guard (Pro Tier — Opt-In)

Default: OFF. Owner enables in Settings → Security → Shift Guard.

When ON, HIGH-severity flags trigger an owner PIN requirement before the **next** shift opens:

```
Triggering flags (configurable threshold):
  CASH_VARIANCE > ₱1,000 (owner sets: ₱500 / ₱1,000 / ₱2,000 / unlimited)
  SHIFT_HIGH_REFUND_RATIO
  SHIFT_HIGH_VOID_RATE

Enforcement flow:
  Shift close: always completes normally (never blocked)
  Server sets: shiftFlags.requiresOwnerAck = true
  Next SHIFT_OPENED attempt: check projections/store_summary.pendingEnforcementFlags
  If true: owner PIN modal — clerk cannot bypass
  Owner acknowledges → requiresOwnerAck cleared → shift opens

Guarantees:
  ✓ Sales never blocked mid-shift
  ✓ Shift close never blocked
  ✓ Only next shift open requires owner PIN
  ✓ Owner can always override
  ✓ Audit trail preserved regardless
```

---

## PART IX — BACKEND API & FIRESTORE

---

### 40. Backend API

**Type:** REST HTTP Cloud Functions (NOT Firebase Callable Functions)
**Structure:** Single Express-based Cloud Function (shared warm instances)
**Region:** `asia-southeast1` (Singapore) — ~20–50ms RTT to Philippines
**Warm instances:** Minimum 1 warm instance for sync endpoint (~₱600/month)
**Idempotency:** `eventId` UUID deduplication on every Cloud Function before processing

**API Domains:**
```
/api/v1/sync/events          → standard batch event upload (max 50/page)
/api/v1/sync/emergency       → corruption recovery upload (not in public API docs)
/api/v1/store/projections    → pull projections for rehydration
/api/v1/store/rebuild-projections  → Gatekeeper operation
/api/v1/devices/register     → device registration
/api/v1/devices/revoke       → device revocation
/api/v1/shifts/...           → shift operations
/api/v1/sales/...
/api/v1/inventory/...
/api/v1/gcash/...
/api/v1/utang/...
/api/v1/clerks/...
/api/v1/consignment/...
/api/v1/expenses/...
/api/v1/promos/...
/api/v1/suppliers/...
```

### 41. Firestore Read Model — Server-Side Projections

Written atomically by Cloud Functions on every sync ingestion. Read directly by monitor device and future dashboard.

```
/stores/{storeId}/projections/products/{productId}
  id, name, barcode, price, cost, stock, isInNegativeStock,
  lowStockThreshold, category, isActive, lastEventId, lastUpdated

/stores/{storeId}/projections/customers/{customerId}
  id, name, phone, utangBalance, creditLimit, isActive,
  consentGiven, sukiCardNumber, pointsBalance,
  lastEventId, lastUpdated

/stores/{storeId}/projections/gcash_state/current
  storeId, shiftId, cashStore, cashGCash, walletStore, walletGCash,
  lastEventId, lastUpdated

/stores/{storeId}/projections/shifts/{shiftId}
  all shift fields + reconciliationStatus, lastUpdated

/stores/{storeId}/projections/store_summary/current
  storeId, todaySales, todayGCash, openShiftId, activeClerkId,
  stockAlertCount, poolAlertCount, pendingEnforcementFlags,
  lastUpdated
```

**Monitor device** (Pro tier, read-only): reads `/projections/*` directly via Firestore Security Rules. `deviceRole: 'monitor'` enforced via Firebase Custom Claim.

### 42. Firebase Configuration

```
Bundle optimization:
  firebase/firestore/lite (~20KB) — Dexie handles offline reads
  initializeAuth() with Google provider only
  Lazy-load Firebase modules after POS screen renders
  Firestore: memoryLocalCache() only — NO Firestore offline persistence
             (prevents Dexie contention and double-caching)

Environments: dev / staging / prod (three separate Firebase projects)
Config: Vite .env files per environment. All gitignored except .env.example.
```

---

## PART X — DEVICE MANAGEMENT & SUBSCRIPTIONS

---

### 43. Device Registration

Universal device registration across all tiers. Every device registers a `deviceId` (UUID) in Firestore `/stores/{storeId}/registeredDevices`.

```typescript
{
  deviceId, storeId,
  type: 'android_pwa' | 'web_browser',
  role: 'primary' | 'monitor',
  status: 'ACTIVE' | 'REVOKED' | 'SUSPENDED',
  registeredAt, lastSeenAt,
  revokedAt: null | string,
  revokedBy: null | ownerUID,
  revokeReason: null | 'TRANSFER' | 'LOST' | 'COMPROMISED' | 'REPLACED'
}
```

**Per-tier device limits:**

| Tier | Devices |
|---|---|
| Free | 1 primary only |
| Pro | 1 primary + 1 monitor |
| Upper | Unlimited |

### 44. Device Revocation

```
Owner initiates revocation (requires connectivity):

1. Owner: Settings → Devices → Revoke
2. Cloud Function:
   a. registeredDevices.status = 'REVOKED'
   b. Writes to /stores/{storeId}/revokedDevices/{deviceId} (fast lookup set)
   c. Revokes Firebase Custom Claims for that deviceId
   d. Logs DEVICE_REVOKED event

3. Revoked device on next API call: 403 DEVICE_REVOKED → PWA locks

4. Offline revoked device:
   Continues locally until next API call
   On reconnection: sync returns 403 → device locked
   Pending events from revoked device → dead_letters (DEVICE_REVOKED reason)
   Owner notified to review
   This is an accepted offline-first tradeoff (see §acknowledged tradeoffs)
```

### 45. Tier State Management

- **Source:** Firebase Custom Claims (`{ tier, deviceRole, deviceId }` in JWT)
- **Upgrade:** Cloud Function updates claim → `getIdToken(true)` force-refresh → PWA reflects within seconds
- **Offline fallback:** Last known tier cached in Dexie
- **Downgrade policy:**
  - Payment lapse: 7-day grace → auto-downgrade to Free
  - Explicit: end of billing cycle
  - Data: NEVER deleted — lock and preserve forever
  - Clerks beyond Free limit: deactivated, not deleted
  - Monitor device: suspended, not deleted
  - Sync: ALWAYS continues regardless of tier

---

## PART XI — PWA, SERVICE WORKER & HARDWARE

---

### 46. PWA State Management

```
useLiveQuery (Dexie)  → all persisted data: products, shifts, customers, gcash, expenses, consignment
Zustand               → active cart, current clerk session, UI state, non-persisted
```

No React Query. Zero extra reactive dependencies.

### 47. Service Worker

- Strategy: `vite-plugin-pwa` with `injectManifest` (custom service worker)
- Update policy: `registerType: 'prompt'` — shown only when cart empty + no active shift + sync queue idle

| Resource | Strategy |
|---|---|
| App shell (HTML/JS/CSS) | Precached, Cache First |
| Static assets | Cache First, long TTL |
| Product images | Cache First + expiry |
| API calls | Network First, 5s timeout, Dexie fallback |

**Storage persistence:**
- `navigator.storage.persist()` called on install AND every launch
- If denied: persistent warning banner
- `navigator.storage.estimate()` on every resume: ≥70% → warning; ≥85% → critical alert

### 48. Performance Budget

| Metric | Limit |
|---|---|
| Initial JS bundle | < 150KB gzipped |
| Total app shell | < 300KB |
| Bundle increase | Requires Gatekeeper sign-off |

- Route-based code splitting via React `lazy()` + `Suspense`
- POS screen: NEVER lazy-loaded — always in initial bundle
- Images: WebP, lazy-loaded everywhere except POS
- No moment.js — date-fns only (tree-shakeable)
- Tailwind PurgeCSS at build

### 49. Hardware Integration

**Principle:** Brand-agnostic via standard protocols. ALL hardware code in `/packages/pwa` only.

| Hardware | Protocol | Notes |
|---|---|---|
| Receipt Printer | ESC/POS — dual transport | Any brand, any BT version |
| Cash Drawer | Via printer RJ11 signal | Automatic — no separate SDK |
| Barcode Scanner (physical) | HID keyboard emulation | Any brand — plug and play |
| Barcode Scanner (camera) | ZXing JS library | Uses device camera |

#### Printer Transport — Dual Path (Capacitor is NOT used)

The ESC/POS command layer is identical for both paths. Only transport differs.

**Path A — Direct BLE (Web Bluetooth API):**
BLE-capable printers. Pure PWA. No companion app. Auto-detected on pairing.

**Path B — Bluetooth Classic via Companion Bridge App:**
The majority of affordable PH market models (Xprinter, GOOJPRT, HOIN, Munbyn — ₱1,500–3,000) use Bluetooth Classic (SPP/RFCOMM) which Web Bluetooth does NOT support.

Solution: Thin native Android app (~200–300 lines). Opens BT Classic socket, exposes local WebSocket on `localhost`. PWA sends ESC/POS bytes to `ws://localhost:PORT` → bridge forwards to printer.

Installed once at Pro onboarding. Runs silently in background. Industry-standard pattern (Loyverse, Square, major SEA POS apps use identical approach).

```
Detection:
  Printer advertises BLE → Path A
  Printer requires Classic → prompt bridge install → Path B
```

All printer code via `IBluetoothTransport` interface (`BLETransport | BridgeTransport`). Identical receipt output on both paths.

**Tier Hardware Access:**

| Tier | Hardware |
|---|---|
| Free | Camera barcode scanning + HID physical scanner + digital receipts only |
| Pro+ | All Free + ESC/POS printer (BLE or Classic) + cash drawer |

#### Receipt Templates

Receipt templates cover:
- Standard sale receipt (existing)
- Consignment acceptance receipt (NEW)
- Consignment return receipt (NEW)
- Consignment settlement receipt (NEW)

All templates: ESC/POS thermal print or digital share via OS share sheet.

---

## PART XII — LONG-TERM STORAGE STRATEGY

---

### 50. Event Storage Lifecycle

Firestore event log: **immutable, never pruned, never TTL'd**. This policy is absolute.

**Local Dexie:** Events pruned after 7 shifts (shift-close snapshot enables delta replay).

**Firestore Tiered Archival (Year 2 activation):**

```
TIER 1 — HOT (0–6 months): Firestore direct query
TIER 2 — WARM (6–24 months):
  Cloud Scheduler monthly job exports events > 180 days old
  to gs://tindar-event-archive/{storeId}/{year}/{month}/ (NDJSON gzipped)
  Firestore record NOT deleted — archivedAt + archivePath pointer added
TIER 3 — COLD (24+ months):
  Cloud Storage Lifecycle → Nearline/Coldline

Activation trigger:
  Any store crosses 50,000 events, OR
  Total Firestore events collection crosses 10M documents
  (Monitor via Cloud Monitoring alert)
```

---

## PART XIII — i18n, NOTIFICATIONS & COMPLIANCE

---

### 51. Internationalization

- Framework: `react-i18next`
- Language packs: JSON files in `/packages/ui/locales/`
- Launch languages: Filipino (primary), English (secondary), Bisaya
- All UI strings key-referenced — never hardcoded
- Active language pack only loads (lazy)
- Domain terms locked: Paninda, Benta, Utang, Suki, Ulat
- New language = one JSON file, zero code change

### 52. Notifications & Monitoring

- **Free tier:** In-app banners and badges computed from Dexie on app resume. No push.
- **Pro tier:** FCM push notifications via Cloud Functions (low stock, large utang balance, shift flags). FCM tokens registered at shift start.
- **Error tracking:** Sentry in both `/packages/pwa` and `/packages/functions`
- **Backup:** Daily Firestore exports to Cloud Storage via Firebase scheduled backups

### 53. Data Privacy Act RA 10173

Compliance-assisted approach. TINDAR is the Personal Information Processor (PIP). Store owner is the Personal Information Controller (PIC).

**Implementation:**
- Customer records created only for utang/suki relationships — walk-in buyers are anonymous (minimal PII collection by design)
- `consentGiven: boolean` on every customer record
- Soft-delete anonymization: `name → "Customer [ID]"`, `phone → null` in Dexie and Firestore
- **Event scrimming:** On privacy request/consent revocation, PII fields in event log are replaced with anonymized placeholder (`"Customer [anonymizedId]"`, `phone: null`). Event financial structure and integrity preserved — only PII fields scrubbed. This is NOT event deletion.
- One-tap customer data export (data subject access request fulfillment)
- Privacy notice displayed at onboarding
- TINDAR registers with NPC as PIP (business action, not dev task)

### 54. GCash BSP Agent Registration (Circular 1166)

Zero architecture impact. TINDAR is a manual ledger — no GCash API, no e-money processing, no fund intermediation.

**Action:** Onboarding disclaimer + ToS clause: TINDAR is a recording tool only. Owners solely responsible for their own GCash agent registration and BSP compliance.

### 55. BIR Compliance Positioning

Launch as internal business tracking tool — not a registered POS. No BIR CAS in build scope. Target market is VAT-exempt (below ₱3M gross). `eventId` UUIDs serve as forward-compatibility hook for future BIR sequential transaction numbering.

---

## PART XIV — DEVELOPMENT WORKFLOW & GOVERNANCE

---

### 56. 5-Agent Swarm Protocol (v6.1)

| Agent | Role | IDE | Focus |
|---|---|---|---|
| Agent 1 | Master AI / Gatekeeper | Both | SOT integrity, collision management, phase-lock validation |
| Agent 2 | Cloud Backend | Windsurf | Firestore, Cloud Functions, projections |
| Agent 3 | Dashboard/State | Windsurf | Global state, dashboard UI |
| Agent 4 | PWA Core | Antigravity | Dexie, offline engine, hardware |
| Agent 5 | PWA UX | Antigravity | Speed optimization, UX polish |

**Phase-Locked Timeline:**

| Phase | Days | Deliverable |
|---|---|---|
| SOT & Schema | 1–6 | Architecture locked, schema defined, sync API spec, shared types |
| Master Planning | 7–17 | QA ready, dependency maps, test plans, mock data |
| Task Drafting | 18–20 | Assignments, branch prep |
| Implementation | 21–32 | Auto-tests, Windsurf dev, PWA dev (3-day micro-sprints) |
| Hardening | 33–42 | Final merge, stress test, gold master |

70% front-loaded planning → estimated 85% reduction in implementation-phase conflicts.

**Sync checkpoints:** Day 24, 27, 30

**Conflict protocol:**
```
IF schema_divergence > 0:
    HALT_IMPLEMENTATION
    REVERT_TO_SOT
    REGENERATE_INTERFACES
```

#### Pre-Implementation Phase (Phase 0) — Package Ownership Lifecycle

Both `/packages/shared` and `/packages/ui` follow a two-stage ownership lifecycle before implementation begins.

**Stage 1 — Construction (Pre-Implementation):**
During the SOT & Schema and Master Planning phases, both packages are built out by the swarm under coordinated specification from the Master Task List. At this stage they are active work products, not yet locked. Worker agents contribute to their construction as assigned tasks.

**Stage 2 — Lock (Implementation onwards):**
Once the Phase 0 Completion Gate is satisfied and declared by Agent 1, both `/packages/shared` and `/packages/ui` transfer to full Gatekeeper ownership. From this point forward, no worker agent may modify either package without a formal Gatekeeper-approved change request. Any required modification triggers the Halt Protocol: worker agent stops, documents the required change, and waits for Agent 1 to update the package and merge to `main` before implementation resumes.

This lifecycle applies to both packages equally. The Shared-First rule in the Governance SOT refers to this locked state — it is the condition of these packages during the Implementation phase, not during pre-implementation construction.

#### Phase 0 Completion Gate

Phase 0 is complete when all of the following are confirmed and merged to `main`. Completion is declared by Agent 1 only. No worker agent self-certifies.

| # | Deliverable | Location |
|---|---|---|
| 1 | All TypeScript entity types exported | `/packages/shared/src/types/` |
| 2 | All abstract repository interfaces defined | `/packages/shared/src/repositories/` |
| 3 | Event taxonomy frozen | `/packages/shared/src/events/` |
| 4 | GCash 4-pool math functions implemented and unit-tested | `/packages/shared/src/business/` |
| 5 | Shift reconciliation validator implemented and unit-tested | `/packages/shared/src/validators/` |
| 6 | Policy engine authorization checks implemented | `/packages/shared/src/policies/` |
| 7 | `/packages/ui` base design tokens defined | `/packages/ui/src/tokens/` |
| 8 | `/packages/ui` core component stubs merged (interface only — no implementation required) | `/packages/ui/src/components/` |
| 9 | Monorepo build confirmed at zero TypeScript errors across all packages | CI pipeline |
| 10 | Mock data seed scripts available for all agents | `/packages/shared/src/seed/` |

Implementation branches are not created until all 10 items are ✅.

---

### 57. Gatekeeper Rules (Complete — 16 Rules)

All of the following require explicit Gatekeeper sign-off before proceeding:

1. Any change to `/packages/shared` (types, validators, interfaces)
2. Any Dexie schema version increment
3. Any increase to the JS bundle budget (>150KB gzipped)
4. Any deviation from the locked decisions in this document
5. Any new dependency addition to `/packages/pwa` that affects bundle size
6. Any Cloud Functions schema or API contract change
7. Any change to GCash 4-pool calculation logic (protected IP)
8. Any modification to the corruption recovery sequence
9. Any change to fraud baseline rules or thresholds
10. Any modification to device revocation flow
11. Any application of TTL or pruning policy to Firestore events collection
12. Any change to projection rebase threshold values
13. The emergency sync endpoint must never appear in public-facing API documentation
14. Shift Guard default must remain OFF at Pro tier launch
15. Event archival Cloud Scheduler job must be tested against full projection rebuild before activation
16. Any change to P&L calculation logic or expense attribution model

### 58. Key Risks & Mitigations

| Risk | Mitigation |
|---|---|
| Hardening phase is thin (10 days) | Prioritize offline sync edge cases, Pro reconnection, consignment reconciliation |
| Agent scope creep | Schema is already forward-compatible — agents never add fields "just in case" |
| Schema divergence between teams | Phase-lock: halt, revert to SOT, regenerate interfaces |
| IndexedDB corruption | 7-step corruption recovery protocol with upload-before-wipe |
| Firestore cold starts | Single Express function + 1 warm instance |
| Extended offline (1 week+) | Paged sync (50 events/page) prevents payload overflow |
| Thundering herd on reconnect | Jittered backoff (random 0–30s delay before first sync) |
| Revoked device offline window | Accepted tradeoff — dead-letter handling, owner review |

---

## PART XV — COMPLETE SCHEMA REFERENCE

---

### 59. PoolIdentifier Enum

```typescript
type PoolIdentifier = 'CASH_STORE' | 'CASH_GCASH' | 'WALLET_STORE' | 'WALLET_GCASH'
```

### 60. Acknowledged Tradeoffs

**RR-04 — Revoked device offline window:**
Device revoked at 9am, remains offline all day → continues transacting locally. On reconnection, events rejected to dead_letters. Owner reviews. Inherent to offline-first design — accepted.

**RR-05 — sortKey millisecond collision:**
Firestore `serverTimestamp()` not guaranteed monotonic within single millisecond at batch boundary. Three-level tie-break (serverTime → deviceId → localSequenceNumber) makes this deterministic regardless. Not material at sari-sari transaction volume. Accepted.

---

## PART XVI — ALL DECISIONS LOCK

### ADR Status — Complete Register

| # | Item | Status |
|---|---|---|
| 1 | Bluetooth printer — BLE + Classic bridge dual path | LOCKED |
| 2 | Event log compaction — shift snapshots + 7-shift local prune | LOCKED |
| 3 | Tingi — unit-level tracking, unitsPerBox convenience multiplier | LOCKED |
| 4 | i18n — react-i18next, 3 launch languages | LOCKED |
| 5 | BIR — internal tracking tool, not registered POS | LOCKED |
| 6 | Device registration — universal, primary/monitor roles | LOCKED |
| 7 | Tier downgrade — 7-day grace, lock-and-preserve, never delete | LOCKED |
| 8 | Split payment + GCash 4-pool + dual domains | LOCKED |
| 9 | Void/return + compensating events + policy engine + RBAC | LOCKED |
| 10 | Cash float — CASH_DROP, CASH_PICKUP, CASH_VARIANCE, reconciliation | LOCKED |
| 11 | RA 10173 — compliance-assisted, consent, anonymization, scrimming | LOCKED |
| 12 | GCash BSP — zero dev impact, onboarding disclaimer | LOCKED |
| 13 | Capital withdrawal + structured expenses + provisional P&L | LOCKED |
| 14 | Consignment domain — full lifecycle + receipts + suppliers | LOCKED |
| 15 | Discounts + promos — policy engine, revenue reduction in P&L | LOCKED |
| 16 | Suki Card — upper tier only, schema-ready at Free/Pro (null) | LOCKED |
| GAP-01 | Conflict resolution — server-time ordered, delta-based financials | LOCKED |
| GAP-02 | HLC — deterministic sortKey, 3-level tie-break | LOCKED |
| GAP-03 | Forced sync gate at shift close, HIGH priority queue | LOCKED |
| GAP-04 | Corruption recovery — upload→snapshot→wipe→rehydrate | LOCKED |
| GAP-05 | Firestore /projections/* read model for monitor device | LOCKED |
| GAP-06 | Firestore event log — immutable, never pruned | LOCKED |
| GAP-07 | HMAC — per-device key: HMAC(pepper, ownerUID+":"+deviceId) | LOCKED |
| GAP-08 | Device revocation — full lifecycle, revokedDevices collection | LOCKED |
| GAP-09 | Inventory overdraw — allowed, flagged, never blocks sale | LOCKED |
| GAP-10 | Fraud baseline — 5 rules, all tiers, informational | LOCKED |
| RR-01 | Projection rebase — server-wins, threshold-triggered, audited | LOCKED |
| RR-02 | Emergency endpoint — 5/hr rate limit, idempotency, audit log | LOCKED |
| RR-03 | Shift Guard — Free=info, Pro=opt-in, Upper=analytics | LOCKED |
| RR-04 | Revoked device offline window — accepted tradeoff | ACKNOWLEDGED |
| RR-05 | sortKey millisecond collision — deterministic tie-break sufficient | ACKNOWLEDGED |
| RR-06 | Event archival — tiered strategy, Year 2 activation | DEFERRED |
| C2-01 | Paged sync — 50 events/page, sequential, priority-ordered | LOCKED |
| C2-02 | RA 10173 event scrimming — field-level PII scrub, structure preserved | LOCKED |
| C2-03 | Hybrid PIN provisioning — tempHash offline, replaced on sync | LOCKED |
| C2-04 | Jittered backoff — random 0–30s delay after outage reconnect | LOCKED |

---

## APPENDIX — LONG-TERM STRATEGIC VISION

TINDAR is designed to evolve into a Micro Retail Intelligence Platform:

- Inventory optimization and float prediction
- Risk detection and branch dashboards
- Integration with micro-financing
- Supplier marketplace
- Embedded lending
- BIR CAS registration (upper tier, post-launch)
- BigQuery analytics for cross-store intelligence
- SEA expansion (Indonesia, Vietnam, Myanmar micro-retail)

**Foundation features that position TINDAR for this evolution:**
- GCash 4-pool model (protected IP) — deepest moat against all competitors
- Consignment domain — captures supplier relationship data no competitor has
- Provisional P&L — creates financial discipline habits in micro-retailers
- Event-sourcing — every future analytics feature is a projection, never a migration
- Suki Card — customer loyalty data for targeted supplier and lending products

---

*TINDAR Master Architecture Document v3.0 | March 2026*
*This document is the single source of truth. All prior documents are superseded.*
*All decisions are LOCKED pending Gatekeeper acknowledgment.*
*No implementation begins until Gatekeeper formally acknowledges this document.*
