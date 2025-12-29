---
alwaysApply: true
---
# TypeScript Coding Standards

Follow these standards strictly. Deviations require explicit agreement with the user.

## Function Signatures: Object Parameters Only

**Always use object parameters for functions. Never use positional arguments.**

Positional arguments become unreadable as functions grow and make refactoring dangerous. Object parameters are self-documenting and order-independent.

```typescript
// ❌ BAD: Positional arguments
function createTransaction(
  amount: number,
  currency: string,
  fromAddress: string,
  toAddress: string,
  memo?: string,
  priority?: number
) { }

// What do these arguments mean at the call site?
createTransaction(100, 'USD', '0x123', '0x456', undefined, 2);

// ✅ GOOD: Object parameter
function createTransaction(params: {
  amount: number;
  currency: string;
  fromAddress: string;
  toAddress: string;
  memo?: string;
  priority?: number;
}) { }

// Self-documenting at the call site
createTransaction({
  amount: 100,
  currency: 'USD',
  fromAddress: '0x123',
  toAddress: '0x456',
  priority: 2,
});
```

**Exception:** Single-argument functions where the meaning is unambiguous (e.g., `parseId(id)`, `toString(value)`).

## No Casting to `any` or `unknown`

**Never cast to `any` or `unknown`. This bypasses TypeScript's type system and hides bugs.**

If you're reaching for a cast, it means either:
- The types are wrong and need fixing at the source
- You need a type guard or assertion function
- The code needs restructuring

```typescript
// ❌ BAD: Casting to escape type errors
const result = someValue as any;
const data = response as unknown as MyType;

// ✅ GOOD: Fix the actual type
const result: ExpectedType = someValue;

// ✅ GOOD: Use a type guard
function isMyType(value: unknown): value is MyType {
  return typeof value === 'object' && value !== null && 'requiredProp' in value;
}

if (isMyType(response)) {
  // TypeScript now knows the type
}

// ✅ GOOD: Use assertion with runtime validation
import { z } from 'zod';

const MyTypeSchema = z.object({ requiredProp: z.string() });
const validated = MyTypeSchema.parse(response);
```

If you genuinely believe a cast is necessary, **stop and discuss with the user first**. Explain what's forcing the cast and agree on the approach before proceeding.

## No Inline Imports

**All imports belong at the top of the file. Never import modules inside function scopes or use inline `import()` syntax.**

Inline imports obscure dependencies, make code harder to trace, and are often a workaround for circular dependency issues that demand proper architectural solutions.

```typescript
// ❌ BAD: Inline type imports
function processTransaction(tx: import('./types.js').Transaction) { }

type Config = import('../../config.js').AppConfig;

// ❌ BAD: Imports inside function scope
async function analyseTransaction(txId: string) {
  const { TransactionParser } = await import('./parser.js');
  const { validateSchema } = await import('../utils/validation.js');
  // ...
}

// ✅ GOOD: All imports at top of file
import type { Transaction } from './types.js';
import type { AppConfig } from '../../config.js';
import { TransactionParser } from './parser.js';
import { validateSchema } from '../utils/validation.js';

async function analyseTransaction(txId: string) {
  // ...
}
```

**If you're tempted to use inline imports to avoid circular dependencies, stop.** This is a code smell indicating the module structure needs refactoring. Discuss with the user rather than papering over the problem.

**Narrow exception:** Dynamic imports for intentional code-splitting in frontend bundles, where deferring module loading is a deliberate optimisation to reduce initial payload. This must be discussed and agreed with the user first — it's an architectural decision, not a convenience.

## Reuse Existing Types

**Before declaring a new type, search the codebase for existing types that serve the same purpose. Import and reuse them.**

Redeclaring types locally defeats the purpose of TypeScript's type system. Types should be shared across the codebase to ensure consistency and catch integration errors at compile time.

```typescript
// ❌ BAD: Redeclaring a type that exists elsewhere
// (Transaction is already defined in src/types/transaction.ts)
interface Transaction {
  id: string;
  amount: number;
  currency: string;
}

function processTransaction(tx: Transaction) { }

// ❌ BAD: Inline type that duplicates existing structure
function getUser(params: { id: string; email: string; role: 'admin' | 'user' }) { }

// ✅ GOOD: Import existing types
import type { Transaction } from '@/types/transaction.js';
import type { User } from '@/types/user.js';

function processTransaction(tx: Transaction) { }

function getUser(params: Pick<User, 'id' | 'email' | 'role'>) { }
```

**Before writing any new types:**

1. Search the codebase for existing types with similar names or structures
2. Check shared type directories (`types/`, `models/`, `interfaces/`)
3. If a similar type exists, import it — use `Pick`, `Omit`, or `Partial` if you need a subset
4. Only declare new types when no suitable type exists, and consider whether it should be shared

If two modules need the same type, that type belongs in a shared location — not duplicated in each module.

## Don't Redeclare Types from Interfaces

**When implementing an interface, don't redeclare return types or parameter types that are already defined by the interface.**

TypeScript infers these from the interface. Redeclaring them adds noise, creates drift risk, and defeats the point of the interface contract.

```typescript
interface TransactionProcessor {
  process(tx: Transaction): Promise<ProcessResult>;
  validate(tx: Transaction): ValidationOutcome;
}

// ❌ BAD: Redeclaring types already defined by the interface
class PaymentProcessor implements TransactionProcessor {
  async process(tx: Transaction): Promise<ProcessResult> {
    // ...
  }
  
  validate(tx: Transaction): ValidationOutcome {
    // ...
  }
}

// ✅ GOOD: Let TypeScript infer from the interface
class PaymentProcessor implements TransactionProcessor {
  async process(tx) {
    // ...
  }
  
  validate(tx) {
    // ...
  }
}
```

The interface is the contract. The implementation inherits its types — don't repeat yourself.

## Summary

| Rule | Standard | Exception |
|------|----------|-----------|
| Function parameters | Always use object params | Single unambiguous argument |
| Type casting | Never cast to `any`/`unknown` | Only with explicit user agreement |
| Imports | Always at top of file | Code-splitting with user agreement |
| Type declarations | Reuse existing types | New types only when none exist |