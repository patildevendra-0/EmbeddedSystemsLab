# USB — Explained Simply

*(Same example as the USB Interactive Lab: plugging a USB Keyboard, Flash Drive, Camera, or Mouse into a computer)*

---

## 1. The real-life scenario: Checking into a secure office building

Imagine a big office building (the **computer / Host**) with a strict front-desk receptionist (the **USB Host Controller**). Nobody just walks in and starts working. Every visitor (a **USB Device** — keyboard, flash drive, camera, mouse) has to check in first, and the receptionist decides everything about when and how they get to interact with the building.

This is different from the CAN "meeting room" example — in CAN, everyone shares one microphone and any of them can start talking. In USB, **only the receptionist (the host) decides when anything happens.** A visitor never just barges into a room and starts talking on their own.

---

## 2. Who's involved in our example

| Real world | USB term |
|---|---|
| The office building | **Host** (your computer) |
| The receptionist who controls everything | **Host Controller** |
| A visitor arriving at the front door | **Device** (e.g. USB Keyboard) |
| The visitor's ID card, listing name, department, badge number | **Descriptors** (Device Descriptor, Configuration Descriptor) |
| The visitor's temporary visitor badge number | **Device Address** |
| The specific rooms/desks the visitor is allowed to use | **Endpoints** |
| The reason the visitor is here (typing, delivering files, recording audio) | **Transfer Type** (Interrupt, Bulk, Isochronous) |

---

## 3. What actually happens, step by step — plugging in a USB Keyboard

This is the same sequence the app's "Run First Demo" plays out.

### Step 1 — The visitor walks in the door
You physically plug the keyboard in. The receptionist's front-door sensor notices someone arrived — this is **device connect detection**.

### Step 2 — "Please start over, you have no identity yet"
Before anything else, the receptionist makes the visitor go through a reset — like saying *"Whatever badge you think you have, forget it, you're starting fresh."* This is the **USB Reset**. The device goes back to a totally default, blank state.

### Step 3 — The visitor is known only as "Visitor Zero"
Right after reset, the device has no identity of its own yet. Every newly reset device is temporarily addressed as **Address 0** — like every new visitor being called "Visitor" until they get an actual badge number.

### Step 4 — "Who are you? Show me your ID"
The receptionist asks: *"What kind of visitor are you, and what department?"* This is the host sending **GET_DESCRIPTOR**. The keyboard hands over its **Device Descriptor** — a small card listing:
- What kind of device it is (**HID class** = "I'm an input device")
- Its manufacturer and product numbers (**VID/PID** — like a company ID number)
- How big its "sentences" can be (**max packet size**)

### Step 5 — "Here's your visitor badge: #5"
Now that the receptionist knows who this is, they issue a real badge number. This is **SET_ADDRESS**. From now on, the device is called "Address 5," not "Visitor Zero" anymore.

### Step 6 — "Which rooms do you actually need access to?"
The receptionist asks for more detail: *"What's your full itinerary? Which desks/rooms will you use, and for what?"* This is **GET_DESCRIPTOR (Configuration)**. The keyboard replies with its full layout:
- **1 Interface** — "I'm a keyboard" (a single function)
- **1 Endpoint** — a specific "mailbox" (EP1 IN) where it will drop key-press reports

*(A flash drive would instead list 2 endpoints — one for sending data in, one for receiving data out. A more complex device — like a keyboard with a built-in trackpad — could list multiple interfaces, similar to one visitor badge covering two different departments.)*

### Step 7 — "You're approved — go ahead and get to work"
The receptionist finalizes everything with **SET_CONFIGURATION**. Only now can the device's actual "mailboxes" (endpoints) be used. Before this moment, the keyboard could only talk through the receptionist's front desk (Endpoint 0) — never directly do its real job.

### Step 8 — The building's directory assigns the right department
The building's directory (the **operating system**) sees "HID class" on the badge and automatically routes this visitor to the right department — the **HID driver**, which knows how to handle keyboards and mice. You never had to install anything by hand for something this standard.

### Step 9 — Now the visitor can actually do their job — but only when asked
Here's the twist that surprises most people: even now, the keyboard **cannot just shout out a keypress whenever it wants.** The receptionist walks by the keyboard's mailbox every few milliseconds and asks, *"Anything new for me?"* — this is **host polling** (an **Interrupt transfer**). If you haven't pressed a key, the keyboard says *"nothing yet"* (a **NAK**). The moment you press a key, the next time the receptionist checks, the keyboard hands over the report, and it instantly appears as a keystroke on your screen.

---

## 4. Same building, different visitor: a Flash Drive

A flash drive goes through the *exact same* check-in process (reset → identify → badge number → configure) — but its "job description" is different:

- Its endpoints are for **Bulk transfer** — think of it as a big delivery truck: it moves large amounts of data reliably, but the receptionist doesn't promise it a fixed time slot. It gets access whenever the building has spare capacity, and if something's misheard, it's simply repeated until it's right.
- Compare that to the keyboard's **Interrupt transfer** — a quick, regular check-in every few milliseconds, like a security guard doing rounds.
- A microphone or camera would use **Isochronous transfer** — imagine a live news broadcast: it gets a *guaranteed* time slot every single round, but if a moment is missed, nobody goes back to re-record it — a late "makeup" delivery would be useless anyway. Timeliness matters more than perfection.

---

## 5. What happens when something goes wrong

- **The receptionist asks something the visitor can't currently answer** (e.g. asking a keyboard for data before you've pressed anything) → the device politely says *"nothing right now"* (**NAK**). This is completely normal, not a failure — the receptionist just asks again shortly after.
- **The receptionist asks something the visitor flatly cannot do** (an unsupported request) → the device says *"I can't do that"* (**STALL**) and the receptionist has to clear that block before continuing.
- **A message gets garbled on the way** (background noise) → whoever received it notices the checksum doesn't add up (**CRC error**), quietly ignores it, and the sender — hearing no confirmation — repeats itself.
- **The visitor storms out mid-conversation** (device unplugged) → the receptionist notices the door sensor went quiet, tells the building directory *"that visitor left,"* and everything about them is cleared. If they walk back in later, they have to check in again from scratch — nothing is remembered from before.
- **The visitor hands over a fake or broken ID card** (a malformed descriptor) → the receptionist refuses to let them proceed, and the check-in process fails outright — you'd see this as "unrecognized USB device" on your screen.

---

## 6. Why the receptionist matters so much (Host vs. Device)

This is the single most important idea in USB: **the device never gets to decide when it talks.** Even after it's fully checked in and configured, it can only respond when the host asks. This is very different from CAN, where every ECU can just start talking onto the shared bus whenever it wants and let arbitration sort out who wins.

- In **CAN**, all "people" share one microphone and anyone can start speaking (then priority sorts it out).
- In **USB**, there's a strict receptionist, and nobody speaks unless spoken to — even a "fast" or "important" device just gets checked more frequently, not given free rein.

This is exactly why USB needs an entire **check-in process (enumeration)** before anything useful happens, while CAN devices can start broadcasting the moment the bus is free.

---

## 7. One-paragraph summary

Plugging in a USB device is like a strict office check-in: the visitor (device) is reset to a blank identity, temporarily called "Visitor Zero" (Address 0), then hands over an ID card (descriptors) describing what kind of visitor it is. The receptionist (host) issues a real badge number (SET_ADDRESS), asks for the visitor's full itinerary of rooms and departments (configuration/interfaces/endpoints), and only after formally approving it (SET_CONFIGURATION) does the visitor get real access. From that point on, the visitor still never acts on its own — the receptionist decides exactly when to check in with them, whether that's a quick, frequent glance (Interrupt, like a keyboard), a big reliable delivery whenever there's spare time (Bulk, like a flash drive), or a guaranteed slot every round with no do-overs (Isochronous, like a microphone or camera). This whole check-in-then-controlled-conversation model is exactly what the USB Interactive Lab lets you watch happen, step by step, using a keyboard, flash drive, camera, and mouse as the "visitors."