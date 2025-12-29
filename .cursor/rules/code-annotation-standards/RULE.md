---
alwaysApply: true
---

# Code Documentation Standards

Annotate code thoroughly to preserve architectural decisions and reasoning. Use JSDoc as the standard documentation format for all functions, classes, and modules.

## Critical: Document Real Decisions Only

**Documentation must reflect actual decisions made during our conversation or your planning process — never invent rationale after the fact.**

- If the user explained why something should work a certain way, capture that reasoning
- If you made a design choice during planning or implementation, document what you considered and why you chose this path
- If a constraint was discussed (performance, API limitations, business rules), record it
- If you're unsure why something is designed a certain way, ask rather than fabricate an explanation

**Bad:** Inventing plausible-sounding reasons for code you've written  
**Good:** Recording the actual conversation context, requirements, or trade-offs that led to this implementation

## Core Principles

1. **Why over what** — Explain _why_ this approach was chosen, not just what the code does. The code itself shows the "what".

2. **Alternatives considered** — Note approaches you actually evaluated and why they were unsuitable.

3. **Constraints and assumptions** — Document external factors discussed or discovered: API limitations, performance requirements, business rules, or dependencies.

4. **Non-obvious behaviour** — Flag anything that might look wrong but is intentional, with the reason it's needed.

5. **Future considerations** — Note known limitations or technical debt identified during implementation.

## JSDoc Standards

Use JSDoc for all exported functions, classes, and significant internal functions:

```typescript
/**
 * Calculates the risk score for a transaction based on counterparty history.
 *
 * DECISION: User specified weighted moving average rather than simple average
 * because recent transactions are more indicative of current behaviour patterns.
 *
 * ALTERNATIVES: Discussed exponential decay scoring but agreed the added
 * complexity wasn't justified given the 90-day analysis window requirement.
 *
 * CONSTRAINTS: Must complete within 50ms to meet API SLA (per requirements).
 * This is why we pre-compute counterparty stats rather than querying on each call.
 *
 * @param transaction - The transaction to analyse
 * @param counterpartyHistory - Previous transactions with this counterparty
 * @returns Risk score between 0-100, where 100 is highest risk
 * @throws {InsufficientDataError} When counterparty history is empty
 */
```

## Inline Comments

Use inline comments sparingly for non-obvious implementation details:

```typescript
// Intentionally not awaiting — discussed with user, fire-and-forget acceptable here
void logAnalytics(event);

// HACK: API returns null for empty arrays — workaround until v2 migration
const items = response.items ?? [];

// NOTE: Order matters — validation must run before transformation per data flow discussion
```

## Module-Level Documentation

For significant modules, include a header block explaining the module's purpose and architectural context:

```typescript
/**
 * @module TransactionAnalyser
 *
 * Handles real-time transaction risk scoring for the TXM pipeline.
 *
 * ARCHITECTURE: Discussed stateless design to support horizontal scaling.
 * All state lives in Redis rather than in-memory.
 *
 * DEPENDENCIES:
 * - Upstream: SQS queue populated by blockchain listeners
 * - Downstream: Alert service via SNS, results cached in Redis
 *
 * ASSUMPTIONS (confirmed with user):
 * - Transactions arrive at most once (deduplication happens upstream)
 * - Redis is available; failure triggers circuit breaker to DLQ
 */
```

## Decision Tags

Use consistent tags to make decisions searchable:

- `DECISION:` — Architectural or design choice with rationale from discussion or planning
- `ALTERNATIVES:` — Other approaches actually considered and why rejected
- `CONSTRAINTS:` — External factors discussed or discovered during implementation
- `HACK:` — Temporary workaround with context for future cleanup
- `NOTE:` — Important context from conversation that isn't obvious from the code
- `TODO:` — Future work identified during discussion, with enough context to action later

## When Not to Document

Don't over-annotate trivial code. Skip documentation for:

- Simple getters/setters with obvious purpose
- Utility functions with self-explanatory names
- Standard boilerplate where no decisions were made

If there was no decision to make, there's nothing to document.
