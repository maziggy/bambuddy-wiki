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
- **A charging policy** — a charge is the filament by default; you choose
  whether completed and failed prints also pay their
  [energy and printer wear](#charging-policy), and how a failed print's
  filament is charged.

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
renamed, deactivated or deleted. An administrator can set their budget; a
budget of 0 refuses every print with an estimated cost.

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

**Every print is charged at the spools it prints with.** When a print starts,
Bambuddy estimates its cost: its plate's planned filament at the spools mapped
for it, which is the mapping chosen in the print dialog or stored on the queue
item, or the loaded trays whose colours match. The print is charged from that
start estimate. A reprint is priced the same way, so printing an archive again
on a spool that cost more, or after the spool's price changed, is charged at
the new price. A first print of one plate of a multi-plate file is charged
from that plate's own figure as well, not from the whole file's cost scaled to
the plate's share.

The charge is the estimate made at the start, not the cost measured at the
end. The [Print Log](print-log.md) shows each print's measured cost, which can
differ from what was charged; in Spoolman mode that measured cost may not be
priced from the spool at all. A completed print with no measured cost, a first
print included, is logged at what it was charged. The archive card keeps
showing its [first print's cost](archiving.md).

If a print ends before its start estimate was worked out (it failed within
seconds, or Bambuddy restarted while it printed), it is priced when it ends,
from the spools in its trays at that moment.

A print Bambuddy could not price from its spools (no mapping, no spool in the
mapped trays, or the file is gone) is charged from the archive's cost. The
budget a queued reprint reserves is priced like its charge when the queue item
has a spool mapping, and is the archive's cost without one. It is worked out
when the job is queued and again when it is sent to the printer. By then the
queue has mapped the job's filaments to the printer's slots, so a job queued
without a mapping is priced at its spools there. The charge can still differ
from the reservation if a spool or its price changes before the print starts.
In Spoolman mode the reservation finds a spool the way the charge does: by its
RFID tag first, then by its slot assignment in Bambuddy.

A spool of the built-in inventory priced 0 counts as free. A print only on
such spools is charged nothing and reserves nothing, and it is not refused
for having no estimated cost. This applies to the built-in inventory only: in
Spoolman a price of 0 means the spool has no price, and its filament is
charged at the default filament cost.

A built-in spool with no price at all is not a free one. Its filament is
charged at the default filament cost. If that is set to 0, such a spool does
not price the print: a print only on spools without a price is charged from
the archive's cost, and reserves that, not 0.

With billing off, prints and queued jobs are not priced for a charge, and
queueing a job does not ask Spoolman for prices.

**Aborted prints are charged proportionally.** A job stopped part way is billed
for the filament it consumed, not for the whole job.

**A job is never charged twice.** Each dispatch carries its own billing
identity, so a reprint of the same file is a new charge, a restart mid-print
does not produce a second one, and a charge that was deleted by an administrator
does not come back on the next reprint.

**A print is billed to the user who queued it.** A reprint of someone else's
archive is charged under the person who sent it, not the archive's owner.

**Deleting a user does not cancel the cost of a print still running.** It is
charged to its cost center when it ends, including the user's own private center
(which stays after them), and shows **No user** in the ledger, as do prints that
never had a user (login turned off). Deleting the user together with their items
releases the budget their queued prints still held; a print of theirs that is
still running is kept until it ends, without a user, and billed the same way.

**A charge that fails is loud.** If a charge cannot be written, Bambuddy reports
it in the interface and through any notification provider subscribed to the
**Billing charge failed** event, rather than quietly not billing.

---

## :material-lightning-bolt: Print charges: energy, wear and failed prints { #charging-policy }

By default a charge covers the filament: all of it for a completed print, the
share it used for a print that failed or was cancelled. Bambuddy also works out
two more costs for every print: its [energy](energy.md), measured by the
printer's smart plug, and its
[printer wear](statistics.md#printer-wear-cost), the print's duration at the
printer's wear cost per printing hour. The charging policy decides which of
these a print pays.

**Settings → General → Cost Tracking → Print Charges**, shown while
billing is enabled:

| | Setting | Default | What it does |
|---|---|---|---|
| **Completed prints** | Energy Cost | Off | Adds the print's energy cost. Needs a smart plug with energy monitoring on the printer. |
| | Wear Cost | Off | Adds the print's wear cost. Needs a wear cost per printing hour on the printer; a printer without one adds nothing. |
| **Failed and cancelled prints** | Filament charge | Amount used | **Amount used** charges the filament in proportion to what the print consumed. **Full print** charges it as if the print had completed. **Not charged** charges no filament. |
| | Energy Cost | Off | Adds the energy the print used before it stopped. |
| | Wear Cost | Off | Adds the wear for the time the print ran. |

A completed print always pays its filament. With the defaults, charges are
exactly as before.

!!! note "Energy and wear do not depend on the filament"
    Energy Cost and Wear Cost are charged whenever they are switched on for
    how the print ended, also when its filament costs nothing: a print on
    free spools still used the printer and the power.

The charge stays **one ledger entry**, and its description lists what it is
made of:

```
Print charge: Benchy (filament 1.20 + energy 0.08 + wear 0.25)
Print charge: Benchy (filament 0.60 + energy 0.08) [failed: 50.0% filament (50.0g/100.0g)]
Print charge: Benchy (filament 1.20; filament charged in full) [failed: full filament]
Print charge: Benchy (energy 0.08 + wear 0.25; filament not charged) [failed: filament not charged]
```

The part in square brackets is how a print that did not finish ended and what
was charged for its filament. The Finance page shows it as a badge on the
entry instead of in the description.

- **Energy and wear are the print's actual values**, also for a print that did
  not finish: what the plug measured and how long the printer ran.
- **A print that pays no filament still pays its energy and wear**, if those
  are switched on for how it ended. That covers a completed print with no
  filament cost (free spools, an archive with no filament data), a failed
  print under **Not charged**, and a failed print Bambuddy has no filament
  data for. Its description then lists only what was charged:
  `Print charge: Benchy (energy 0.08 + wear 0.25)`. With nothing left to
  charge, no entry is written and the print's reservation is released.
- **Energy is read a few seconds after the print ends.** A print that pays
  energy, and whose plug was read at its start, is charged once the plug has
  been read again, instead of at the moment it ends. Every other print is
  charged when it ends, as before.
- **If the energy cannot be read** (no plug, the plug is unreachable, or its
  counter was reset during the print), the print is charged without it and the
  description says so: `(filament 1.20 + wear 0.25; energy not available)`.
- **Energy is charged only from one plug.** The energy is the difference
  between the plug's counter at the start and at the end of the print. If the
  plug read at the start does not answer at the end and another plug of the
  printer does, the two readings are from different counters, and the print
  is charged with `energy not available`.
- **A print that ended while its printer was offline pays no energy.** That
  is any print whose end Bambuddy did not see: the printer was disconnected,
  or Bambuddy was not running. Its end is noticed at the reconnect, and the
  plug's reading then also holds everything since the print really ended (idle
  draw, other prints). Such a print is charged when it is noticed, with
  `energy not available`. Its wear is charged when Bambuddy can still work out
  when the print ended, for that duration, and is nothing otherwise.
- **The policy that counts is the one in force when the print ends.** A charge
  waiting for its energy is not changed by a setting changed meanwhile.
- **A restart does not lose the charge.** A charge that was waiting for its
  energy when Bambuddy stopped is written at the next start, with the energy
  if it had been saved, without it otherwise. If that fails, the
  **Billing charge failed** alert is sent, once, and the charge is tried
  again at each following start. Whatever removes the print's log entry or its archive
  while its charge is waiting writes the charge first: clearing the print log
  or deleting an entry, deleting or purging the archive, deleting its printer
  with its archives, deleting its user with their items, and merging archives.
  A waiting charge whose archive is gone all the same cannot be written: the
  **Billing charge failed** alert is sent and the print's reservation is
  released.
- **Budgets cover the filament only.** The budget check and the reservation
  taken for a queued print use the filament estimate, without energy or wear.
  With those switched on, a center can end up slightly over its budget.
- **The Print Log's cost stays the filament.** With Energy Cost or Wear Cost
  on, the [Print Log](print-log.md) still shows a print's filament cost as its
  cost, and its energy and wear in their own fields. The charge is the three
  together: the filament, priced at the spools the print prints with, plus its
  energy and wear. So a completed print with no measured cost is logged at the
  filament part of its charge, not at the whole charge.
- The policy applies to every cost center.

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

### API keys { #api-keys }

With billing on, a job queued through an [API key](api-keys.md) needs a
`cost_center_id` like any other. A key with **Manage Queue** finds the ones
it may use at `GET /api/v1/finance/cost-centers/mine`. That endpoint returns
the cost centers of the key's owner, and a key can charge only those, or any
center if its owner is an administrator. A key created before keys had owners
has none, and needs to be recreated to queue with billing on. Balances, the
ledger and cost-center administration stay closed to API keys.

---

## :material-alert-octagon: Printer kill switch { #printer-kill-switch }

**Settings → Workflow → Billing → Printer kill switch.** Off by default, and
separate from billing itself.

An install that bills its users can be bypassed by sending a job straight to the
printer from Bambu Studio or Handy — that print is not queued, not costed and
not billed. With the kill switch on, Bambuddy stops such a print as soon as it
sees it, raises an alert in the interface, and sends the configured print-stopped
notification. Its archive is marked cancelled, with the reason **Stopped by the
printer kill switch**, so it does not count as a failed print.

!!! danger "This stops prints, and a stopped print is not recoverable"
    Enable it only where prints genuinely must not start outside Bambuddy. It
    acts on any print it can show did not come from Bambuddy, including one
    started deliberately by an administrator at the machine. The printer's
    built-in calibration (started on the printer's screen or from the slicer's
    device page) is left alone; a calibration print sent from Bambu Studio, such as
    the pressure-advance line, is a print like any other and is stopped. On an
    A1, A1 Mini, P1 or A2L the built-in calibration reports itself under a
    plain file name that any file could be given, so it is stopped too: switch
    the kill switch off while calibrating those printers.

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
