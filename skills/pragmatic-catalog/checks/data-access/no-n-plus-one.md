---
id: data-access/no-n-plus-one
category: data-access
phases: [design, coding, review]
tags: [query, collection, cardinality]
triggers: [query-in-loop, per-item-resolver, repeated-remote-read]
status: active
---

## Failure

A collection path performs database or remote access per item, so the access count grows with collection cardinality.

## Signals

Inspect per-item resolvers, repository calls in iteration, lazy relation loading, and repeated remote reads. Signals only select this check.

## Evidence

Show the collection source, the access inside its item path, and how the number of accesses grows as the collection grows. A loop, recursion, or `await` alone is not evidence.

## Bad

```ts
const orders = await db.order.findMany();
for (const order of orders) {
  order.customer = await db.customer.findById(order.customerId);
}
```

## Good

```ts
const orders = await db.order.findMany();
const ids = [...new Set(orders.map(order => order.customerId))];
const customers = await db.customer.findManyById(ids);
const byId = new Map(customers.map(customer => [customer.id, customer]));
for (const order of orders) order.customer = byId.get(order.customerId);
```

## Allow

Allow iteration over data already in memory. Allow access whose count is fixed and independent of input cardinality. Do not report a batched read followed by in-memory lookup.

## Fix

Batch or prefetch the needed records, then join in memory. If batching is unavailable, bound the access count explicitly and preserve required ordering and failure behavior.
