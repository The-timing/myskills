---
name: lanyan-backend-data
description: Use for LanYan Spring Boot, MyBatis-Plus, database schema, Entity/VO/DTO, Mapper, Service, Controller, join query, public API, authentication boundary, callback, or business-state implementation. Apply project-style layering and protect data contracts from schema to response.
---

# LanYan Backend And Data

## Build In Layers

1. Confirm table fields, defaults, indexes, migration SQL, and historical-data compatibility.
2. Keep persistence entities limited to real columns. Mark calculated, joined, or UI-only fields as non-persistent.
3. Expose client/admin data through VO or DTO; aggregate names, labels, and counts on the server.
4. Put validation, idempotency, state changes, ledger writes, and business side effects in a service method.
5. Let controllers handle request/response and permissions only; mapper methods handle data access only.

## Query And API Rules

- Use stable sorting for lists: parent/order/id or an equivalent deterministic sequence.
- Prefer a targeted join or batch aggregate over client-side N+1 lookups.
- Separate anonymous read endpoints, authenticated user extensions, and admin management endpoints. Do not weaken a public endpoint to support an admin action.
- When implementing miniapp user registration, validate that the submitted phone is unique before creating the account; protect the check with a short lock when concurrent registration can reuse the same phone.
- For multi-match upgrades, retain compatibility with the previous single-object field until callers are migrated.
- Pair every occupancy, reservation, lock, or hold with release paths for success, failure, timeout, cancel, and retry.

## Callback Rules

- Verify signature and authorization before processing business data.
- Make duplicate callbacks safe with a unique business key and idempotent state checks.
- Keep the primary transaction short. Trigger slow external notifications or supplemental actions after commit and with retry-safe handling.

## Verify

Exercise normal, duplicate phone, insufficient-data, canceled, and retried requests. Confirm database rows, returned VO fields, and downstream state match.
