# Building the flow, v2 — the order of work

*What gets built first, what next, and what you can do end to end after each step. Engineering's proposal, 10 September, for the CTO to agree before the first PR. Builds [The order → delivery flow, v2](order-delivery-flow-v2.md).*

## The rules every step follows

- **Small, focused PRs.** One thing each. Nothing lands that cannot be shown working over real HTTP against a running app the same day.
- **Testable end to end after every step.** The whole flow — place, accept, ready, crew, collect, deliver, pay — keeps running on `develop` after every merge. A step replaces a piece of the old flow with a piece of the new one; it never leaves a gap.
- **Delete before add.** The first six PRs remove what the flow no longer needs, so everything after is built on a smaller surface.
- **Money last within each step, and never blind.** Any PR that posts to the ledger or decides who may act gets mutation tests: remove the guard, watch the pin fail. Anything that talks to Paystack is verified against Paystack's test keys, confirmed each time.
- **Merged in order.** Each step is a short stack of PRs merged one after another; `develop` stays green and bootable at every point.

## The sequence

```mermaid
flowchart LR
    S0[0 · The floor<br/>the deletions] --> S1[1 · Settings and taxes]
    S1 --> S2[2 · Telling people<br/>in-app · email · SMS]
    S2 --> S3[3 · Money rails]
    S3 --> S4[4 · The farm<br/>check · READY · crew · cancellation]
    S4 --> S5[5 · The gate<br/>HANDOFF · IN TRANSIT · collection failed]
    S5 --> S6[6 · The door<br/>code · photo · delivery failed · retry · cash]
    S6 --> S7[7 · After delivery<br/>disputes · ratings · docs]
```

| Step | PRs | What it builds | What you can do end to end after it | How it is verified |
| --- | --- | --- | --- | --- |
| **0 · The floor** | A drop the order-history table · B "refund due" gets a single writer · C1 orders carry their own crew and pickup time · C2 the duplicate delivery record and the farm's dispatch button go · C3 a trip's label and times worked out from its orders, with Cancelled for a trip called off before collection · D drop the packing step and the day-before reminder | Nothing new. Four duplicates removed; two facts moved to the one place that owns them | The old flow, unchanged in behaviour: place, accept, ready, crew, pickup, confirm, cancel from every state, refund. The farm can no longer dispatch or mark "processing" | A: the parked branch's mutation check. B: mutations on every writer of the flag. C1 and C2: mutations on the four crew guards and on the pickup write, plus a drift check proving the two copies agreed before the record went. A live run after every PR, over every actor and state |
| **1 · Settings and taxes** | E the settings catalogue — every number Ops sets, grouped, with defaults, bounds and an audit trail · F the tax catalogue — name, description, percentage, on or off; shipped empty | The place every later step reads its numbers from. Produce tax switched off | Admin changes the delivery fee in the grouped settings screen and the next order uses it. Admin adds a tax and the next order carries it; removes it and the next order does not | Live: order totals before and after; a receipt for an order placed under the old levies still reads |
| **2 · Telling people** | X the notification frame — in-app as a real channel beside email and SMS, the feed and its unread count, the category set the flow actually needs, and staff as recipients · Y the twelve messages the flow names and the six that exist, all moved onto it | The one place that decides who hears what, on which channel, and what a person may switch off | A buyer opens a feed and sees every message about their order; switching off SMS for one kind of message stops the text and keeps the record; a field agent is told an order is waiting for a check | Live: every message the flow names, to the right person, on the right channel, with the feed still holding it after both other channels are off |
| **3 · Money rails** | G refunds of a stated amount, with the credit-note line · H the payout run reads the ledger: pays what is owed, skips an order with an open dispute or one still inside its dispute window · I refund a cash buyer by mobile money | The three money moves every later step needs, each usable on its own | Admin refunds part of an order and the ledger and credit note agree. A disputed order is skipped by the payout run; it is paid once resolved. A cash order's refund reaches the buyer's phone | Mutations on all three. G and I against Paystack test keys |
| **4 · The farm** | J the availability check — the agent's form, three photos a line, the derived outcome, Short and Not-available handled, the overdue list · K READY and the crew gate — no check, no crew; the no-field-agent exception; the acceptance and READY deadlines; the crew-assigned text · L the cancellation windows — free until READY plus grace; the penalty from escrow to the farmer; strikes for cash buyers, cash switched off after the limit; farm strikes; Admin/Ops cancel with a fault | The farm side of the decided flow. **Ops can start operating on it:** field agents check, farmers mark READY, Admin/Ops assign crews from the order detail | Accept → check (all four outcomes) → READY → crew → pickup → confirm, online and cash. Cancel in each window and see the right refund and penalty. A buyer with too many strikes is refused cash on delivery | Mutations on L. Live run after L with every window and every check outcome, flows listed and reviewed first |
| **5 · The gate** | M HANDOFF by the farmer, IN TRANSIT by the delivery agent, the alert when one follows the other too slowly · N COLLECTION FAILED — the reasons, nothing taken, the check made void, retry through READY, free cancel while it lasts · O the farmer paid in full after HANDOFF whatever follows, funded by the penalty and by us | Custody as two people's words; the first failure state; the farmer's money settled | The farmer taps HANDOFF, the agent taps IN TRANSIT, the buyer sees "on its way". The crew records a failed collection and the order comes back through READY or is cancelled. An order cancelled after HANDOFF still pays the farmer | Mutations on O. Live run after O |
| **6 · The door** | P the handoff code — sent at placement, verified at the door with no signal · Q the delivery photo, required · R DELIVERY FAILED, the return to the farm, the alerts · S the second attempt — the buyer asks and pays the fee again, or the order is cancelled with a strike · T "cash received" text and the agent's daily cash remittance | The door as decided; every state now has an exit | The full flow, online and cash, with every failure branch: wrong code, no photo, not paid in full, nobody home, a second attempt, a cash day remitted | Mutations on S. **The big live run:** the whole flow end to end, both payment methods, every branch |
| **7 · After delivery** | U disputes — produce or delivery, the payout held, Admin/Ops name the amount and who bears it, the refund follows, the five-day clock · V two ratings · W the API document rewritten; the old flow pages retired | The flow complete | A buyer disputes, Admin/Ops resolve against the farm or against us, the buyer is refunded, the farmer is paid what is left at the next run. Two ratings on one order | Mutations on U. Live run of a dispute through to a mobile-money refund |

Twenty-seven PRs. Steps 0 to 3 are deletions, plumbing and the message frame; step 4 is the first thing Operations can use; step 6 is the first time the whole decided flow exists.

## What changed since 10 September

This page is the living order of work, so it is corrected as the CTO decides and as the work lands. Three corrections so far, all to step 0.

- **"Refund due" is stored with a single writer, not derived.** Decided by the CTO during B, 12 September. The flag now has exactly one writer called from every path that can change the answer, and a report-only check that names any order where the flag and the refund records disagree. The reason to store it stands on its own: an admin list that filters on it should not re-run a derivation per row, and one writer is easier to prove correct than a rule copied into every reader. Nothing is derived twice either way.
- **C became three PRs, and the run's status is still stored.** C1 put the crew and pickup time on the order and proved the copies agreed; C2 deleted the duplicate record and the farm's dispatch button. Deriving the run's status turned out not to be needed for either, so it is C3 and still to do. It is worth keeping inside step 0 rather than deferring: it is a deletion, the surface shrinks before everything after it, and the stranded-run problem Operations sees today comes from that stored status being the authority.
- **A trip called off before the van went reads Cancelled.** Decided by the CTO on 13 September, with two smaller answers alongside it: a finished trip that gets a new order reads In progress, and a trip's start and finish times are worked out from its orders rather than removed. All three are questions 44 to 46 on the [working-answers page](order-delivery-flow-v2-working-answers.md). They settle what C3 builds; they do not move it in the order.

## Why this order

- **Deletions first** because every later PR is smaller and safer on the reduced surface, and because the four deletions were already audited on 2 September.
- **Settings before anything that reads a number**, so no step ships with a number hardcoded and moved later.
- **Telling people before anything that has something to tell them.** The decided flow names eighteen messages and **five** of them exist (eleven messages fire today, but five of those eleven — the two admin-cancellation notices and all three dispute notices — are not on the decided list at all, and step 2 has to decide whether each is kept, folded in, or dropped). Without a frame, each later step invents its own message, its own channel and its own opt-out, and the platform ends up with eighteen local decisions instead of one design. Money is the first step with something to say — a refund, cash received — so the frame goes before it. **In-app is new**: today a message exists only as a sent email or text, so if a person switches both off nothing survives and "what did we tell this buyer?" has no answer.
- **Money rails before the transitions that post** — but each rail is usable on its own the day it lands, through an admin action, so it is tested end to end and not "callable but unused".
- **The farm before the door** because it is where the old flow is weakest (no check, no gate on the crew) and where Ops can start operating soonest. Between steps 3 and 4 the old pickup button still moves an order to IN TRANSIT, so the flow never has a gap.
- **Disputes last** because they read everything before them: the delivery photo, the check photos, the farmer's payable, the refund rails.

## What has to be true before the first PR

- The [decided flow](order-delivery-flow-v2.md) stays as decided. Any change there reorders this page, not the other way round.
- The engineering spec behind this page has had its review round. It is the local design document engineering builds from; this page is the agreement on the order.
- A plan per step, listing each PR's tasks, files, tests, mutations and live-verify flows, written and reviewed before that step starts — step 0's is drafted.
