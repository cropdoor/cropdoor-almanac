# Working answers — closing the gaps before we build

*The working document for finalising the flow. Questions come from the [gap analysis](order-delivery-flow-v2-gap-analysis.md); the plain-language versions are on the [questions page](order-delivery-flow-v2-questions.md). Answers are recorded here as they land, and the flow is redrawn from them.*

## How to use this page

Every question is tagged by **who owns the answer**:

| Tag | Meaning | Who answers |
| --- | --- | --- |
| **FACT** | How the operation actually works today. Not a debate. | Operations |
| **POLICY** | A business rule we have to choose. | Leadership, with Operations |
| **DESIGN** | How to build it, once the facts and policy are known. | Engineering proposes; the team confirms |

And by **what it hangs on** — the three shape questions from the analysis. Answer those first; many rows below change or vanish depending on them.

| Shape question | If the answer is… | Then… |
| --- | --- | --- |
| **S1** — is the inspection of the packed crate, at the gate, once per farm? | **Answered 9 Sep: no.** The order visit is an *availability check before a crew is committed* — the produce may still be in the ground. It is distinct from the farm-verification visit at registration, and distinct from the handoff, which counts what went in the van. Defined in [The order availability check](order-availability-check.md). | Steps 3/4 and 6 stay separate, deliberately. The check is the promise, the handoff is the count — recorded by the farmer's own HANDOFF tap and the crew's IN TRANSIT (S3). **Ops' answers (9 Sep):** the check follows acceptance; the agent has 8 working hours; one visit per farm per day; the delivery agent is the fallback where a zone has no field agent; when the farmer marks READY, **Admin/Ops assign the crew — and only if the check is on the order** (question 37) |
| **S2** — does a dispute hold the farmer's payout, and who owns the outcome? | **Answered 9 Sep: yes, and Admin/Ops own it.** A buyer's dispute holds that order's payout. Admin/Ops receive the dispute and investigate from there. | Question 21 is answered. Still open under S2: what Admin/Ops may *conclude* with (22, 23); who is responsible when our own agent certified the quality (6); the rejection paths at the gate and the door (8, 14); the ratings (29, 30); whether anyone but the buyer may dispute (35) |
| **S3** — when the farmer hands the produce to the crew, is that the one moment that matters — after which the buyer cannot cancel and the produce is ours to deliver — with "in transit" just what the buyer *sees* afterwards, rather than a second button the delivery agent must remember to tap? | **Answered 10 Sep: no — keep both, because they are two people's words.** The farmer marks HANDOFF: "I gave it to the crew." The delivery agent marks IN TRANSIT: "we have it and it is moving" — which tells the buyer the goods are coming and tells Admin/Ops that we actually hold the goods. | Two statuses stay, each written by a different person. Their pair is the two-party record at the gate — which makes the farmer code (question 10) unnecessary; dropped. "Handed off but not in transit" becomes a *real* signal for Admin/Ops, not a forgotten tap (question 27) |

**S3 in plain words, and the answer.** The team's flow has two taps at the farm gate, moments apart: the farmer hands over (HANDOFF, step 6), then the delivery agent marks IN TRANSIT (step 7). Engineering asked whether the second tap could be dropped, since the platform knows the van's trip and could show "on its way" by itself. Operations' answer: keep both, because they mean different things from different people. The farmer's HANDOFF is the farmer saying the produce left their hands. The delivery agent's IN TRANSIT is the crew saying *we have it* — for the buyer, the goods are coming; for Admin/Ops, the goods are confirmed in our hands. A handoff with no IN TRANSIT after it is therefore worth a look, not a stale screen.

**All three shape questions are now answered.** The flow as it stands is drawn on [The flow, v2](order-delivery-flow-v2.md), with what is still open listed under each step.

**Where engineering already has a view, it is pre-filled in the last column** — so the meeting reacts to a proposal rather than starting from blank. A pre-filled view is an opinion, not a decision.

---

## Start here — three facts that decide the shape

These are FACT questions. Ops can answer them in a sentence each, and the answers determine which of the remaining thirty apply.

| # | Question | Tag | Hangs on | What the world already forces · engineering's view | **Answer** | Flow change |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | When the field agent visits, are they looking at produce already picked and packed, or at what is still growing? | FACT | S1 | If standing crop: the inspection certifies nothing about the crate that goes in the van, and a second inspection at handoff is needed anyway. If the packed crate: the inspection *is* the handoff moment. | **Either.** The visit checks that the produce *exists, in the quantity and quality ordered*, and records its state (packed / harvested / ready to harvest / not ready). Readiness itself is the farmer's declaration — marking READY — not a date the agent writes down. It is not the crate inspection. | Step 3 is the availability check as defined on its own page; the handoff (step 6) counts what went in the van, with the farmer's code. The check is made within 8 working hours of acceptance |
| 2 | Is the visit once per order, or once per farm on a collection day covering everything ready? How many farms can one agent inspect in a day? | FACT | S1 | Field agents are zone-based. Per-order visits do not scale past a handful of farms. **Proposal:** one visit per farm per collection day, submitting one check per accepted order at that farm. | **Answered 9 Sep (Ops): one visit per farm per day.** One check submitted per accepted order waiting at that farm | Step 3 is scheduled per farm per day, not per order. Admin/Ops see "accepted, awaiting check" by zone |
| 9 | Do farmers ever bring produce to a shed or collection point rather than the van coming to the farm? If yes, who is present at that handoff? | FACT | S1, S3 | The model already has aggregation points — "the shed or warehouse within a zone where farmers bring produce a van cannot collect at the farm". If this happens, the farm-gate sign-off has a different signer, possibly no farmer. | **Answered 9 Sep: no — collection points are retired for now.** Every collection is at the farm gate, with the farmer or a farm member present. Deliberately simple; to be revisited once daily operations have taught us. | One geometry only: the van comes to the farm, and the farm-gate sign-off always has the farmer (or a farm member) as the signer. A farm the van cannot reach cannot be served for now, so van access — at verification and at the check — matters more |

**Then decide:**

| # | Question | Tag | Hangs on | Engineering's view | **Answer** | Flow change |
| --- | --- | --- | --- | --- | --- | --- |
| 3 | If the agent is at the farm anyway to inspect, why not make that visit the collection? What is gained by two trips? | POLICY | S1 (after 1, 2, 9) | **Collapse them** if 1 = packed crate. One farm-gate moment: farmer presents, agent inspects and receives, both sign. If 1 = standing crop, keep the early visit *but* as a per-farm-per-day act, not per order — and accept that the crate is inspected again at handoff. | **Two trips are deliberate.** What is gained: a crew is never sent for produce nobody has seen. The check follows acceptance; collection follows READY and Admin/Ops' assignment. Where a zone has no field agent, the delivery agent makes both in one visit (question 4). | Keep step 3 and step 6 as separate acts, even when one person does both. Accept a light count at handoff |

---

## The farm visit and the farm gate

| # | Question | Tag | Hangs on | What the world forces · engineering's view | **Answer** | Flow change |
| --- | --- | --- | --- | --- | --- | --- |
| 4 | If there is no field agent in a zone this week, does that zone stop selling, or does someone else sign off — and who? | POLICY | S1 | Every gate needs a named fallback or it is a stall. Candidates: the delivery agent signs at collection; Ops overrides with a reason; the zone pauses. | **Answered 9 Sep (Ops): the delivery agent makes the check on collection day**, at the gate before collecting — the one case where Admin/Ops send a crew without a check already on the order. The zone never pauses for want of a field agent | The check form is available to the delivery agent as well as the field agent. Two acts on one visit: the check first, then the handoff count |
| 5 | If the agent finds the farm short, that affects every other buyer on the same listing. Who tells them, and what are they offered? | POLICY | — | Stock is per listing, not per order. A short finding is a listing event: reduce the listing, notify every open order on it, offer each buyer cancel-free or wait. **Proposed on the check page:** a Short / Not-available finding writes the corrected quantity to the listing in the same step, and every other open order on it is re-checked against the new number. | — *(proposal awaiting Ops)* | — |
| 6 | When our agent has certified the quality and the buyer still complains about quality, who is responsible — the farmer or us? | POLICY | S2 | The flow puts our signature in the chain. Engineering's view: a certified-quality dispute is **ours to resolve with the buyer**, and separately ours to take up with the farmer — not passed straight through as a farmer clawback. | — | — |
| 7 | Do farmers have smartphones and data at the gate? If not, who taps READY and "handed over" for them? | FACT | S1, S3 | Only an SMS rail reaches every farmer. If an agent taps for the farmer, every "two-way" sign-off is one person's word unless the farmer holds something the agent doesn't — see 10. | **Answered 10 Sep: a farmer must have a smartphone.** There is no flow yet for anyone tapping on a farmer's behalf | READY and HANDOFF are the farmer's own taps, on their own phone. A farm without a smartphone is not on the platform for now. The farmer code (10) is not needed |
| 8 | What happens today when the van arrives and the produce is not there, is short, or the farmer is absent? Who decides on the spot? | FACT | S2 | This is the missing **rejection path at the gate**. Whatever happens today becomes a named outcome with an owner. | — | — |
| 10 | Should the farmer have their own code — read out to the crew — so a disagreement about what was handed over has two records, not one? | POLICY | S3 | **Yes.** The buyer-door code solves exactly this problem at the other end. A farmer code sent by SMS works on any phone and turns the agent's tap into a two-party record. **After S3 and 7 (10 Sep):** the farmer taps HANDOFF on their own phone — that is the second record. | **Dropped 10 Sep.** Not needed | — |
| **34** | **What is the cancellation rule while the order waits for the field agent — between "accepted" and "READY"?** | POLICY | S1 | *Not asked before.* The flow says free until accepted, penalty at READY; the wait in between — the longest window — is unspecified. **Proposal (9 Sep):** the buyer may cancel **free until the farmer marks READY** — where the team's flow puts the penalty. Acceptance alone does not open it: the farmer has committed nothing yet and the check has not happened. Because the check comes first, the buyer hears whether the produce is there before their free window closes. | **Answered 10 Sep: free until READY.** | The flow's "free until step 2" becomes "free until READY (step 4)". Acceptance opens no penalty; the penalty window runs from READY to HANDOFF |

---

## The door

| # | Question | Tag | Hangs on | What the world forces · engineering's view | **Answer** | Flow change |
| --- | --- | --- | --- | --- | --- | --- |
| 11 | In what order do things happen at the door: cash, code, hand over, photo? What if the buyer gives the code and then will not pay? | POLICY | — | Engineering's proposed order: **code → cash (if POD) → hand over → photo → confirm.** The code proves the right buyer; cash before the crate leaves the agent's hands; the photo records what was handed over. Crucially, **entering the code must not itself mark the order delivered** — only the final confirm does, so "code then refuses to pay" leaves the order undelivered. | — | — |
| 12 | What does the agent do when the buyer is present but cannot show the code — dead phone, text never arrived? And if a named representative receives, whose code do they give? | POLICY | — | The address already lets a buyer name a contact person. Options: resend the code to the registered phone on the spot; the agent calls the registered number and the buyer reads it; Ops override with a reason, logged. Refusing delivery to the right person at the right door is the worst outcome. | — | — |
| 13 | How often does a POD buyer come up short or want to pay part? What do agents do today? | FACT | — | Payment at the door is currently all-or-nothing. If partials are common, the flow needs "partial collected, balance owed" or "order reduced at the door" — both are money design. | — | — |
| 14 | Nobody home, or the buyer refuses the crate: retry (who pays), redirect, return to farm, write off? Who decides — agent, supervisor, office? | POLICY | S2 | This is the missing **rejection path at the door**. It needs a named outcome, an owner, and an answer to "is the farmer paid" — they did everything right. | — | — |
| 15 | One buyer, three farms, same day: three codes at one gate? | POLICY | — | Engineering's view: **one code per buyer per delivery day**, not per order. The code verifies the person, not the crate. | — | — |

---

## Money

| # | Question | Tag | Hangs on | What the world forces · engineering's view | **Answer** | Flow change |
| --- | --- | --- | --- | --- | --- | --- |
| 16 | Is the penalty to compensate the farmer, deter buyers, or cover our van cost? The answer decides who receives it. | POLICY | — | Engineering cannot pick the recipient. It can say: if the farmer receives it, that is a payable on an undelivered order (a new kind of payout line) *and* the farmer keeps the produce. If the platform receives it, the farmer who harvested bears the loss. | — | — |
| 17 | How do we collect a penalty from a cash-on-delivery buyer who has never paid us anything? | POLICY | — | Honestly: **we can't, today.** Options: a deposit on POD orders (see 36); block further ordering until paid; suspension. Without one of these, POD buyers are penalty-immune. | — | — |
| 18 | If a farmer accepts and then cancels or fails to hand over, does the farmer pay anything? Does the buyer get anything for the wait? | POLICY | — | The flow's penalties are one-sided. Engineering's view: at minimum the buyer is refunded in full *including the delivery fee*, and a farmer failure after READY is recorded against the farm. | — | — |
| 19 | Van breaks down, or we miss the date: farmer still paid? Buyer refunded the fee? Buyer's penalty waived if they cancel because we were late? | POLICY | S2 | Engineering's view: platform failure → farmer paid (they delivered to us), buyer refunded fee, penalty waived. It is our failure. | — | — |
| 20 | When an agent collects cash, when and how does it reach us? How do we know at month-end which cash is still in agents' pockets? | FACT | — | Today cash is recorded as ours the moment the agent confirms. Whatever the real remittance is (daily to office, mobile money, bank) becomes a step with a record, so "collected" and "banked" are different facts. | — | — |
| 21 | Does a complaint after delivery pause the farmer's payment? For how long? If already paid, do we take it back from their next sale? | POLICY | S2 | Engineering's view: **yes, an open dispute holds that order's payout**, and the payout clearance window should be at least the dispute window so an *undisputed* order clears exactly as the window closes. Clawback from future sales should be the exception, not the mechanism. | **Answered 9 Sep (S2): yes.** A buyer's dispute holds that order's payout; Admin/Ops receive it and investigate. Still open: for how long, and what happens if the farmer was already paid | The payout run skips any order with an open dispute; Admin/Ops' dispute queue is where it is released or resolved |
| 22 | "Partial refund", "replacement", "goodwill credit" — what actually happens, who moves what money, who pays for a replacement delivery? | POLICY | S2 | None of these outcomes moves money today. Each needs a rail: partial refund = a refund of a stated amount; replacement = a new order at whose cost?; goodwill credit = a buyer balance we do not have. Pick which outcomes we actually offer. | — | — |
| 23 | A cash buyer wins a complaint — how do we pay them back? | POLICY | — | Needs a pay-to-buyer rail (mobile money) and an answer to "has the agent's cash even been banked". | — | — |
| 24 | Does a cancellation fee attract VAT or levies? What document does the buyer receive for it? | FACT | — | An accountant's question. The answer decides whether the penalty is one ledger line or two, and whether a credit note can carry it. | — | — |
| **36** | **Should cash-on-delivery orders require a deposit?** | POLICY | — | *Not asked before — engineering's proposal.* It is the only mechanism that makes a POD penalty enforceable, and it partly answers 17, 23 and the phantom-POD-buyer case. It also changes the buyer's experience. | — | — |

---

## Time

| # | Question | Tag | Hangs on | What the world forces · engineering's view | **Answer** | Flow change |
| --- | --- | --- | --- | --- | --- | --- |
| 25 | How long may an order wait for the farmer to accept before we cancel it for them? | POLICY | — | Every state needs a maximum or it is a stall. Engineering's view: a number of hours, then auto-cancel with the buyer refunded in full. | — | — |
| 26 | Once READY, how quickly must the van collect? Is produce saleable after that? | POLICY | S1 | Sets the collection SLA — and whether a buyer whose order sits READY waiting for *our* van is still in the penalty window. Engineering's view: they should not be. | — | — |
| 27 | How long may an order sit "handed off" or "in transit" before the office is alerted? Who is alerted? | POLICY | S3 | This is the ops list from the previous round — "ready and not collected", now also "collected and not delivered". Needs an owner. **After S3 (10 Sep):** "handed off, not yet in transit" joins the list — the farmer says it left, the crew has not said they have it. | — | — |
| 28 | A buyer taps cancel a moment after the farmer accepted and is told a penalty applies. Grace period, or hard line? | POLICY | — | Engineering's view: a short grace period (minutes) costs nothing and removes a complaint generator. **After 34:** the moment in question is READY, not acceptance. | — | — |

---

## After delivery

| # | Question | Tag | Hangs on | What the world forces · engineering's view | **Answer** | Flow change |
| --- | --- | --- | --- | --- | --- | --- |
| 29 | The buyer rates "the farm" — but the van, the agent and the timing were ours. Split the rating? Should a farmer answer a complaint about a late van? | POLICY | S2 | Engineering's view: **two ratings** — produce (farm) and delivery (platform) — and disputes routed by type: quality/quantity to the farm, late/damaged/agent to us. | — | — |
| 30 | Can a buyer both rate and dispute the same order? Should an upheld complaint change the rating? | POLICY | — | Engineering's view: both allowed; an upheld quality dispute suppresses the produce rating from the farm's average. | — | — |
| **35** | **Should anyone other than the buyer be able to raise a dispute — a farmer short-paid or refused at the gate, an agent robbed?** | POLICY | S2 | *Not asked before.* Disputes are buyer-only in the flow. The rejection paths (8, 14) will generate farmer-side grievances with no channel. | — | — |

---

## People and messages

| # | Question | Tag | Hangs on | What the world forces · engineering's view | **Answer** | Flow change |
| --- | --- | --- | --- | --- | --- | --- |
| 31 | Will the same person ever be the field agent, the delivery agent and the confirmer? Are we comfortable with one person holding all three? | FACT → POLICY | S1 | If S1 collapses the steps, inspector and collector are *the same role by design* — then the question becomes whether the confirmer at the door must be a different person from the collector at the gate. | — | — |
| 32 | Are delivery agents employees or gig workers? | FACT | — | Cash custody and the override rule both rest on this. | — | — |
| **37** | **Who assigns the crew, and on what signal?** | FACT | — | *The flow said "is assigned" — no actor named.* | **Answered 9 Sep: Admin/Ops.** When the farmer marks READY, Admin/Ops assign a delivery crew from their dashboard — and only when the order's details show the field agent's check. No check on the order, no crew. | Step 5 gets its actor. The order detail for Admin/Ops shows the check (who, when, outcome, photos); assignment without it is refused — design, to confirm |
| 33 | Which moments produce a text, to whom: agent coming; READY; handed over; van on the way; delivered; cash received; complaint raised? | POLICY | — | Today farmers are told about new orders and cancellations only. Engineering's view: farmer gets *agent coming* and *handed over* (with the count); buyer gets *on the way*, *delivered*, and *cash received (GHS X)* as a receipt. | — | — |

---

## Coverage — every gap and stall, and the question that closes it

So we can see when we are done. A gap with no question was a hole in the analysis itself; four were found and added above (34, 35, 36, 37).

| Gap / stall from the analysis | Closed by |
| --- | --- |
| Buyer refuses at the door / absent | 12, 14 |
| POD buyer short or partial | 13, 36 |
| Cancellation between accepted and READY undefined | **34** |
| Penalties one-sided | 18 |
| Farmer without a smartphone | 7 (answered: a farmer must have one), 10 (dropped) |
| Door sequence unordered | 11 |
| Buyer present, no code | 12 |
| One buyer, several farms, one door | 15 |
| One farm, several orders, one van | 2, 3 |
| No time budget after READY | 26, 27 |
| Van breaks down mid-run | 19, 27 |
| Buyer rates the farm for our delivery | 29 |
| Field-agent coverage | 2, 4 |
| Who assigns the crew (passive in the flow) | **37** |
| Availability is per listing | 5 |
| Dispute on a POD order | 23 |
| Cancel/accept race | 28 |
| Order stuck awaiting acceptance | 25 |
| Stuck awaiting the agent | 4, 34 |
| READY, no crew | 26, 37 |
| Van at the gate, no handoff | 8 |
| HANDOFF never IN_TRANSIT | S3 (kept as two taps — now a real signal), 27 |
| IN_TRANSIT never DELIVERED | 14, 27 |
| Dispute open, farm silent | 21 |
| POD cash never remitted | 20 |
| POD penalty owed | 17, 36 |
| Only buyers can dispute | **35** |
| Field agent certifies bad produce | 6 |
| One person, three hats | 31 |
| Phantom POD buyer | 17, **36** |

## What happens as answers land

1. Each answered row gets its **Flow change** filled in — "none", or the concrete edit to the flow.
2. When S1, S2 and S3 are answered, the flow is **redrawn wholesale** on the [questions page](order-delivery-flow-v2-questions.md), and the remaining rows are re-read against the new shape — some will no longer apply.
3. When every row is answered or explicitly deferred, the flow is final, and the build plan is written from it. Not before.
