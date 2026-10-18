# Worked example: order settlement and customs declaration

A before/after pair for one TypeScript module. The before version is a realistic pile of comment anti-patterns: names that need a comment to be read, step narration, chat residue, dead code, and a scary workaround note that explains nothing. The after version keeps only the comments the code cannot carry.

The example also extracts `calculateFinalSettlementCents` so several comments can disappear. That extraction is what "refactor instead of explaining" means; it is not a license to rewrite unrelated code.

## Before

```typescript
// Author: Zhang San
// 2025-11-03 Li Si: fixed the amount precision bug
// Added the direct-mail branch the PM asked for in the group chat

const REG = /^[A-Z]{2}[0-9A-Z]{8,15}$/; // tax id regex

/**
 * Handles the order
 * @param o order object
 * @param u user object
 * @returns a boolean
 */
export async function handle(o: any, u: any): Promise<boolean> {
  // Step 1: check the arguments
  if (!o || !u) {
    return false;
  }

  // Step 2: if the order type is not 1, return false
  if (o.type !== 1) {
    return false;
  }

  // Step 3: validate the tax id format
  if (!REG.test(u.taxId)) {
    throw new Error("Invalid tax ID");
  }

  // Step 4: calculate the amount and the surcharge
  // fixed the float precision problem mentioned earlier
  let d = o.amt;
  // old VIP discount, commented out in case we need it again
  // if (u.isVip) {
  //   d = d * 0.9;
  // }
  if (u.st === 3 && d > 100000) {
    // if the user status is 3 and the amount is above 100000, skip the surcharge
    d = d + 0;
  } else {
    d = d + Math.round(d * 0.12);
  }

  // update the order amount and status
  o.finalAmt = d;
  o.status = "LOCKED";

  // Step 5: sleep for 300 milliseconds
  // WARNING: this looks dumb but DO NOT TOUCH IT. Someone deleted it once and production broke!
  await new Promise((r) => setTimeout(r, 300));

  // Step 6: call the customs declaration API
  // TODO: make this async someday
  const res = await customsClient.submit(o.id, o.finalAmt, u.taxId);

  // return true if the call succeeded, otherwise false
  if (res.code === 200) {
    return true;
  } else {
    return false;
  }
}
```

## After

```typescript
// Cross-border direct-mail order settlement and customs declaration.
//
// Boundary: this module only handles direct-mail orders. Bonded-warehouse
// orders are settled by the batch pipeline and must not enter this path.

/**
 * EU EORI tax id: two uppercase country letters followed by 8 to 15
 * alphanumeric characters.
 * Matches: "DE123456789", "NL987654321B01"
 * Rejects: "123456789" (no country code), "DE123" (too short)
 */
const EU_EORI_TAX_ID_REGEX = /^[A-Z]{2}[0-9A-Z]{8,15}$/;

// Legacy status code for B2B merchants verified in person before the 2023
// system migration. They keep the surcharge exemption until re-verification.
const LEGACY_VERIFIED_MERCHANT_STATUS = 3;

// Surcharge-exempt threshold in cents (100,000 cents = 1,000.00 CNY).
const SURCHARGE_EXEMPT_THRESHOLD_CENTS = 100_000;

// 12% cross-border handling fee. Applied to integer cents and rounded once,
// because IEEE 754 multiplication drifts on amounts such as 0.1 + 0.2.
const CROSS_BORDER_SURCHARGE_RATE = 0.12;

// Customs gateway replica lag budget, in milliseconds. Cause and removal
// condition are in the workaround below.
const CUSTOMS_REPLICA_SYNC_WAIT_MS = 300;

/**
 * Settles a direct-mail order and submits it for customs declaration.
 *
 * Precondition: only direct-mail orders are handled; any other shipping type
 * returns false without touching the order.
 * Side effects: mutates `order` in place (writes `finalAmountCents`, sets
 * `status` to LOCKED) and performs a network call to the customs gateway.
 * Throws: when the buyer's tax id is not a valid EU EORI number. Callers must
 * catch this and route the buyer back to the profile form.
 */
export async function settleAndDeclareDirectMailOrder(
  order: Order | null,
  buyer: Buyer | null
): Promise<boolean> {
  if (!order || !buyer || order.shippingType !== ShippingType.DIRECT_MAIL) {
    return false;
  }

  if (!EU_EORI_TAX_ID_REGEX.test(buyer.taxId)) {
    throw new Error(`Invalid EU EORI tax ID format: ${buyer.taxId}`);
  }

  order.finalAmountCents = calculateFinalSettlementCents(order.amountCents, buyer);
  order.status = OrderStatus.LOCKED;

  // Workaround(PROJ-1042): the customs gateway reads from a replica that lags
  // the primary. Submitting right after the status write makes the gateway read
  // a stale order and return 404. Remove this wait once gateway v3.2 ships
  // event-based order pushes.
  await sleep(CUSTOMS_REPLICA_SYNC_WAIT_MS);

  // TODO(PROJ-892): replace this blocking call with a queue dispatch after the
  // Q4 cross-region queue cluster is provisioned.
  const response = await customsClient.submit(
    order.id,
    order.finalAmountCents,
    buyer.taxId
  );

  return response.code === 200;
}

function calculateFinalSettlementCents(
  baseAmountCents: number,
  buyer: Buyer
): number {
  // Compliance policy: legacy in-person-verified merchants pay no handling fee
  // on orders above 1,000.00 CNY, to encourage large B2B purchases.
  const isExemptFromSurcharge =
    buyer.verificationStatus === LEGACY_VERIFIED_MERCHANT_STATUS &&
    baseAmountCents > SURCHARGE_EXEMPT_THRESHOLD_CENTS;

  if (isExemptFromSurcharge) {
    return baseAmountCents;
  }

  const surchargeCents = Math.round(baseAmountCents * CROSS_BORDER_SURCHARGE_RATE);
  return baseAmountCents + surchargeCents;
}
```

## Others that pass: warning, safety, suppression, deprecation, data

The module above demonstrates *Contract*, *Why*, and *Pointer*. Warning and
Safety are the gate's two remaining kinds; suppressions, deprecations, and
exported data are location-mandated (C9, C10, C11). The file header is a plain
comment block on purpose: a `/** */` block there would attach to the first
declaration in TypeScript and never reach the generated file docs.

```typescript
// Warning: callers render this list directly and binary-search it by
// `createdAt`. Returning unsorted data breaks pagination without an error.
function listRecentOrders(merchantId: string): Order[] { ... }
```

```typescript
// eslint-disable-next-line no-console -- boot banner must reach stdout before the logger is configured
console.log(`[boot] env=${env} build=${buildSha}`);
```

```typescript
/**
 * @deprecated Use `settleAndDeclareDirectMailOrder`. This function still accepts
 * bonded-warehouse orders, which the new one rejects; it is removed when the
 * batch pipeline v2 ships.
 */
export function settleOrder(order: Order, buyer: Buyer): Promise<boolean> { ... }
```

```rust
// Pointer: DECIMAL(18,6) is the wire format the ledger reads. Widening it
// needs a coordinated change with the settlement service (SPEC-441).
const LEDGER_SCALE: u32 = 6;

// SAFETY: `ptr` comes from `values.as_ptr()`, `len` is `values.len()`, and
// `values` is not mutated or dropped for the lifetime of the returned slice.
let slice = unsafe { std::slice::from_raw_parts(ptr, len) };
```

```typescript
class LedgerEntry {
  // -1 means the balance has not been reconciled yet.
  reconciledAtMs: number;
}
```

```typescript
test("rejects a malformed EORI", () => {
  // Fixture: the buyer's EORI is too short, so settlement throws before any
  // network call; no gateway stub is needed.
  const buyer = makeBuyer({ taxId: "DE123" });
  const order = makeOrder({ shippingType: ShippingType.DIRECT_MAIL });
  expect(() => settleAndDeclareDirectMailOrder(order, buyer)).toThrow(
    InvalidTaxIdError
  );
});
```

Each of these would be dropped by a naive "no comment by default" reading, and
each passes the gate: none is derivable from the code, and each changes what a
caller or maintainer may do.

## What changed

| Anti-pattern in the before version | Action taken |
| --- | --- |
| Step narration (`Step 1` through `Step 6`) and the closing "return true if it succeeded" | Deleted. Guard clauses and the extracted `calculateFinalSettlementCents` make the flow readable without commentary. |
| Author banner, dated edit log, "the PM asked for this in the group chat" | Deleted. The product requirement became a domain rule on the exemption branch; history stays in git. |
| Commented-out VIP discount | Deleted outright. |
| `// tax id regex` | Replaced with the actual EU EORI rule plus two matching and two non-matching examples. |
| Magic numbers `3`, `100000`, `0.12` | Named constants with units (cents, milliseconds) and the reason each value exists. |
| "DO NOT TOUCH, production broke once" | Rewritten as a workaround with the cause (replica lag), the symptom (404), the tracking issue, and the removal condition. |
| Undeclared mutation of the input order and the network call | Declared as side effects in the exported function's header. |
| Bare `// TODO: make this async someday` | Rewritten with an owner, a trigger, and the reason it is not done yet. |
