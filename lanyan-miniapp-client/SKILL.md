---
name: lanyan-miniapp-client
description: Use for LanYan uni-app/WeChat mini program pages, API wrappers, request headers, static image URLs, navigation, forms, share views, toast duration, or user-facing status display. Keep client behavior aligned with backend contracts and mini-program platform constraints.
---

# LanYan Mini Program Client

## Implement

1. Reuse the project's request wrapper and obtain base URLs, headers, auth, and error handling from one place.
2. Define the image contract once: backend returns relative paths or absolute URLs; the client prefixes only when required and never twice.
3. Render server-computed business statuses and configuration values. Do not duplicate reward, eligibility, or permission rules in page code.
4. Route by product/order type deliberately. A direct-exchange voucher must not pass through an unnecessary confirmation page.
5. Keep backend error text visible long enough to read; do not replace meaningful messages with a generic short toast.

## Platform Checks

- Use HTTPS-compatible remote resources and keep static resource paths configurable.
- Keep page initial state stable while async requests complete; handle loading, empty, failure, and retry states.
- Build sharing from the target user's ID/data, especially on team-member pages; never silently use the logged-in user's poster.
- For specialized input such as vehicle plates, use a deliberate input mode or component rather than relying on an unsuitable generic keyboard.

## Verify

Test fresh entry, logged-in entry, failed request, no-data case, image rendering, share result, back navigation, and a real-device or simulator request trace.
