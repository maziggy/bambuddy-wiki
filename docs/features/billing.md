---
title: Billing & Cost Centers
description: Manage cost centers, wallets, transactions, and monthly budget windows
---

# Billing & Cost Centers

Billing and cost centers are an optional feature to allocate, control, and audit printing costs across teams and projects. With this feature you can:

- Show the estimated cost in front of a print and check if the estimated cost is still in the available budget (if set).
- Assign budgets to teams/users to track costs where they belong.
- Prevent accidental overspend with pre-print budget reservations and configurable monthly budget windows.
- Streamline accounting and chargebacks by recording deposits, withdrawals, and adjustments in one place.
- Support distributed teams with timezone-aware budget boundaries and flexible reset days.
- Prevent prints from outside of BamBuddy to ensure all costs are tracked.
- If a print is aborted, the partial filament consumed is used to calculate the costs.

---

## Budget Validation Flow

1. When billing is enabled, print requests have to specify a cost center and must pass budget validation.
2. The system computes:
   - `used` = sum of negative transactions for the cost center (optionally limited to current budget window)
   - `reserved` = sum of open queue-item reservations + active budget reservations
   - `available` = budget_limit - used - reserved
3. If `requested` > `available` the print is rejected with a clear error.
4. A **Budget Reservation** is created at queue-time to hold funds until the print completes (or is released).
