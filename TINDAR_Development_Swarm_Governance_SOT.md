# TINDAR — Development & Swarm Governance SOT
**Status:** LOCKED — Single Source of Truth. No changes without Gatekeeper sign-off.
**Version:** 1.0 — Final
**Authority:** Agent 1 (Master AI / Gatekeeper)
**Methodology:** Contract-Based Implementation (CBI) — "Planning-Heavy, Implementation-Stateless"
**Root Authority:** TINDAR Master Architecture Document v3.0 (`TINDAR_Master_Architecture_SOT.md`)

> This document governs how the 5-agent swarm executes the TINDAR build.
> It does not supersede the Master Architecture SOT — it operationalizes it.
> In any conflict, the Hierarchy of Truth in Section 4, Rule 6 is the arbiter.

---

## 1. Swarm Composition & Environment Mapping

The swarm is divided into two distinct IDE environments to ensure logic isolation and prevent environment-specific leaks (e.g., using Node.js Admin SDK logic in the PWA, or Dexie.js browser logic in Cloud Functions).

| Agent | Role | IDE Environment | Responsibility Scope |
|---|---|---|---|
| Agent 1 | Master AI / Gatekeeper | Omnipresent | SOT integrity, schema authority, `/packages/shared` and `/packages/ui` ownership post-lock, sole PR reviewer, Phase 0 gate declaration. |
| Agent 2 | Cloud Backend | Windsurf | Firestore Admin SDK, Cloud Functions, server-side projections, REST API endpoints. |
| Agent 3 | Dashboard/State | Windsurf | Global Zustand state management, Pro tier dashboard UI components as specified in the Master Task List, and server-side projection read layer for dashboard views. The `/web` package (upper tier) is scaffolded as an empty package with no UI implementation in this sprint. |
| Agent 4 | PWA Core | Antigravity | Dexie.js repository implementations, offline sync engine, event sourcing, sync queue management, HLC ordering. |
| Agent 5 | PWA UX | Antigravity | Hardware integration (ESC/POS printer, barcode scanner, cash drawer), performance budgeting, PWA UX polish. Performance budgeting means: maintaining initial JS bundle under 150KB gzipped, virtualizing all list components (product search, customer lookup, transaction history), enforcing 48dp minimum touch targets, ensuring 60fps UI on low-end Android (no unnecessary re-renders), and validating high-contrast outdoor display readiness. |

**Agent 1 commit protocol:** Agent 1 commits directly to `main` for all `/packages/shared` and `/packages/ui` updates. Agent 1 does not create feature branches. This is the Gatekeeper's exclusive direct-commit privilege — no other agent may commit directly to `main` under any circumstance.

---

## 2. The "Shared-First" Logic Gate

`/packages/shared` and `/packages/ui` are the dual sources of truth for all implementation contracts. No worker agent begins implementation in Windsurf or Antigravity until both packages are defined, built out, and locked by Agent 1.

### Package Contents

**`/packages/shared`** — Business contracts:
- TypeScript entity types (all domains)
- Event taxonomy (all event types — frozen at Phase 0 lock)
- Abstract repository interfaces
- Core business logic (GCash 4-pool math, shift reconciliation, policy engine, validators)
- Seed/mock data scripts

**`/packages/ui`** — Design contracts:
- Base design tokens (colors, spacing, typography)
- Core React component stubs
- Tailwind configuration
- i18n locale JSON files

### The Two-Stage Ownership Lifecycle

Both packages follow a two-stage ownership lifecycle. This is not a permanent day-one restriction — it is a phased handoff.

**Stage 1 — Pre-Implementation Construction:**
During the SOT & Schema and Master Planning phases (Phase 0), both `/packages/shared` and `/packages/ui` are active work products. The swarm builds and defines them under coordinated specification from the Master Task List. Worker agents may contribute to their construction as assigned tasks during this stage.

**Stage 2 — Gatekeeper Lock (Implementation onwards):**
Once Agent 1 declares the Phase 0 Completion Gate satisfied (see below), both packages transfer to full Gatekeeper ownership. From this point, no worker agent may modify either package without a formal change request to Agent 1. Any required modification triggers the Halt Protocol: the worker agent stops, documents the required change, and waits for Agent 1 to update the package and merge to `main` before implementation resumes.

### The "Shared Gate" Rule (Locked Phase)
- Worker agents (2–5) are strictly forbidden from modifying `/packages/shared` or `/packages/ui` after lock.
- If a schema or component change is needed, the agent halts and submits a change request to Agent 1.
- No workarounds, no local copies, no re-implementations of shared logic.

### Phase 0 Completion Gate

Phase 0 is complete when all of the following are confirmed and merged to `main`. **Completion is declared by Agent 1 only.** No worker agent self-certifies. Implementation feature branches are not created until all 10 items are ✅.

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

---

## 3. Git Workflow & Branching Protocol

### 3.1 Branching Model

- **`main`:** Stable and deployable. No direct commits except Agent 1 for shared package updates.
- **Feature Branches:** Short-lived branches created from `main` by worker agents only.
- **Naming Convention:** `agent-[ID]/[team-prefix]/[phase]-[feature]`
  - Examples: `agent-4/antigravity/phase-1-dexie-setup`, `agent-2/windsurf/phase-2-firestore-projections`
  - Team prefixes: `windsurf` (Agents 2–3), `antigravity` (Agents 4–5)

### 3.2 Commit Standards

- **Format:** `[Phase X] Agent [ID]: [Action Verb] [Feature]`
  - Example: `[Phase 2] Agent 2: Implement Firestore projection for products`
- **Scope:** Each commit addresses exactly one task from the Master Task List. No bundled commits.
- **Tests:** Unit tests must be included in the same commit as the implementation they test.

### 3.3 Pull Request & Review Gate

- **Sole Reviewer:** Agent 1 (Gatekeeper) reviews all PRs. No peer reviews between agents.
- **Review Criteria:** Code is evaluated against all 16 Gatekeeper Rules (Section 4).
- **Statelessness:** If an agent's code diverges from the SOT, the branch is discarded — not negotiated. Implementation is "transient labor" bound by the contracts. Discarding and regenerating from the SOT is faster and safer than merging divergent logic.
- **PR Description:** Must include: task reference from Master Task List, what was implemented, what tests were added, and any edge cases handled.
- **Review SLA:** Agent 1 reviews and resolves all PRs within the same micro-sprint day they are submitted. A PR sitting unreviewed overnight is a blocker. If Agent 1 cannot review same-day, the submitting agent pauses and does not begin the next task.

---

## 4. Operational Guardrails — The 16 Gatekeeper Rules

Agent 1 will reject any PR that violates any of these rules. Violations result in branch discard, not negotiation.

**1. Contract Integrity**
All implementations must use the exact TypeScript interfaces defined in `/packages/shared`. Agents cannot add, remove, or rename fields from these entities unilaterally.

**2. No "Shadow Logic"**
Any business math — especially GCash 4-pool calculations, shift reconciliation, policy engine checks, and P&L computation — must reside only in `/packages/shared`. It is forbidden to replicate this logic locally in the PWA or Cloud Functions. Cloud Functions invoke shared validators; they do not rewrite them.

**3. Bundle Budget Compliance**
PWA additions must not push the initial gzipped JS bundle over 150KB. PRs that exceed this without a Gatekeeper-approved exception are automatically rejected.

**4. Domain Language Enforcement**
Code must use the Philippine-specific taxonomy defined in the Master Architecture SOT. Terms like Paninda, Utang, Suki, Tingi, and 4-Pool are canonical. Generic terms like "Inventory", "Credit", or "Customer" are prohibited in variable names and function names.

**5. Strict Typing**
The use of `any` types or `@ts-ignore` is a violation. Every data structure must be explicitly typed according to the shared TypeScript entities. TypeScript compiler must pass with `strict: true`.

**6. Hierarchy of Truth Adherence**
If a conflict arises between any documents, code, or instructions, the following hierarchy is absolute and non-negotiable:

```
1. TINDAR Master Architecture Document v3.0  (TINDAR_Master_Architecture_SOT.md)
      ↓ supersedes all prior ADRs and supplements
2. This Governance SOT  (TINDAR_Development_Swarm_Governance_SOT.md)
      ↓ operationalizes the Master Architecture SOT
3. /packages/shared and /packages/ui compiled types
      ↓ locked code contracts derived from Master Architecture SOT
4. Master Task List
      ↓ task assignments derived from the above
5. Local implementation code
      ← NEVER a source of truth. Always the output of the above.
```

An agent's local implementation code is never correct by definition if it conflicts with any layer above it. The branch is discarded.

**7. Protected Core IP — GCash 4-Pool**
Rule 7 specifically protects the 4-pool ledger model. No agent may alter the logical flow between CASH_STORE, CASH_GCASH, WALLET_STORE, and WALLET_GCASH pools. Any change to GCash 4-pool calculation logic requires Gatekeeper sign-off.

**8. Event Immutability**
Once written to the event log (Dexie or Firestore), data cannot be modified or deleted. Only compensating events (SALE_VOIDED, SALE_RETURNED, CONSIGNMENT_RETURNED) are allowed to change state. No UPDATE or DELETE operations on the events table or collection — ever.

**9. Offline-First Determinism**
All standard POS operations must be fully functional without a network connection. If an agent introduces a "waiting for server" state for any standard operation (sale, inventory deduction, GCash entry, utang, expense), the PR is rejected. The only operations that legitimately require connectivity are explicitly defined sensitive operations (void authorization, capital withdrawal, consignment settlement) as specified in the Master Architecture SOT.

**10. PII Scrimming (RA 10173)**
Customer PII fields (name, phone) must be handled in compliance with RA 10173. Event scrimming — field-level replacement of PII in synced events — must be implemented per the Master Architecture SOT §53 specification. Raw PII must never appear in Firestore event payloads after the scrimming step.

**11. Zero-Dependency Shared Package**
`/packages/shared` must remain pure TypeScript with zero external runtime dependencies. It cannot import from Dexie, Firebase, any browser API, or any Node.js-specific SDK. Violations break the package's ability to run in both browser and server contexts simultaneously.

**12. Idempotent Syncing**
Every Cloud Function event ingestion must be idempotent. The server must be able to receive the same `eventId` multiple times without duplicating any financial effect or projection update. `eventId` UUID deduplication is required on every ingestion endpoint before any processing begins.

**13. PIN Security (Bcrypt Only)**
Raw PINs must never be stored anywhere — not in Dexie, not in Firestore, not in logs. Only server-side bcrypt hashes are authoritative for permanent storage. The hybrid PIN provisioning flow (tempHash for offline clerk add) is the only exception and follows the exact protocol specified in the Master Architecture SOT §12.

**14. Deterministic Conflict Resolution**
Event ordering tie-breaks must follow the exact sequence defined in the Master Architecture SOT: `serverTime` → `deviceId` → `localSequenceNumber`. Agents cannot assume or invent ordering logic. The `sortKey` is computed server-side only — never by client code.

**15. Performance Budgets (60fps UI)**
UI components must not cause unnecessary re-renders. List components (product search, customer lookup, utang ledger, transaction history) must be virtualized or otherwise optimized for low-end Android hardware. Minimum 48dp touch targets on all interactive elements. High-contrast support required for outdoor use.

**16. Stateless Implementation**
Worker agents are prohibited from creating "hidden" state, utility functions, helper modules, or abstractions outside the agreed-upon task breakdown. If a piece of code is not traceable to a task in the Master Task List and a contract in `/packages/shared`, it does not belong in the codebase. Agents do not architect — they implement contracts.

---

## 5. Conflict & Recovery Handshakes

### Collision Management
IDE separation (Windsurf vs. Antigravity) acts as a structural "clean room" to prevent environment-specific dependency leakage. Node.js Admin SDK logic stays in `/packages/functions`. Dexie.js browser logic stays in `/packages/pwa`. Neither bleeds into the other. `/packages/shared` has zero environment-specific dependencies — it is the safe bridge between both environments.

### Sync Checkpoints
Mandatory cross-team syncs on **Days 24, 27, and 30** to check for schema divergence, interface drift, and task blockers. At each checkpoint:
- Agent 1 compares both teams' merged output against `/packages/shared` contracts
- Any divergence triggers the Halt Protocol immediately
- Checkpoint sign-off by Agent 1 is required before the next micro-sprint begins

### Halt Protocol
```
IF schema_divergence > 0:
    HALT_IMPLEMENTATION       ← all agents stop immediately
    REVERT_TO_SOT             ← divergent branch is discarded
    REGENERATE_INTERFACES     ← Agent 1 updates shared package if needed
    RESUME_FROM_CONTRACTS     ← worker agents re-branch from updated main
```

Divergence is never negotiated. It is always resolved by returning to the SOT and regenerating from contracts. This is the CBI philosophy in practice — branches are transient, contracts are permanent.

### Merge Barrier
Agent 1 must sign off on per-agent test logs before any branch merges to `main`. No self-merge. No peer-merge. Gatekeeper-only.

### Branch Isolation
Each agent works in a strictly scoped feature branch. No agent reads from or depends on another agent's unmerged branch. If Agent 4 needs an interface that Agent 2 hasn't merged yet, the work waits — it does not proceed against an unmerged dependency.

---

## 6. Phase-Locked Timeline Reference

| Phase | Days | Key Governance Events |
|---|---|---|
| SOT & Schema | 1–6 | Phase 0 construction begins. Both shared packages built out by swarm under Agent 1 coordination. |
| Master Planning | 7–17 | Phase 0 construction continues. Master Task List finalized. QA plans, dependency maps, mock data. |
| Task Drafting | 18–20 | Agent 1 declares Phase 0 Completion Gate. Feature branches created. Assignments confirmed. |
| Implementation | 21–32 | Packages locked. Gatekeeper owns `/packages/shared` and `/packages/ui`. 3-day micro-sprints. Sync checkpoints Day 24, 27, 30. |
| Hardening | 33–42 | Final merge, stress test, offline edge cases, reconnection scenarios, gold master. |

**70% front-loaded planning → estimated 85% reduction in implementation-phase conflicts.**

---

## 7. What Agents Know vs. What They Don't Need To Know

**Agents know:**
- Their assigned tasks from the Master Task List
- The TypeScript contracts from `/packages/shared` and `/packages/ui`
- The 16 Gatekeeper Rules
- This governance document
- The sections of the Master Architecture SOT relevant to their domain

**Agents do not need to know:**
- The full product roadmap or upper tier feature set
- Why specific architectural decisions were made (unless directly relevant to their task)
- What other agents are building (interfaces are the only coupling — not implementation details)
- Business strategy, pricing, or competitive positioning

This is intentional. Agents are contract-bound implementors. Their statelessness is a feature, not a limitation. The less they know beyond their contracts, the less they can deviate from them.

---

*TINDAR Development & Swarm Governance SOT v1.0 | March 2026*
*Authority: Agent 1 (Master AI / Gatekeeper)*
*Root document: TINDAR Master Architecture Document v3.0*
*Status: LOCKED — This document is the Single Source of Truth for swarm governance.*
*No modification without explicit Gatekeeper sign-off. All agents are bound by its rules from the moment implementation begins.*
