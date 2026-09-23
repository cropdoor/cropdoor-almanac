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
| **0 · The floor** | A drop the order-history table · B "refund due" gets a single writer · C1 orders carry their own crew and pickup time · C2 the duplicate delivery record and the farm's dispatch button go · C3 a trip's label and times worked out from its orders, with Cancelled for a trip called off before collection · D1 the day-before reminder goes · D2 the packing step goes | Nothing new. Four duplicates removed; two facts moved to the one place that owns them | The old flow, unchanged in behaviour: place, accept, ready, crew, pickup, confirm, cancel from every state, refund. The farm can no longer dispatch or mark "processing" | A: the parked branch's mutation check. B: mutations on every writer of the flag. C1 and C2: mutations on the four crew guards and on the pickup write, plus a drift check proving the two copies agreed before the record went. A live run after every PR, over every actor and state |
| **1 · Settings and taxes** | E1 the settings catalogue underneath — every setting declares what it holds and what is legal; the delivery fee moves onto it · E2a the settings screen — a tab per group, each tab saved all or nothing · E2b every save recorded — who, the old value, the new — and a save locks what it changes · F1 the three Ghanaian levies go; new orders carry no tax · F2a a tax becomes a row and an order is charged every active one, with the catalogue starting empty · F2b finance maintains taxes — the list, adding one, switching it on and off, deleting one never used · F3 the platform commission rate on the Finance tab — a change adds a new rate from that moment, so an order already priced keeps its commission | The place later steps read their numbers from, and a tax list that starts empty. Two tabs: Delivery, and Finance for taxes and commission. Each number arrives with the step that uses it, not all at once | Admin changes the delivery fee on the settings screen and the next order uses it. A nonsense value is refused, and a tab holding one bad value saves nothing. Every save appears in the change history, with who made it. Finance adds a tax and the next order is unchanged; switches it on and the next order carries it; switches it off and the one after does not. Finance changes the commission rate and the next order uses it, while an order already placed keeps its own | Live after each PR. E2a: every refusal leaves the stored value and its stamp untouched. E2b: two saves at once, the second waiting for the first and recording its value as the old one. F1: an untaxed order's receipt and credit note. F2: the full add, switch-on, switch-off cycle, and a used tax refusing deletion. F3: a rate change that leaves placed orders alone |
| **2 · Telling people** | X1 the feed — in-app as a real channel beside email and text, every message written there first, one channel's failure unable to stop another, read state and the unread count · X2a how a message is sent — the channel chosen per message rather than per kind, and a new order status that tells nobody no longer compiles · X2b the outbox — a message is recorded in the same breath as the thing that caused it, and a dispatcher sends it afterwards, so a restart mid-send loses nothing · X3 who can be told — staff as recipients, the switch for a person's own work, and one owner told for a whole org (the money switch arrives with the message that fills it, in Y) · Y1 the decided channels — each message rides what the team chose for it rather than what its category allowed, and a text about an order arrives unless somebody turns it off · Y2 the messages — the nine that already fire moved onto the frame and onto the list, and the five more whose trigger already exists | The one place that decides who hears what, on which channel, and what a person may switch off. **Ten of the twenty-four messages arrive with the step that creates their trigger**, not here | A buyer opens a feed and sees every message about their order; switching off texts for one kind stops the text and keeps the record; a delivery agent is told they have a collection today; a farmer is told when an order is cancelled and when they have been paid | Live: every message the frame can send, to the right person, on the right channel, with the feed still holding it after both other channels are off — and the handoff code proved un-switchable |
| **3 · Money rails** | G refunds of a stated amount, with the credit-note line · H the payout run reads the ledger: pays what is owed, skips an order with an open dispute or one still inside its dispute window · T1 the agent's collected cash recorded as received · I refund a cash buyer by mobile money | The four money moves every later step needs, each usable on its own | Admin refunds part of an order and the ledger and credit note agree. A disputed order is skipped by the payout run; it is paid once resolved. A farm selling for cash can be paid at all. A cash order's refund reaches the buyer's phone | Mutations on all four. G and I against Paystack test keys |
| **4 · The farm** | J the availability check — the agent's form, three photos a line, the derived outcome, Short and Not-available handled, the overdue list · K READY and the crew gate — no check, no crew; the no-field-agent exception; the acceptance and READY deadlines; the crew-assigned text · L the cancellation windows — free until READY plus grace; the penalty from escrow to the farmer; strikes for cash buyers, cash switched off after the limit; farm strikes; Admin/Ops cancel with a fault | The farm side of the decided flow. **Ops can start operating on it:** field agents check, farmers mark READY, Admin/Ops assign crews from the order detail | Accept → check (all four outcomes) → READY → crew → pickup → confirm, online and cash. Cancel in each window and see the right refund and penalty. A buyer with too many strikes is refused cash on delivery | Mutations on L. Live run after L with every window and every check outcome, flows listed and reviewed first |
| **5 · The gate** | M HANDOFF by the farmer, IN TRANSIT by the delivery agent, the alert when one follows the other too slowly · N COLLECTION FAILED — the reasons, nothing taken, the check made void, retry through READY, free cancel while it lasts · O the farmer paid in full after HANDOFF whatever follows, funded by the penalty and by us | Custody as two people's words; the first failure state; the farmer's money settled | The farmer taps HANDOFF, the agent taps IN TRANSIT, the buyer sees "on its way". The crew records a failed collection and the order comes back through READY or is cancelled. An order cancelled after HANDOFF still pays the farmer | Mutations on O. Live run after O |
| **6 · The door** | P the handoff code — sent at placement, verified at the door with no signal · Q the delivery photo, required · R DELIVERY FAILED, the return to the farm, the alerts · S the second attempt — the buyer asks and pays the fee again, or the order is cancelled with a strike · T2 the "cash received" text and the agent's daily remittance round | The door as decided; every state now has an exit | The full flow, online and cash, with every failure branch: wrong code, no photo, not paid in full, nobody home, a second attempt, a cash day remitted | Mutations on S. **The big live run:** the whole flow end to end, both payment methods, every branch |
| **7 · After delivery** | U disputes — produce or delivery, the payout held, Admin/Ops name the amount and who bears it, the refund follows, the five-day clock · V two ratings · W the API document rewritten; the old flow pages retired | The flow complete | A buyer disputes, Admin/Ops resolve against the farm or against us, the buyer is refunded, the farmer is paid what is left at the next run. Two ratings on one order | Mutations on U. Live run of a dispute through to a mobile-money refund |

Thirty-seven PRs. Steps 0 to 3 are deletions, plumbing and the message frame; step 4 is the first thing Operations can use; step 6 is the first time the whole decided flow exists.

## Where each PR stands

The one place that says what is finished. A PR is **Done** only once it is merged, and the number beside it is the evidence — open it and you see what was built, how it was tested and what was observed running. Updated twice per PR: when it opens, and when it merges.

| Step | PR | What it does | Status | Evidence |
| --- | --- | --- | --- | --- |
| 0 · The floor | A | the order-history table goes | **Done** | #224 |
| 0 · The floor | B | "refund due" gets a single writer and a guard | **Done** | #226 |
| 0 · The floor | C1 | orders carry their own crew and pickup time | **Done** | #227 |
| 0 · The floor | C2 | the duplicate delivery record and the farm's dispatch button go | **Done** | #228 |
| 0 · The floor | C3 | a trip's label and times are worked out from its orders | **Done** | #229 |
| 0 · The floor | D1 | the day-before reminder goes | **Done** | #230 |
| 0 · The floor | D2 | the packing step goes | **Done** | #231 |
| 1 · Settings and taxes | E1 | every setting declares what it holds and what is legal; the delivery fee moves onto it | **Done** | #233 |
| 1 · Settings and taxes | E2a | the settings screen — a tab per group, saved all or nothing | **Done** | #234 |
| 1 · Settings and taxes | E2b | every save recorded — who, the old value, the new — and a save locks what it changes | **Done** | #235 |
| 1 · Settings and taxes | F1 | the three Ghanaian levies go; new orders carry no tax | **Done** | #236 |
| 1 · Settings and taxes | F2a | a tax is a row, and an order is charged every active one; the catalogue starts empty | **Done** | #237 |
| 1 · Settings and taxes | F2b | finance maintains taxes — the list, adding one, switching it on and off, deleting one never used | **Done** | #238 |
| 1 · Settings and taxes | F3 | the platform commission rate on the Finance tab | **Done** | #239 |
| 2 · Telling people | X1 | the feed — in-app as a real channel, every message written there first, one channel's failure unable to stop another | **Done** | #240 |
| 2 · Telling people | X2a | how a message is sent — the channel per message, and a status that tells nobody no longer compiles | **Done** | #241 |
| 2 · Telling people | X2b | the outbox — recorded with the thing that caused it, then sent by a dispatcher that can retry | **Done** | #242 |
| 2 · Telling people | X3 | who can be told — staff as recipients, the work switch, one owner told per org | **Done** | #245 |
| 2 · Telling people | Y1 | each message picks its own channels, and an order text arrives switched on | **Done** | #248 |
| 2 · Telling people | Y2 | the four new messages whose trigger already exists, and the money switch | **Done** | #256 |
| 2 · Telling people | Y3 | the three silences Y2 made visible, and the crew change that spoke twice | **Done** | #257 |
| 2 · Telling people | Y4 | the refund nobody mentions — the buyer hears that a refund started, and that it was sent | In review | #282 |
| 3 · Money rails | G1 | a refund carries its own name at the gateway, so two on one payment can be told apart | **Done** | #281 |
| 3 · Money rails | G2 | refunds of a stated amount, and who bears them | Not started | — |
| 3 · Money rails | G3 | one credit note per refund, and the penalty line | Not started | — |
| 3 · Money rails | T1 | the agent's collected cash recorded as received, so a cash order can be paid at all | **Done** | #277 |
| 3 · Money rails | H | the payout run reads the ledger and skips a disputed order | **Done** | #276 |
| 3 · Money rails | I | refund a cash buyer by mobile money — and collect the buyer payout destination nothing else collects | Not started | — |
| 4 · The farm | J | the availability check — the form, the photos, the outcomes, the overdue list | Not started | — |
| 4 · The farm | K | READY and the crew gate, with the acceptance and READY deadlines | Not started | — |
| 4 · The farm | L | the cancellation windows, the penalty, and strikes | Not started | — |
| 5 · The gate | M | HANDOFF by the farmer, IN TRANSIT by the delivery agent, and the alert between them | Not started | — |
| 5 · The gate | N | COLLECTION FAILED — reasons, the check made void, retry through READY | Not started | — |
| 5 · The gate | O | the farmer paid in full after HANDOFF, whatever follows | Not started | — |
| 6 · The door | P | the handoff code — sent at placement, verified at the door with no signal | Not started | — |
| 6 · The door | Q | the delivery photo, required | Not started | — |
| 6 · The door | R | DELIVERY FAILED, the return to the farm, the alerts | Not started | — |
| 6 · The door | S | the second attempt — the buyer asks and pays again, or the order is cancelled with a strike | Not started | — |
| 6 · The door | T2 | the "cash received" text and the agent's daily remittance round | Not started | — |
| 7 · After delivery | U | disputes — the payout held, Admin/Ops name the amount and who bears it | Not started | — |
| 7 · After delivery | V | two ratings | Not started | — |
| 7 · After delivery | W | the API document rewritten; the old flow pages retired | Not started | — |

Nineteen of thirty-nine are merged: **steps 0 and 1 are complete**, step 2 is three PRs in, and step 3 has two — H, the payout reading the ledger, and T1, the cash an agent is holding, which is what made a cash order payable at all. **#232 and #243 are not among them:** #232 corrected a Javadoc about the buyer's cancellation window, found while building step 0, and #243 lowered how many messages the dispatcher sends at once, decided while reading X2b's live run.

## Work that is not in this order

The table above tracks the thirty-seven PRs that build the flow. A second thread ran alongside it in September: the documents the platform issues, and the brand they carry. It is recorded here because it shipped and nothing else says so — not because it belongs to a step.

| | What it did | Status | PR |
|---|---|---|---|
| Receipts | the delivery fee the buyer paid, on the document that explains it | **Done** | #258 |
| Orders | an order carries a delivery fee, always | **Done** | #260 |
| Documents | the key write at issue was never committing, so every document was drawn twice | **Done** | #261 |
| Documents | the download renders without holding a database connection | **Done** | #262 |
| Documents | one way to draw a document — receipts and credit notes stop being two copies | **Done** | #263 |
| Documents | the logo a document already has, instead of fetching it on every render | **Done** | #264 |
| Emails | the brand URL emails read, so the logo can move with one setting | **Done** | #265 |

Two of these were not planned. #261 and #262 were found by writing the design down before the code: the first meant every receipt was rendered twice and neither copy was recorded, the second held a database connection open through a rasterise and an upload. Neither was visible from the outside.

The brand logo now lives in CropDoor's own bucket rather than a personal Cloudinary account. Two pieces remain: the marketing pages use a different account, and one email still borrows an icon from a free CDN.

## What changed since 10 September

This page is the living order of work, so it is corrected as the CTO decides and as the work lands. Nine corrections so far: four to step 0, two to step 1, two to step 2, and one to step 3.

- **"Refund due" is stored with a single writer, not derived.** Decided by the CTO during B, 12 September. The flag now has exactly one writer called from every path that can change the answer, and a report-only check that names any order where the flag and the refund records disagree. The reason to store it stands on its own: an admin list that filters on it should not re-run a derivation per row, and one writer is easier to prove correct than a rule copied into every reader. Nothing is derived twice either way.
- **C became three PRs, and the run's status is still stored.** C1 put the crew and pickup time on the order and proved the copies agreed; C2 deleted the duplicate record and the farm's dispatch button. Deriving the run's status turned out not to be needed for either, so it is C3 and still to do. It is worth keeping inside step 0 rather than deferring: it is a deletion, the surface shrinks before everything after it, and the stranded-run problem Operations sees today comes from that stored status being the authority.
- **A trip called off before the van went reads Cancelled.** Decided by the CTO on 13 September, with two smaller answers alongside it: a finished trip that gets a new order reads In progress, and a trip's start and finish times are worked out from its orders rather than removed. All three are questions 44 to 46 on the [working-answers page](order-delivery-flow-v2-working-answers.md). They settle what C3 builds; they do not move it in the order.
- **D became two PRs.** The day-before reminder and the packing step shared nothing, so they shipped separately (#230, #231) rather than as one change carrying a farmer-visible button removal alongside a dormant job.
- **Step 2 is five PRs, and thirty-six in all.** Decided by the CTO on 15 September, after looking at how a message is actually sent today. It is published inside the app and delivered on a background thread — so if the app restarts between the order being saved and the message going out, which a deploy does routinely, **the message is lost and nothing records that it ever existed**. Separately, that background worker is unbounded while the database allows twenty connections at once, so a burst of messages can exhaust them and the ones that lose are dropped silently. **X2b** answers both: a message is written down in the same breath as the thing that caused it, and a dispatcher sends it afterwards. A restart loses nothing, a failed send can be tried again, and how many go at once is finally bounded. It is **less** machinery than what exists today, not more — it removes the background hop and the coupling that lets one channel's failure stop another — and it needs no new infrastructure, because the database we already have is the queue.

- **I turns out to be the only place a buyer can be paid, and it is bigger than cash refunds.**
  Found 23 September while specifying the refund notifications. We hold everything needed to pay a
  farm — mobile-money number, network, bank code, account number, account name, and a screen that
  asks for them. For a buyer we hold a verified phone number and nothing else: no network, no bank
  details, and no screen that asks. Two consequences. A cash refund cannot be paid at all, which is
  what I was always for. And an **online** refund that the provider cannot complete — it asks us for
  a destination — cannot be finished **by our own code**, because the retry call needs a destination
  we have never stored. *Corrected 23 September:* the first draft of this note said such a refund
  "cannot be rescued by anyone", which is wrong. A person can complete it on the provider's
  dashboard, where the money may go to any account belonging to the customer, and the admin queue
  that surfaces it already exists. So I is still the buyer payout destination — and still what makes
  a cash refund possible at all — but a stuck online refund has a manual route home today. Online
  refunds do normally complete: a live mobile-money refund settled on 23 September.

- **G splits three ways.** Decided 22 September, on a second review of G's own design. G was one PR
  containing a gateway contract change, a ledger arithmetic change, a new feature, a document rework
  with a data migration, and two event listeners — each independently verifiable, which is the test
  this build order applies everywhere else. **G1** gives a refund its own identity at the gateway and
  changes no money behaviour: today every refund on a payment shares the charge's reference, so with
  two the platform cannot say which one a webhook settles. **G2** is the money — the stated amount,
  who bears it, and the credit note's amount, which travels with G2 rather than G3 because G2 alone
  would print "Refunded in full" on a partial refund. **G3** is what a *second* refund needs: one
  note per refund instead of one per order, and the penalty line. The cut also keeps the step's
  acceptance test — "the ledger and credit note agree" — inside the PR that would otherwise break it.

- **T splits, and the half that matters for money comes forward into step 3.** Decided by the CTO on 21 September, on evidence from H's live run. Settling a cash order writes that the platform float paid out, but the notes are in the delivery agent's bag — so the books say we hold cash we have not received, and the payout was sending real money against it. H refuses that outright, which is safer and leaves a farm selling for cash unable to be paid at all: on the development database one farm is owed 335.70, of which 254.70 is nineteen delivered cash orders with no route to payment. Waiting for step 6 would leave those farms unpaid for twenty PRs. So **T1** — the agent's collected cash recorded as received, and posted where it belongs — lands right after G, and **T2** keeps the agent-facing half at the door: the "cash received" text and the daily remittance round.

- **Step 2 is four PRs, and it cannot carry every message.** Decided by the CTO on 15 September (questions 56–62). Of the twenty-four messages [the messages page](order-delivery-notifications.md) names, five are sent today, four more already fire without ever having been on the list, and five can be sent as soon as there is a frame — ready at the farm, a crew is coming, you have a collection today, you have been paid, and "we have your complaint" to the buyer. **Ten cannot**, because nothing triggers them until the check (step 4), the gate (step 5) and the door (step 6) exist, so each of those steps brings its own messages onto the frame instead. The frame itself was one PR larger than anything shipped so far, so it splits in three: the feed, how a message is sent, and who can be told. Four decisions came with it — **four messages the platform already sends** (a cancellation to the farmer, and the three dispute messages) join the list rather than being dropped; **a person switches off a kind of message, not a message**, and there are four kinds; **the handoff code can never be switched off**, because it is a credential and the agent's fallback at the door is to resend that same text; and there are **no quiet hours and no Ops switch for texting** — every text is a message someone must act on, and a switch that silently stopped "the collection failed" is worse than the spend it would save.

- **F2 became two PRs.** As specced it was a table, four permissions, five admin routes, an audit event, the pricing change and a test-isolation rule in one change — the size that was split once already, for E2. **F2a** makes a tax a row and prices every active one, with a catalogue that ships empty and that nothing can add to, so no price can move. **F2b** is the surface finance uses: the list, adding a tax, switching it on and off, and deleting one never used. Step 1 is seven PRs.
- **Step 1 is six PRs, and the screen saves a whole tab.** The web app's settings screen is a tab per group with one Save button, so saving is per tab and all or nothing: one bad value and nothing on the tab is stored (E2a, #234). The record of each save and the lock that keeps it true became their own PR, E2b, so the screen could ship without them; today's fee save had neither, so nothing regresses in between. Asked by the CTO on 13 September to decide what else the screen should carry, engineering added **commission as F3**, because every order already reads it, on a Finance tab beside taxes. The other fields stay off: **minimum order** waits for launch and two answers only Operations can give (per farm or per basket; before or after the fee); **supported regions** are never a setting, because zones already decide where CropDoor operates; **maintenance mode** is not a switch inside the app, because it would have to keep payment webhooks and admin login working; **new registrations** waits for launch planning, because signup has several doors and a switch that misses one lies. **No Security tab:** multi-factor rules stay in role management, and token lifetimes and rate limits in deployment settings, so one compromised admin cannot weaken security for everyone. Two answers from the CTO the same day: the delivery fee has **no business ceiling** until Operations or finance want one, and **a save that changes nothing still appears in the history**, so it answers who saved a setting and when.
- **Step 1 is four PRs, not two, and does not register every number at once.** Decided by the CTO on 13 September (questions 49–55). A setting arrives with the step that reads it, because one Ops can change that nothing consults looks like it works. The settings catalogue and its screen split in two, and so do taxes: the three levies are deleted first, then the tax list is built — delete before add, as in step 0.

## Why this order

- **Deletions first** because every later PR is smaller and safer on the reduced surface, and because the four deletions were already audited on 2 September.
- **Settings before anything that reads a number**, so no step ships with a number hardcoded and moved later.
- **Telling people before anything that has something to tell them.** The decided flow names twenty-four messages and **nine** of them fire today — five that were on the list, and four that fire without ever having been on it. Those four were decided on 15 September (question 56): they are kept, and written onto the list. Without a frame, each later step invents its own message, its own channel and its own opt-out, and the platform ends up with twenty-four local decisions instead of one design. Money is the first step with something to say — a refund, cash received — so the frame goes before it. **In-app is new**: today a message exists only as a sent email or text, so if a person switches both off nothing survives and "what did we tell this buyer?" has no answer.
- **Money rails before the transitions that post** — but each rail is usable on its own the day it lands, through an admin action, so it is tested end to end and not "callable but unused".
- **The farm before the door** because it is where the old flow is weakest (no check, no gate on the crew) and where Ops can start operating soonest. Between steps 3 and 4 the old pickup button still moves an order to IN TRANSIT, so the flow never has a gap.
- **Disputes last** because they read everything before them: the delivery photo, the check photos, the farmer's payable, the refund rails.

## What has to be true before the first PR

- The [decided flow](order-delivery-flow-v2.md) stays as decided. Any change there reorders this page, not the other way round.
- The engineering spec behind this page has had its review round. It is the local design document engineering builds from; this page is the agreement on the order.
- A plan per step, listing each PR's tasks, files, tests, mutations and live-verify flows, written and reviewed before that step starts — step 0's is drafted.
