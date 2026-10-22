# Comment decision evals

A graded set for the gate, the deletion gate, the location table, the scope
boundary, and the review output in `SKILL.md`. Each case is a comment in context; the expected
answer is one of **keep**, **delete**, **rewrite**, **leave**, or **add**.

Use it two ways:

- **Regression check after editing the skill.** Prompt a model with Part 1 and
  the skill, collect its verdicts, compare with Part 2. The answer key is
  authoritative; the pass rule and the critical cases are under "Scoring".
- **Calibration when the skill feels too strict or too loose.** The same cases
  show where the balance moved.

Expected verdicts are per case, not per possible style improvement. "Rewrite"
means the comment must survive in some form; "leave" means do not touch it: the
file is outside the skill's scope (generated/vendored, license), the comment is
machine-read (build tag, pragma, region), or the file is outside the diff under
review; "add" means the required comment is missing. `keep` is an acceptable
synonym for `leave` in every scope case: the failure is adding, deleting,
reformatting, or rewriting, not the label.

---

## Part 1: cases

Prompt only this half.

### E1

```typescript
// increment the retry counter
retries += 1;
```

### E2

```typescript
// if the user is active, send the email
if (user.isActive) {
  sendEmail(user);
}
```

### E3

```typescript
// Callers binary-search this array by `createdAt`; it must stay sorted.
const orders = sortByCreatedAt(rawOrders);
```

### E4

The referenced note is in the same file, immediately below.

```typescript
// Customs gateway replica lag budget. Cause and removal condition are in the
// workaround below.
const CUSTOMS_REPLICA_SYNC_WAIT_MS = 300;

// Workaround(PROJ-1042): the gateway reads a replica that lags the primary, so
// a submit right after the status write sees a stale order. Remove this wait
// once gateway v3.2 ships event-based order pushes.
await sleep(CUSTOMS_REPLICA_SYNC_WAIT_MS);
```

### E5

```typescript
/**
 * EU EORI tax id: two uppercase country letters followed by 8 to 15
 * alphanumeric characters.
 * Matches: "DE123456789", "NL987654321B01"
 * Rejects: "123456789", "DE123"
 */
const EU_EORI_TAX_ID_REGEX = /^[A-Z]{2}[0-9A-Z]{8,15}$/;
```

### E6

```typescript
// We sleep 300ms because of the gateway replica. Do not remove.
await sleep(CUSTOMS_REPLICA_SYNC_WAIT_MS);
```

### E7

```typescript
// TODO: make this async someday
const response = await customsClient.submit(order.id, amount, taxId);
```

### E8

```typescript
// TODO(PROJ-892): remove the sleep once the Q4 event-push cluster is live.
await sleep(CUSTOMS_REPLICA_SYNC_WAIT_MS);
```

### E9

The build runs `mvn javadoc:javadoc` with doclint, so the tags are mandatory.

```java
/**
 * @param orderId the order id
 * @param amount the amount
 * @return the result
 */
public Result submit(String orderId, long amount) { ... }
```

### E10

```go
// SettleDirectMailOrder settles a direct-mail order and submits it for customs
// declaration. It returns ErrInvalidTaxID when the buyer's EORI number is
// malformed.
func SettleDirectMailOrder(order *Order) (bool, error) { ... }
```

### E11

```typescript
// SPDX-License-Identifier: Apache-2.0
// Copyright 2024 Example Corp.
```

### E12

```typescript
// eslint-disable-next-line no-console
console.log(`[boot] env=${env} build=${buildSha}`);
```

### E13

```typescript
// const oldRate = 0.09;
// if (isVip) { discount = oldRate; }
const rate = CROSS_BORDER_SURCHARGE_RATE;
```

### E14

```typescript
// Added per the user's request; rewritten by the assistant on 2026-10-18.
export function computeTotalCents(items: Item[]): number { ... }
```

### E15

```rust
let slice = unsafe { std::slice::from_raw_parts(ptr, len) };
```

### E16

This is the first block in `settlement.ts`.

```typescript
// Cross-border direct-mail settlement. Bonded-warehouse orders are settled by
// the batch pipeline and must not enter this path.

export function settleDirectMailOrder(order: Order): boolean { ... }
```

### E17

```typescript
/**
 * Locks the order for settlement. The caller must already hold the merchant
 * lock; this function does not acquire it. Returns false when the order is
 * already locked.
 */
export function lockOrder(order: Order): boolean { ... }
```

### E18

```typescript
// Legacy in-person-verified merchants keep the surcharge exemption until
// re-verification, so their status still bypasses the fee.
if (buyer.verificationStatus === 3) { ... }
```

### E19

```typescript
/**
 * Sorts ledger entries in place.
 *
 * Runs in O(n log n). Do not call this on the request path with more than
 * 10,000 entries; the p99 budget is the gateway timeout.
 */
export function sortLedger(entries: LedgerEntry[]): void { ... }
```

### E20

The project's doc convention does not record deprecation versions.

```typescript
/**
 * @deprecated Use `settleAndDeclareDirectMailOrder`. Removed when the batch
 * pipeline v2 ships. Callers that still pass bonded-warehouse orders break at
 * removal: the replacement rejects them.
 */
export function settleOrder(order: Order, buyer: Buyer): Promise<boolean> { ... }
```

### E21

```typescript
class LedgerEntry {
  // -1 means the balance has not been reconciled yet.
  reconciledAtMs: number;
}
```

### E22

```typescript
// Code generated by protoc-gen-ts. DO NOT EDIT.
export interface Order { ... }
```

### E23

```typescript
// Retry after 500 ms; the gateway replica lags by about 200 ms.
await sleep(300);
```

### E24

```typescript
// Settlement runs in three phases: validate the buyer, lock the order, then
// submit to customs. A phase-3 failure leaves the order LOCKED.
```

### E25

This is an exported function in a library; other packages call it.

```typescript
export function computeTotalCents(items: Item[]): number {
  return items.reduce((sum, item) => sum + item.unitPriceCents * item.quantity, 0);
}
```

### E26

```python
from settlement.models import Order

"""Settle a direct-mail order and submit it for customs declaration."""

def settle(order: Order) -> bool:
    ...
```

### E27

```typescript
// prettier-ignore
const matrix = [
  [1, 0],
  [0, 1],
];
```

### E28

```typescript
test("rejects a malformed EORI", () => {
  // Arrange
  const buyer = makeBuyer({ taxId: "DE123" });

  // Act
  const submit = () => settle(buyer);

  // Assert
  expect(submit).toThrow(InvalidTaxIdError);
});
```

### E29

```typescript
test("rejects a malformed EORI", () => {
  // expect the error
  expect(() => settle(buyer)).toThrow(InvalidTaxIdError);
});
```

### E30

This is a public open-source repository; readers cannot open the project's
internal JIRA.

```typescript
// See JIRA-8842 for why the retry budget is 3.
const RETRY_BUDGET = 3;
```

### E31

```typescript
// The build takes ~20 min, so the timeout is 30 min.
const BUILD_TIMEOUT_MS = 30 * 60 * 1000;
```

### E32

```typescript
export interface Order {
  id: string;
  status: OrderStatus;
  totalCents: number;
}
```

### E33

The reviewer checked: `PROJ-9999` does not exist.

```typescript
// Workaround(PROJ-9999): the gateway drops the first request after a cold start.
await sleep(300);
```

### E34

```typescript
// Staging gateway: https://internal-gw.corp.example, token sk-live-REDACTED
const GATEWAY_URL = process.env.GATEWAY_URL;
```

### E35

The file uses JDK 23 Markdown doc comments.

```java
/// Settles a direct-mail order and submits it for customs declaration.
///
/// @param order the order to settle; must be in `PENDING` state
/// @return true when the declaration was accepted
public boolean settle(Order order) { ... }
```

### E36

This file is not in the diff; the change touches a different file.

```typescript
// Retry after 500 ms.
await sleep(300);
```

### E37

```cpp
// foo.h
/// Does something important.
void example();

// foo.cpp
/// Does something important.
void example() { ... }
```

### E38

This is the first declaration in a public library package; there is no module
header.

```typescript
/**
 * Settles a direct-mail order and submits it for customs declaration. Returns
 * false for bonded-warehouse orders.
 */
export function settle(order: Order): boolean { ... }
```

### E39

The task asks for one change: make the `entityManager` doc say which proxy it
receives. Nothing else in the constructor is in scope.

```java
/// Creates the compiler.
///
/// @param entityManager entity manager, one per request
/// @param meta          the JPA metamodel
public JpaCompiler(EntityManager entityManager, JpaMeta meta) { ... }
```

### E40

The file documents its parameters; `buyer` follows the same shape.

```typescript
/**
 * @param order the order to settle; must be in `PENDING` state
 * @returns true when the declaration was accepted
 */
export function settle(order: Order, buyer: Buyer): boolean { ... }
```

---

## Part 2: answer key

Do not show this half when testing.

| Case | Verdict | Rule                         | Why                                                                                                                                               |
|------|---------|------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------|
| E1   | delete  | N1                           | `retries += 1` states it; the comment only narrates the statement.                                                                                |
| E2   | delete  | N2                           | Translates syntax into prose and adds nothing to the guard.                                                                                       |
| E3   | keep    | Warning                      | The ordering requirement is invisible in the code and breaks callers if violated.                                                                 |
| E4   | keep    | C7                           | States the unit and points to a note in the same file. The note is a workaround with a ticket and removal condition, not a current-state measurement. |
| E5   | keep    | C8                           | The regex is not readable as a rule; examples make it testable.                                                                                   |
| E6   | rewrite | C4, N12                      | Names a vague cause but not the specific upstream problem, the symptom, or the removal condition. `Do not remove` is a threat, not a reason. |
| E7   | rewrite | editing 6                    | No owner and no trigger.                                                                                                                          |
| E8   | keep    | editing 6                    | Owner, trigger, and reason are present, provided PROJ-892 is real; see N8.                                                                        |
| E9   | rewrite | java.md                      | Javadoc tags are required by the toolchain here, but all three echo the signature. Each must add a range, unit, or thrown condition.              |
| E10  | keep    | go.md                        | Name-first doc comment plus the error condition, which a caller cannot see from the signature.                                                    |
| E11  | leave   | scope                        | License header is policy; the skill does not govern it. Do not delete or reformat. |
| E12  | rewrite | C9                           | Suppression without a reason; add why the violation is intentional, in the tool's one-comment form.                                               |
| E13  | delete  | N5                           | Commented-out code; git holds the history.                                                                                                        |
| E14  | delete  | N3                           | Chat and AI residue, plus a dated banner. Keep nothing.                                                                                           |
| E15  | add     | C5, rust.md                  | No `// SAFETY:` invariant exists; add one immediately above the unsafe block, stating why it is sound and what must stay true. |
| E16  | keep    | C1                           | Module responsibility and the boundary that keeps bonded-warehouse orders out; no reader can reconstruct it from the file.                        |
| E17  | keep    | C2                           | Precondition (caller holds the lock), side effect, and return condition; none is visible from the signature.                                      |
| E18  | keep    | C3                           | The branch looks arbitrary without the policy; the rule explains why status 3 still bypasses the fee.                                             |
| E19  | keep    | C6                           | Caller-observable complexity and a hard limit/budget, not a current-state measurement; the caller needs the bound. |
| E20  | keep    | C10                          | Replacement, removal trigger, and what breaks. The project records no deprecation versions, so the removal trigger carries the version. |
| E21  | keep    | C11                          | Sentinel value the type cannot express; a reader would otherwise treat `-1` as a timestamp.                                                       |
| E22  | leave   | scope                        | Generated file; do not add, delete, or reformat. Change the generator.                                                                            |
| E23  | rewrite | editing 2, editing 5, C6         | The comment contradicts the code (`500` vs `300`), and "lags by about 200 ms" is an undated measurement. Trust the code, fix the prose or the value.                                                    |
| E24  | keep    | N1 exception                 | A phase overview at the function head is not step narration; it explains ordering and the failure state.                                          |
| E25  | add     | C2                           | Exported function has no contract; add preconditions, side effects, and thrown conditions.                                                        |
| E26  | rewrite | python.md placement          | The import is the module's first statement, so the string is an ordinary expression; move it into the body as the function's first statement.     |
| E27  | leave   | scope                        | `// prettier-ignore` is a machine directive, not prose; keep it and do not reformat. `keep` is an acceptable synonym; deleting it is the failure. |
| E28  | keep    | Tests                        | `Arrange`/`Act`/`Assert` are structural markers the file uses; they do not narrate assertion mechanics.                                           |
| E29  | delete  | Tests, N1                    | The test name and the assertion state it; the comment only narrates.                                                                              |
| E30  | rewrite | Pointer, C7                  | The ticket is unreachable to a public reader; state the retry-budget fact inline or link a public issue.                                          |
| E31  | rewrite | N11, C6                      | An undated current-state measurement rots; state the durable relationship (timeout sized past the slowest build) or date it and give the conditions. |
| E32  | add     | C12                          | Exported interface has no contract; document what an `Order` represents, its status lifecycle, and the unit/invariant of `totalCents`.            |
| E33  | rewrite | N8                           | Remove the fabricated ticket; keep the symptom only if it is verified, and state the removal condition.                                           |
| E34  | delete  | N9                           | Credentials and internal hostnames in a comment; remove it and rotate the token if it was real.                                                   |
| E35  | keep    | java.md                      | JDK 23 `///` form; the tags add a state precondition and a return condition.                                                                      |
| E36  | leave   | scope, editing 5             | The file is outside the diff; do not sweep untouched files. If it were touched, the contradiction would be a rewrite.                             |
| E37  | rewrite | c-cpp.md, one fact one place | The header contract is duplicated in the implementation; keep the header doc and use the `.cpp` for implementation notes.                         |
| E38  | add     | C1                           | Public package has no module header; add the boundary and non-goals.                                                                              |
| E39  | rewrite | D1, D2                       | Extend the tag the task names. `@param meta` stays: it is outside the change, and a tag set is never reduced one member at a time.                 |
| E40  | add     | D2, C2                       | The tag set exists but `buyer` is missing from it; add its `@param` with a constraint. Half a set is a defect, not a quieter style.               |

## Scoring

A run passes when it matches at least 37 of 40 verdicts **and** misses no
critical case.

Critical cases:
- Noise (the skill is letting noise through): E1, E2, E13, E14.
- Over-filter or scope (deleting or touching what must stay): E3, E10, E11,
  E16, E22, E24, E27, E36.
- Deletion gate (removing what the edit did not cover, or shrinking a tag set):
  E39.
- Rewrite obligations: E6, E7, E9, E12, E23.
- Missing comment: E15.
- New rules: E26, E34.

Everything else is coverage: a miss lowers the score but does not fail the run.
The notes below call out the coverage that matters most.

Notes:
- E11 is a scope case: `leave` is correct; `keep` is an acceptable label
  because it also leaves the header alone. Deleting or reformatting is the miss.
- E27 accepts `keep` or `leave`; both preserve the machine directive. Only
  deleting or rewriting it is a miss.
- E15 is an `add` case; a miss means the Rust SAFETY rule is not landing.
- E6, E7, E9, E12, and E23 are rewrite obligations; keeping them as-is is the
  failure.
- E25, E32, and E38 are the "add" coverage for missing C1, C2, or C12; a miss is
  a signal, not a gate failure.
- E28/E29 are test coverage; E30/E31 are pointer and measurement coverage;
  E35/E37 are language override coverage.
- E39 and E40 are the two directions of one rule: the tag set is whole or it is
  not there. E39 fails by dropping a member, E40 by tolerating a missing one.
- E39 doubles as the reproduction probe for the deletion gate. Run it on the same
  model against the pre-gate skill
  (`git show 5873159:skills/code-comment-guidelines/SKILL.md`) and against the
  current one: `delete` is the failing baseline, `rewrite` is the fix. Two
  identical verdicts mean the gate is insurance rather than the cause, and the
  question moves to whether the failing run had the skill in context at all.
  Measured 2026-10-23 on one model, one run per cell, with the tidy-up variant of
  E39 as the reproducing case: the pre-gate skill dropped the tag, the gated
  skill rewrote both tags and kept the set whole.
- E23 is the only in-scope case where comment and code disagree. E36 is the
  out-of-scope control: a wrong comment in an untouched file is left alone, not
  silently fixed.
