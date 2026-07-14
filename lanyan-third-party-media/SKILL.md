---
name: lanyan-third-party-media
description: Use for LanYan third-party callbacks, request signing, authorization headers, WeChat Pay/refund/transfer, logistics submission, uploads, remote images, share posters, QR/image generation, bundled fonts, or production media verification. Validate external contracts exactly and verify the rendered output, not only code paths.
---

# LanYan Third-Party And Media

## External Contract

1. Compare method, URL, exact header format, timestamp/nonce/sequence, request body, encoding, and signature input with the provider documentation.
2. Keep signing/callback verification separate from business processing and record enough safe diagnostics to identify mismatches.
3. Select WeChat payment/refund/transfer protocol from available certificates, account capability, and provider prerequisites; do not mix V2 and V3 conventions.
4. Assemble logistics requests from the provider's actual order-key and delivery-mode requirements, omitting unsupported optional fields.

## Upload And Image Contract

- Decide whether upload APIs return a relative path or absolute URL. Prefix domains in exactly one layer.
- When moving static assets online, replace client references consistently and verify HTTPS accessibility.
- Generate a missing share poster from the target user/product data, persist its path, and regenerate only under an explicit invalidation rule.
- Bundle a known font with server-side poster generation; do not depend on an operating-system font.
- Compare poster output to the live background and reference image for dimensions, crop, whitespace, text, QR code, and cache behavior.

## Verify

Use an actual provider-style request or sandbox callback where available. Open the returned image URL, inspect its real pixels/content, and confirm the client renders the same resource.
