# Who owns a farm, and what suspending someone should mean

**Open question for the team.** Nothing here is decided, and nothing here is built. It was found on
17 September 2026 while building the notification work, when a test deliberately suspended a farm
owner and the farm carried on trading.

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
| A suspended farm's listings leave the marketplace — but the basket checks only the **listing's** status | a buyer holding a link can still order from a suspended farm |

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

## The questions, for Operations and the CTO

| # | Question | Why it needs an answer |
|---|---|---|
| 1 | **When an admin suspends someone who owns a farm, what should they be told, and what should they choose?** Today they are told nothing and choose nothing. | An admin can end a business by accident, in one click, and find out later |
| 2 | **Should a farm with no active owner stop taking new orders?** | This is narrower than suspending the farm, it addresses the real harm, and it heals itself the moment ownership is restored |
| 3 | **Should ownership be transferable — and by whom?** The owner themselves, or an admin, or both? | Without it, an owner suspension is permanent in effect. It is also what a real business needs when a founder leaves |
| 4 | **What should suspending a farm actually stop?** Today it hides the listings but does not block an order placed from a direct link. | Deciding question 1 on top of a switch that does not stop trade would give false comfort |
| 5 | **What happens to orders already in flight** when a farm is suspended — accepted, paid, produce packed? | Buyer money sits in escrow. Suspension must not strand it |
| 6 | **Is suspending a person the same as suspending them as an owner?** A compromised account is not a dishonest farm. | The reason for the suspension may decide the answer to every question above |

## A shape worth considering

Not a proposal, a starting point: **do not cascade automatically.**

1. When an admin suspends an org owner, **say so and make them decide** — the farm as well, or the
   person only.
2. **A farm with no active owner refuses new orders** — the honest statement of "nobody is home",
   whether the owner was suspended, left, or was never replaced.
3. **Build ownership transfer**, so that the answer to a departed owner is a new one rather than a
   dead farm.
4. **Make a farm's suspension actually stop trade** before anything else leans on it.

## How it was found

Building the notification work, a test suspended a farm owner's membership to see what the farm's team
would be told. The team heard about the order; nobody was emailed; the farm kept trading. Every part of
that is today's intended behaviour — which is how the question surfaced. It is not a notification
problem and no notification work will fix it.
