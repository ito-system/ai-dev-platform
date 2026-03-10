# Example: Validation Flow

This example shows a validation failure and resolution.

---

## Scenario

After running `playbooks/saas/add-feature.md` for a payments feature, validation fails.

---

## Validation run 1: FAIL

### validate-backend result

```
[FAIL] Context propagation: PaymentUseCase.Process missing context.Context parameter
       File: internal/usecase/payment_usecase.go:23

[FAIL] Tests exist: No test file found for payment_usecase
       Expected: internal/usecase/payment_usecase_test.go

Overall: FAIL
```

### validate-database result

```
[FAIL] Down migration missing: 20240316090000_add_payments_table.sql has no migrate:down section

Overall: FAIL
```

---

## Resolution

Fixes applied:
1. Added `ctx context.Context` as first parameter to `PaymentUseCase.Process`.
2. Created `internal/usecase/payment_usecase_test.go` with happy path and error case.
3. Added `-- migrate:down` section to `20240316090000_add_payments_table.sql`.

---

## Validation run 2: PASS

```
validate-backend:  PASS
validate-database: PASS
validate-frontend: PASS
validate-feature:  PASS
```

Playbook proceeds to commit phase.

---

## Key takeaway

Validation is not optional. It catches issues that would otherwise cause bugs in production.
Fix all FAIL results before committing.
