---
id: boundary-trust/validate-external-data
category: boundary-trust
phases: [design, coding, review]
tags: [webhook, schema-validation, provider-response]
triggers: [asserted-parsed-input, unchecked-webhook-payload, unvalidated-provider-response]
status: active
---

## Failure

Untrusted external data reaches trusted domain, persistence, or command logic without runtime validation at or before the trust crossing.

## Signals

Inspect network payloads, files, environment values, IPC messages, and provider responses. Type assertions and static types are routing signals, not validation.

## Evidence

Show the external origin, the data path into trusted logic, and the absence of prior runtime validation. Also show the expected shape or invariant that malformed data can violate.

## Bad

```ts
const event = JSON.parse(body) as PaymentEvent;
charge(event.amount, event.kind);
```

## Good

```ts
const raw: unknown = JSON.parse(body);
const event = PaymentEventSchema.parse(raw);
charge(event.amount, event.kind);
```

## Allow

Allow internal values that do not cross a trust boundary. Allow external input already validated at an earlier boundary when only the validated value continues. Opaque forwarding without interpretation is not trusted use.

## Fix

Parse into `unknown`, validate the required shape and domain invariants, reject invalid data, and pass only the validated value into trusted logic.
