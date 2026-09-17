# Who owns a farm, and what suspending someone should mean

**Answered by the CTO on 17 September 2026**, the day it was found — while building the notification
work, when a test deliberately suspended a farm owner and the farm carried on trading. The answers are
below; **none of it is built yet** except the first piece, which went out on its own because it was a
live gap.

## The short version

A farm's owner can be suspended, and **nothing happens to the farm**. That may be right — but today it
is not a decision, it is an absence. Behind it sits a gap that is harder to live with: **ownership
cannot be handed to anyone else**, so a suspended owner leaves a farm that can never have another one.

## What is true today, read from the code

| Fact | What it means |
|---|---|
| A farm's **Owner** role is minted once, when the farm is created, for the founder | it is the only way anyone becomes owner |
| The Owner role **cannot be invited, edited or reassigned** | there is **no way to transfer ownership**, to anyone, ever |
| Suspending a person suspends **their account** and signs them out | it does not touch their farm |
| Suspending a **farm** is a separate admin act, on its own screen | nothing links the two |
| A farm can be suspended only from active, by an admin, with a reason | it is deliberate and reversible |
| Any member holding the fulfil permission can accept and dispatch orders | **a farm can trade without its owner** |
| A suspended farm's listings leave the marketplace, and the basket refuses to add one | browsing and adding are already closed |
| **Placing an order never checked the farm at all** — it loaded the farm by id, then checked only each listing's own status | a buyer with a direct link, or an item added before the suspension, could order from a suspended farm. **Closed — see "What was built first"** |

## Why it matters, in two different farms

**A one-person farm.** The owner is the farm. Suspend them and orders keep arriving at a business
where nobody can accept them, nobody can be reached, and nobody is watching. Produce is promised that
will not be picked. This is the common case.

**A farm with a team.** The owner is suspended — they left, or their account was compromised, or they
are under investigation. Five people still work there, still hold orders, still have produce in the
yard. Suspending the whole farm punishes them and every buyer with an order in flight. Here the right
answer is almost certainly to replace the owner, not to close the farm.

So "should suspending an owner suspend the farm?" has no single answer. The harm is not that the owner
is suspended; **the harm is that the farm has nobody accountable and nobody notices.**

## What was built first

**Suspension now stops new orders, at the one place both routes pass through.** Placing an order
refuses a farm that is not active, whether the buyer came from the basket or straight to the
placement endpoint. It is the whole of question 4's answer and the only part of this page that
exists in code today.

## The questions, answered

| # | Question | **Decided 17 September (CTO)** |
|---|---|---|
| 1 | When an admin suspends someone who owns a farm, what are they told, and what do they choose? | **Tell them, and make them choose.** The screen says this person owns that farm, and offers two acts: the person alone, or the person and the farm. An admin must never end a business without meaning to |
| 2 | Should a farm with no active owner stop taking new orders? | **Yes.** It is the real harm — an order nobody is accountable for — and it is narrower than suspending the farm. It clears itself the moment the farm has an owner again, however that happens |
| 3 | Should ownership be transferable, and by whom? | **Yes: by the owner, and by an admin.** The owner for succession, the admin for when the owner is gone, suspended, or unreachable. Without it every owner suspension is permanent in effect |
| 4 | What should suspending a farm actually stop? | **New orders.** Refused where an order is placed, so both the basket and a direct link are covered by one rule. Browsing and adding were already closed |
| 5 | What happens to orders already in flight? | **They run to delivery.** Produce is promised and money may already be held in escrow; cancelling them would strand the buyer. A suspended farm is stopped from taking more, not from finishing what it owes |
| 6 | Is suspending a person the same as suspending them as an owner? | **No.** A compromised account is not a dishonest farm. That is precisely why question 1 is a choice an admin makes rather than a rule the system applies |

**The thread through all six: do not cascade — choke at the right point.** One refusal where an order is
placed, one prompt where a person is suspended, and ownership that can move. A farm and the person who
owns it are two things, and the system should stop asking one question when it means the other.

## What is still to build

1. **The admin's choice when suspending an org owner** (question 1) — today they are told nothing.
2. **A farm with no active owner refuses new orders** (question 2) — the placement guard exists; this
   is a second reason to refuse, and it needs the wording that says which is which.
3. **Ownership transfer** (question 3) — the largest of the three, and the one a real business needs
   first when a founder leaves.

## How it was found

Building the notification work, a test suspended a farm owner's membership to see what the farm's team
would be told. The team heard about the order; nobody was emailed; the farm kept trading. Every part of
that is today's intended behaviour — which is how the question surfaced. It is not a notification
problem and no notification work will fix it.
