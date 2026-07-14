---
name: lanyan-team-operations
description: Use for LanYan teams, small/large team capacity, member lists, rewards, recharge targets, monthly settlement, promotion eligibility, announcement generation, birthday notices, scheduled jobs, or team status indicators. Keep rules configurable, calculations server-owned, and time/event triggers idempotent.
---

# LanYan Team Operations

## Rule Ownership

- Read team capacity, monthly recharge target, and reward points from admin-managed settings.
- Calculate team qualification, reduction, suspension, restoration, and status color on the backend.
- Expose a final status and reason for the client to display; do not ask the client to infer eligibility from scattered fields.

## Team Membership

- In a user's own small team, place that user first when the business view requires it.
- Use the selected member's identity for member details and poster generation.
- Enforce capacity when joining and provide operator actions such as member removal only where rules permit.

## Settlement And Notices

1. Make monthly settlement idempotent per team/month/reward type.
2. At month start, settle rewards and reset the required monthly accumulation fields.
3. At year start, reset birthday-gift eligibility.
4. Generate birthday and full-team upgrade announcements both from scheduled reconciliation and the immediate registration/team event.
5. Deduplicate announcements by type, business object/user, and date or event key.

## Verify

Cover qualifying, first failure, reduced reward, promotion suspension, restored qualification, capacity boundary, manual reopen, scheduler rerun, and immediate event generation.
