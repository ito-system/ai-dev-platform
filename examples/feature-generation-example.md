# Example: Feature Generation

This example shows running `playbooks/tob-saas/add-feature.md` to add a new feature.

---

## Inputs provided

```
product_name: "OnboardFlow"
feature_name: "Email notifications"
user_story: "As a new hire, I want to receive email reminders for overdue checklist items so that I don't miss onboarding tasks."
repo_path: "./onboardflow"
```

---

## Phase 1: Feature spec output

**Skill: generate-feature**

Acceptance criteria:
- [ ] System sends email reminder when checklist item is 1 day overdue
- [ ] Email contains item name, description, and due date
- [ ] New hire can unsubscribe from reminders
- [ ] HR manager can enable/disable notifications per checklist

API endpoints:
- `PATCH /api/v1/checklists/{id}/notifications` — toggle notifications
- `DELETE /api/v1/notifications/unsubscribe` — unsubscribe

Data model changes:
- `checklists` table: add `notifications_enabled BOOLEAN DEFAULT true`
- New table: `notification_preferences` (`user_id`, `unsubscribed_at`)

UI components:
- `NotificationToggle`: toggle switch on checklist settings page
- `UnsubscribeConfirmation`: confirmation dialog

---

## Phase 2: Generated files

- `internal/domain/notification.go`
- `internal/usecase/notification_usecase.go`
- `internal/handler/notification_handler.go`
- `internal/repository/notification_repository.go`
- `migrations/20240315120000_add_notification_preferences.sql`
- `src/features/notifications/NotificationToggle.tsx`
- `src/features/notifications/NotificationToggle.test.tsx`

---

## Phase 3: Validation output

- validate-database: PASS
- validate-backend: PASS
- validate-frontend: PASS
- validate-feature: PASS

---

## Commit

`feat: add email notifications for overdue checklist items`
