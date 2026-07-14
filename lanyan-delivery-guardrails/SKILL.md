---
name: lanyan-delivery-guardrails
description: Use for LanYan/RuoYi business changes or production fixes spanning database, Spring Boot, Vue2 ElementUI admin, uni-app mini program, H5, uploads, screenshots, curl requests, or live error reports. Map the complete business chain, trace every field and state, and verify the actual operator and user flows before completion.
---

# LanYan Delivery Guardrails

## Execute

1. Turn each request, screenshot, table row, curl sample, and error into a numbered acceptance item.
2. Map the object across `SQL -> Entity/Mapper -> Service -> Controller/VO -> admin API/view -> mini program/H5 view -> user result`.
3. For every changed field, state who maintains it, who returns it, who displays it, and who submits it.
4. Check schema and deployed SQL before coding. Do not add a mapper field that the target database lacks.
5. Change every affected surface in one delivery; do not stop after a backend-only or page-only change when the business path crosses surfaces.
6. Verify with the shortest real path: create or edit data in admin, read it through the API, use it on the client, then confirm the final state and messages.

## Guardrails

- Treat screenshots, online tables, curl, production URLs, and logs as evidence, not hints. Link each to a concrete diagnosis or unresolved item.
- Return business labels and aggregates in a VO; do not make an admin list show foreign-key IDs as its usable value.
- Put business state transitions and side effects in one service-layer entry point. Test and production entrances must reuse it.
- Use backend validation and fallbacks for essential flows. The client must not be the only guardian of a rule.
- When DDL cannot be deployed immediately, keep the release runnable with an explicit fallback and provide a separate upgrade SQL.

## Finish

Report completed items, executed checks, and any item that still needs deployment, data repair, or real-device verification.
