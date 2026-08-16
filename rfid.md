# NFC & RFID — What Actually Happens (Simple Notes)

**One example used everywhere in this doc:**
👉 **You tap your bank card on a shop's payment machine (POS terminal).**

Every concept below is explained using this same example, step by step, so you can follow one real story from start to finish instead of jumping between random examples.

---

## 1. The big picture (in one line)

> The payment machine creates an invisible radio field. Your card, which has no battery, "wakes up" using power from that field, and then the machine and card talk to each other in a fixed sequence of steps until the payment goes through.

That's it. Everything below is just zooming into each part of this sentence.

---

## 2. What is inside your bank card?

Your card looks like plain plastic, but inside it has:

- A **tiny coil of wire** (the antenna) — usually you can't see it, it's a thin loop going around the edge of the card
- A **tiny chip** connected to that coil
- **No battery**

This is called a **passive tag**. It has no power of its own — it depends completely on the reader for power.

---

## 3. Step 1 — The machine creates a radio field

Even before you tap, the payment machine is constantly sending out a **weak radio signal** at **13.56 MHz** (this is the NFC/HF frequency — same frequency used for most bank cards, metro cards, and access badges).

This is like the machine "shouting into the air" 13.56 million times per second, all the time, waiting for a card to come close.

You cannot see or feel this — but it's real, physical, electromagnetic energy sitting right in front of the machine.

---

## 4. Step 2 — You bring the card close → the card gets power

When your card enters that field:

- The coil inside your card picks up the radio energy (this is called **inductive coupling** — same basic idea as a wireless charger for your phone)
- That energy is converted into a tiny electric voltage
- This voltage powers up the chip inside your card

**Important real-life detail:** the closer and better-aligned the card is, the stronger the power it gets. That's exactly why:
- Tap too far away → card doesn't power up → "not detected"
- Card sideways/tilted → weaker power → sometimes fails
- Card behind a metal object → field gets blocked → fails

This is not the machine's fault or card's fault — it's just physics of the radio field getting weaker with distance and bad angle.

---

## 5. Step 3 — The machine asks "is anyone there?"

Once your card is powered, the machine sends a very short radio command asking:

> "Any card in range, please respond."

(Technically this command is called `REQA` for the most common card type, NFC-A / ISO 14443-A — but you don't need to remember the name, just the idea: **machine polls, card answers**.)

Your card, now powered, replies with a short "yes, I'm here" signal.

---

## 6. Step 4 — What if you have 2 cards in your wallet? (Collision)

This is a very common real situation: your bank card and metro card are both in the same wallet, and you tap the whole wallet.

Both cards get powered by the same field. Both try to answer the machine's question **at the same time**. Their answers overlap and get scrambled — this is called a **collision**.

The machine notices the response is garbled and runs an **anti-collision** process:

- It asks again, but this time in a smarter way that lets it tell the two cards apart bit by bit (like asking "does your ID start with 0? does your next digit start with 0?" repeatedly)
- Eventually, it narrows it down and identifies **one single card's ID**
- It tells that one card "you are selected, others please stay quiet"

This is why sometimes tapping a full wallet fails or picks the wrong card — the machine genuinely got confused by multiple replies, and anti-collision is trying to sort out that confusion.

---

## 7. Step 5 — Every card has an ID (UID) — but it is NOT a password

Every card has a unique ID number (called **UID**), something like:

```
UID: 04 A3 91 72 6B 4F 80
```

The machine uses this ID only to tell your card apart from other cards nearby — like a name tag, not a secret code.

**Important real understanding:**
> UID is NOT the same as a password or a secret key. It's just an identifier. If a system only checks "is this UID in my allowed list?" and nothing else, someone who copies that ID number could potentially pretend to be your card. That's why serious systems (like real payment cards) never rely on UID alone — they use real cryptographic authentication (next section).

---

## 8. Step 6 — The machine and card do a "secret handshake" (Authentication)

Once your card is selected, the payment app on the machine talks to the payment app on your card's chip. This involves:

1. Machine picks which application on the card to use (e.g., "Visa payment app")
2. Machine sends a random challenge number to the card
3. Card's chip uses its **secret cryptographic key** (never sent over the air, never leaves the chip) to calculate a response to that random number
4. Card sends back the calculated response
5. Machine verifies the response is mathematically correct

Because the actual secret key never leaves the card, and the challenge number is random and different every time, this cannot be faked just by copying the UID. This is real security, unlike a bare UID check.

---

## 9. Step 7 — Data is actually exchanged (Memory / transaction data)

Your card also has small memory blocks storing data (account info references, transaction counters, etc — real payment cards don't store your raw account number readable in plain memory, this is simplified for understanding).

The machine reads/writes small chunks of this memory as part of finishing the transaction — for example, incrementing a transaction counter so the same "response" can never be reused for a second fake transaction (this stops simple replay attacks).

---

## 10. Step 8 — Transaction completes → Application decides result

After all the above steps finish (in well under 300 milliseconds — faster than you can blink), the machine's payment application decides:

- ✅ **Approved** → beep, green light, receipt prints
- ❌ **Declined** → error shown

This final decision is made by the **application layer** — the actual banking software — not by the radio/protocol layer. The radio and protocol steps above just made sure the right card was found, talked to securely, and gave the right data. What to *do* with that data (approve/decline a payment) is a separate decision made by the bank's software.

---

## 11. Same story, but for other real cases (quick notes)

| Real scenario | What's different from the bank card example |
|---|---|
| **Metro/transit card tap** | Same steps 1–7. Instead of a bank app, the card's transit app checks balance and deducts fare. Faster because less/no cryptographic negotiation on some systems. |
| **NFC phone payment (Apple Pay/Google Pay)** | Phone doesn't "read" a card — it *pretends to be* one. This is called **card emulation**. Phone's secure chip runs the same handshake described above, phone → machine, exactly like a real card would. |
| **Door access badge** | Many cheap door systems skip Step 6 (real authentication) and only check the UID (Step 5). This is why access badges can sometimes be copied with a cheap cloner — it's copying the UID, not breaking any encryption, because there was no real encryption to begin with. |
| **Warehouse RFID inventory tag** | Completely different frequency (UHF, ~900 MHz, not 13.56 MHz) and much longer range (meters, not centimeters). Instead of "load modulation" (how HF cards answer), UHF tags answer by **reflecting the radio wave back differently** — called **backscatter**. Reader can read hundreds of tags a second this way. |
| **Animal/asset RFID chip** | Usually simplest of all — just a fixed ID number, LF or HF frequency, no encryption, no app logic. Purely "who is this." |

---

## 12. Why NFC and RFID are not the same thing (in one line)

> RFID is the big family (LF + HF + UHF + more). NFC is one specific member of that family — it only uses the HF band (13.56 MHz) and adds its own rules for phones, tags and apps (like the card-emulation trick your phone uses for payments).

So: **your bank card, your phone's tap-to-pay, your office badge, and your metro card are all "NFC."**
Your warehouse box tags and animal chips are "RFID" but usually **not** NFC (different frequency, different reader hardware entirely).

---

## 13. Common real-world failures — explained with the same card example

| What you see | What's actually happening |
|---|---|
| "Tap failed, try again" | Card too far / bad angle → not enough power delivered → machine never even got a response |
| Wrong card charged from wallet | Multiple cards answered → anti-collision picked a different one than you expected |
| "Card error" after long pause | You pulled the card away mid-transaction → field lost → card lost power mid-handshake → transaction aborted |
| Payment declined instantly | Card powered fine, ID found fine, but the bank's authentication/application check failed (wrong app, expired card, bank server said no) — nothing to do with the radio part at all |
| Door badge cloned easily | That badge system only checks UID (Step 5) and skips real authentication (Step 6) |

---

## 14. One-paragraph summary you can remember

> When you tap a card, the reader is already broadcasting a small radio field. Your powerless card borrows energy from that field to wake up. The reader then asks "who's there," and if more than one card answers at once, it sorts them out with anti-collision until one card is selected. That selected card and the reader then do a secure back-and-forth (a real cryptographic handshake, not just an ID check) to confirm it's genuine, read/update a bit of memory, and hand the result to the actual payment/access/inventory software — which makes the real decision to approve or deny. The radio and protocol steps only find and verify the tag; the application on top decides what happens next.