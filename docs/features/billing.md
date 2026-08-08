---
title: Billing & Cost Centers
description: Allocate print costs to teams, budgets and projects — cost centers, budgets, reservations and a full print ledger
---

# Billing & Cost Centers

Bambuddy has always been able to tell you what a print cost. Billing tells you
**whose budget it came out of** — and, when you want it, stops a print that would
go over.

It is aimed at installs where somebody has to be billed: a lab charging research
groups, a makerspace with member accounts, a department splitting machine time
between projects. It is entirely optional, off by default, and nothing about an
existing install changes until you turn it on.

---

## :material-cash-multiple: What it does

- **Cost centers** — a budget somebody can print against: a team, a course, a
  customer, a grant. Every user also gets a private one automatically.
- **Budgets** — an optional total or monthly limit per cost center. A center
  with no budget is explicitly unlimited, not blocked.
- **Reservations** — a queued print's cost is held against the budget while it
  waits, so ten queued jobs cannot each pass a check that only the last would
  fail.
- **A ledger** — every charge, deposit, withdrawal and adjustment, on one page,
  with the cost center it belongs to.
- **Server-side costing** — the estimate that budgets are enforced against is
  computed by Bambuddy, not sent by the browser.
- **Proportional charging** — a print that fails half way is charged for the
  filament it actually used.

---

## :material-toggle-switch: Turning it on

**Settings → Workflow → Billing**

| Setting | Default | Range | What it does |
|---|---|---|---|
| Billing enabled | Off | — | Turns the whole feature on. While off, no cost center is required, no budget is checked, and the **Finance** page is not in the sidebar. |
| Printer kill switch | Off | — | Stops prints that start without going through Bambuddy. Separate switch — see [Printer kill switch](#printer-kill-switch) below, and read it before enabling. |
| Budget reset day | `1` | 1–31 | Day of the month a monthly budget window restarts. Clamped on short months, so `31` means the last day of February. |
| Budget reset timezone | `UTC` | IANA name | The timezone that reset day is measured in, e.g. `Europe/Berlin`. Teams spread across timezones otherwise disagree about which month a print landed in. |

!!! note "The Finance page appears when billing does"
    The **Finance** entry sits between *Statistics* and *Settings* in the
    sidebar, and only while billing is enabled. If you have rearranged your
    sidebar, it appears wherever you put it.

---

## :material-account-group: Cost centers

A cost center is created and staffed on the **Finance** page by an administrator.

**Private cost centers** are created automatically, one per user, and are the
reason a single-user install works the moment billing is switched on. They
belong to their owner: nobody else can print against them, and they cannot be
edited or deleted by other administrators.

**Shared cost centers** are created by an administrator, who then adds members.
Each member carries a **can print** flag, so somebody can be given visibility of
a budget without being able to spend it.

Each center can carry a **total budget** (a fixed pot) or a **monthly budget**
(refreshed on the reset day above). A center with neither is shown as
**Unlimited** — printing against it is never blocked, and its spending is still
recorded.

!!! warning "A cost center in use cannot be deleted"
    Deletion is refused while any transaction or active reservation references
    the center, because removing it would rewrite the history of everyone who
    ever printed against it. Deactivate it instead — an inactive center cannot
    be selected for new prints, and its ledger stays intact.

---

## :material-scale-balance: Balances and budgets are not the same thing { #balances-vs-budgets }

This is the one thing worth reading twice.

| | What it means | Does it stop a print? |
|---|---|---|
| **Balance** | What a user has spent, deposited and been adjusted by | **No** |
| **Cost center budget** | The limit set on the center a print is billed to | **Yes** |

A balance is a record, not a credit limit. A user sitting at a negative balance
can still print; only the budget on the selected cost center can refuse a job.
If you want spending to stop somewhere, set a budget there.

A personal balance counts charges with no cost center, plus charges to the
user's **own private** center. A print billed to a **shared** center does not
move the personal balance — it belongs to the team that paid for it.

---

## :material-printer-3d: What happens when you print

1. **Pick a cost center.** With billing on, the print dialog shows an estimated
   cost and a cost center picker. If you have no active center you are allowed
   to print against, the dialog says so and the print is refused — ask an
   administrator for print access on one.
2. **The budget is checked** against what the center has already spent in the
   current window, plus anything reserved for prints that have not run yet. If
   the job does not fit, it is refused with the amounts in the message.
3. **A reservation is taken** when the item is queued, and released if the print
   never happens.
4. **The charge is written when the print ends**, from the archive's measured
   cost rather than the estimate.

!!! info "The estimate and the charge are different numbers"
    The estimate is what the file and your printer's rates predict; the charge
    is what the print actually consumed. They will rarely be identical, and the
    charge is the one that lands in the ledger. The figure in the dialog is a
    display hint only — budget enforcement uses Bambuddy's own calculation, so
    editing it in the browser cannot buy a bigger budget.

**Aborted prints are charged proportionally.** A job stopped part way is billed
for the filament it consumed, not for the whole job.

**A job is never charged twice.** Each dispatch carries its own billing
identity, so a reprint of the same file is a new charge, a restart mid-print
does not produce a second one, and a charge that was deleted by an administrator
does not come back on the next reprint.

**A charge that fails is loud.** If a charge cannot be written, Bambuddy reports
it in the interface and through any notification provider subscribed to the
**Billing charge failed** event, rather than quietly not billing.

---

## :material-book-open-variant: The Finance page

Users see their own balance and ledger. Administrators additionally get an
**Admin view** covering every user and cost center:

- **Cost centers** — create, rename, deactivate, set total or monthly budgets,
  add and remove members, and set who may print
- **Transactions** — filter the whole ledger by user, cost center and type;
  edit or delete an entry; move a transaction back to a personal account
- **Deposits and withdrawals** — credit or debit a user's account
- **Manual print charge** — record a job that happened outside Bambuddy

Amounts use the currency configured in **Settings → General**.

---

## :material-shield-lock: Permissions

| Permission | Grants |
|---|---|
| `cost_centers:read_own` | See your own balance, ledger and the cost centers you may print against |
| `cost_centers:read_all` | See every user's balance and the full ledger |
| `cost_centers:modify` | Edit cost centers, budgets, members and transactions |
| `cost_centers:create` | Create new cost centers |

The default **Administrators** group holds all four. **Operators** hold
`cost_centers:read_own`, so they can see their own spending and print against
the centers they belong to. Grant the rest to whoever does your accounting.

---

## :material-alert-octagon: Printer kill switch { #printer-kill-switch }

**Settings → Workflow → Billing → Printer kill switch.** Off by default, and
separate from billing itself.

An install that bills its users can be bypassed by sending a job straight to the
printer from Bambu Studio or Handy — that print is not queued, not costed and
not billed. With the kill switch on, Bambuddy stops such a print as soon as it
sees it, raises an alert in the interface, and sends the configured print-stopped
notification.

!!! danger "This stops prints, and a stopped print is not recoverable"
    Enable it only where prints genuinely must not start outside Bambuddy. It
    acts on any print it can show did not come from Bambuddy, including one
    started deliberately by an administrator at the machine.

It is deliberately cautious about what it can prove. When ownership of a running
print cannot be established — during the moments after a Bambuddy restart, or
while a job Bambuddy dispatched is still being recorded — it does nothing and
logs that it deferred. Stopping a print is irreversible; declining to act costs
a log line.

---

## :material-alert-circle-outline: Limitations

- **Balances do not gate printing** — only cost center budgets do. See
  [above](#balances-vs-budgets).
- **Monthly budgets reset, they do not roll over.** Unspent budget from last
  month is not added to this month's.
- **The kill switch works on prints it observes**, so it acts after the printer
  has started rather than preventing the start.

---

## :material-help-circle: Troubleshooting

**"No active cost center is available for printing"** — you are not a member of
any active cost center with print rights. An administrator can add you on the
Finance page, or check that the center has not been deactivated.

**A print was refused for budget** — the message carries the requested amount
and what remains. Remember that queued prints hold reservations: cancelling a
queued job releases its reservation immediately.

**The Finance page is missing from the sidebar** — billing is off, or your
account lacks `cost_centers:read_own`.

**A balance looks wrong after an administrator edited a transaction** — balances
are recomputed from the ledger, so the ledger is the source of truth. If a
figure still looks wrong, an administrator can rebuild the balance ledger from
the Finance page.
