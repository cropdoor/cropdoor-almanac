# Who we tell, and when

*The eighteen messages the [decided flow](order-delivery-flow-v2.md) promises, drawn against the
moment each one is sent. Settled 13 September. This page is the frame step 2 of
[the order of work](order-delivery-flow-v2-build-order.md) builds; every later step hangs its
messages on it rather than inventing its own.*

## The rule, before the detail

**Everything is written in the app.** That feed is the record of what we told someone, it never
expires, and it cannot be switched off — the one exception we would make is a regulator requiring it,
and the system is built so that switch can be turned on without rebuilding anything.

**Email and text are amplifications of that record, not copies of it.** Email carries detail and
anything with a document. **A text costs money every time**, so it is reserved for the five moments
where someone has to act now. Both can be switched off per kind of message; the record stays either
way.

## The happy path

Time runs down. Every arrow out of CropDoor is a message; **TEXT** marks the five that also send an
SMS, and everything else is in-app, with email where the table below says so.

```mermaid
sequenceDiagram
    autonumber
    actor B as Buyer
    participant CD as CropDoor
    actor F as Farmer
    actor FA as Field agent
    actor DA as Delivery agent

    B->>CD: places an order
    CD->>B: placed — here is your handoff code (TEXT)
    CD->>F: you have a new order
    F->>CD: accepts
    CD->>B: the farm accepted your order
    CD->>FA: an order is waiting for your check
    FA->>CD: records what is actually available
    CD->>B: the check result
    CD->>F: the check result
    F->>CD: marks READY
    CD->>B: ready at the farm
    Note over CD: Admin/Ops assign a crew
    CD->>F: a crew is coming
    CD->>DA: you have a collection today
    F->>CD: hands off; the agent marks IN TRANSIT
    CD->>B: your order is on its way (TEXT)
    DA->>CD: takes cash at the door, if paying on delivery
    CD->>B: we received your cash (TEXT)
    DA->>CD: hands over and photographs it
    CD->>B: delivered
    Note over CD: the payout run, later
    CD->>F: you have been paid
```

## When something goes wrong

```mermaid
sequenceDiagram
    autonumber
    actor B as Buyer
    participant CD as CropDoor
    actor F as Farmer
    actor DA as Delivery agent

    rect rgba(128,128,128,0.12)
    Note over DA,CD: The crew could not collect
    DA->>CD: records COLLECTION FAILED and why
    CD->>B: your collection is delayed (TEXT)
    CD->>F: the collection failed, and why
    end

    rect rgba(128,128,128,0.12)
    Note over DA,CD: The produce could not be delivered
    DA->>CD: records DELIVERY FAILED and why
    CD->>B: we could not deliver, and what happens next (TEXT)
    CD->>F: your produce is coming back
    end

    Note over CD: either one ends in READY again, or a cancellation
    CD->>B: cancelled, and your refund
    B->>CD: raises a dispute
    CD->>B: we have your complaint
```

## Every message, and how it travels

**Buyer — eleven messages**

| When | What they hear | In-app | Email | Text |
| --- | --- | --- | --- | --- |
| They place the order | Placed, with the handoff code for the door | yes | yes | **yes** |
| The farm accepts | The farm has your order | yes | — | — |
| The check is done | What the field agent found | yes | yes | — |
| The farmer marks it ready | Ready at the farm | yes | — | — |
| The crew collects it | On its way | yes | — | **yes** |
| Cash is taken at the door | We received your cash | yes | yes | **yes** |
| It is delivered | Delivered | yes | yes | — |
| The crew could not collect | Delayed, and what happens next | yes | yes | **yes** |
| We could not deliver | Not delivered, and what happens next | yes | yes | **yes** |
| The order is cancelled | Cancelled, and the refund | yes | yes | — |
| They raise a dispute | We have your complaint | yes | yes | — |

**Farmer — seven messages**

| When | What they hear | In-app | Email | Text |
| --- | --- | --- | --- | --- |
| A buyer orders | You have a new order | yes | yes | — |
| A check is scheduled | A field agent is coming | yes | — | — |
| The check is done | What the agent found | yes | yes | — |
| Ops assign a crew | A crew is coming, and when | yes | — | — |
| The crew could not collect | It failed, and why | yes | yes | — |
| A delivery comes back | Your produce is on its way back | yes | yes | — |
| The payout runs | You have been paid | yes | yes | — |

**Staff — their own work, in the same feed**

| Who | When | What they hear | In-app | Email | Text |
| --- | --- | --- | --- | --- | --- |
| Field agent | An order needs a check | This order is waiting for you | yes | — | — |
| Delivery agent | A crew is assigned | You have a collection today | yes | — | — |

**Admin and Ops work watch lists, with one exception.** The lists on
[the flow](order-delivery-flow-v2.md) are a queue they work through, and a notification for every
stalled order is a queue nobody reads. The exception is **a handoff with no IN TRANSIT within the
hour**, which is an alert: it is the only moment where produce has left the farmer's hands and nobody
has yet said they have it.

**When a farm or a buyer has several people, one person is told and everybody sees it.** The owner
gets the text and the email; the rest of the team sees the same message in the shared feed when they
open it. Five people on one farm should not mean five texts about one order.

**A check is offered to every field agent in that farm's zone.** Agents belong to zones, not to
orders, and Operations has no rule yet for picking one. With one agent to a zone today it is the same
message either way.

**The driver gets nothing — for now.** A driver taps nothing today and none of them has an account,
so the delivery agent riding with them is the one we tell. This is "not yet" rather than "never": the
door is left open deliberately, in case drivers are given the app later.

## The five texts, and why only those

A text costs money every time it is sent, and a single buyer could receive eleven messages on one
order. These five are the ones where someone has to do something, now:

- **the handoff code**, because they need it at the door and may not have the app open — and at the
  door, an agent can have it **resent** to the buyer's registered phone;
- **on its way**, because someone has to be there to receive it;
- **cash received**, because it is a receipt for money that changed hands;
- **collection delayed** and **could not deliver**, because both need a decision from the buyer.

Everything else is in the app, with email where there is detail or a document to keep.

## What is not here

Push notifications to a phone, which need a mobile app. Marketing. The buyer↔farm chat, which is a
conversation between two people and stays separate from this.
