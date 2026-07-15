---
name: lanyan-operation-content
description: Use for LanYan operational settings, bus_setting values, service agreements, privacy policy, platform introduction, points rules, customer-service information, team parameters, rich text, or UEditor. Keep runtime business content editable in admin and consistently rendered by clients.
---

# LanYan Operations And Content

## Split Configuration By Ownership

- Put operator-managed business content and runtime rules in grouped settings with a stable `groupKey` and `valueKey`.
- Put third-party credentials and deployment secrets in system configuration or environment variables.
- Put project build/static defaults in code configuration.
- Let the mini program and H5 consume a published configuration result, not configuration schema logic.

## Build The Admin Surface

1. Use clear operation-oriented groups such as account, team, contact, agreement, prize rule, and platform introduction.
2. Match input controls to value type: numeric inputs for amounts, image upload for QR/image fields, and rich text for articles.
3. Show only fields belonging to the selected group; do not expose developer-only settings as generic CRUD.
4. Supply empty-value fallback behavior for client pages.

## Rich Text / UEditor

- Treat editor initialization, visible container timing, value write-back, upload response mapping, preview, and XSS policy as one integration.
- Initialize only after the container is visible; avoid input handlers that continuously reset editor content and move the cursor.
- Adapt the upload response to the editor's expected protocol and retain the safe HTML tags needed by trusted admin content.
- When an admin setting endpoint saves rich text or UEditor HTML, add that endpoint to `application.yml` `xss.excludes` so the global XSS filter does not strip trusted operational content before persistence.

## Verify

Edit a setting in admin, save, reload it via the public/client API, and validate formatted text, images, phone links, numeric rules, XSS exclude coverage, and empty values.
