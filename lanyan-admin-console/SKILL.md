---
name: lanyan-admin-console
description: Use for LanYan Vue2 ElementUI admin pages, menus, permissions, business lists, filters, master-detail views, team members, category management, or operational CRUD. Design usable operator workflows with business labels, constrained actions, and server-backed data.
---

# LanYan Admin Console

## Shape The Operator Flow

1. Start from the operator's recurring task: filter, scan, inspect, edit, process, or remove.
2. Show readable business information in list columns; use IDs only as secondary trace information.
3. Put dense or multi-field information in a detail dialog/drawer with clear field grouping.
4. Keep only actions supported by the current business state and permission. Remove obsolete edit/delete controls instead of leaving dead buttons.
5. Load filter candidates from their real source and restore sensible defaults after the keyword is cleared.

## Common Layout Rules

- Use status tabs when status is the primary way operators divide work.
- For two-level categories, make the first level the selection pane and show its children in the work pane. Only render image controls where that level owns images.
- For member/team pages, open member detail from the selected member ID; never reuse the current administrator's identity or poster.
- Keep menu, route, button permission, and backend endpoint changes aligned.

## Data Contract

- Add display fields to the backend VO instead of resolving foreign keys in the browser.
- Make list queries and detail queries return the fields the screen actually needs.
- Preserve Vue2 form reactivity: populate existing reactive fields instead of replacing the whole form object when selection state is involved.

## Verify

Check first load, filtering, empty state, clear/reset, detail opening, allowed actions, permission-hidden actions, create/edit persistence, and refresh.
