# Leonidas Demo Script — Delmar Walkthrough

**Audience:** Delmar (CEO, Spartan Energy Services)
**Format:** Screen share of live mockup (spartan-leonidas.vercel.app)
**Duration:** 30–40 minutes
**Goal:** Validate that Leonidas captures how Delmar actually works, surface what's wrong or missing, and confirm priority order for Phase 1 build.

---

## Before the Call

- Have the mockup open and logged in (click Sign In on the login screen)
- Have the Planning Board loaded as your starting screen
- Know the sample data: 7 jobs across 3 districts, 2 with shortfall flags (Diamondback Midland B12 and Ovintiv Eagle Ford 5)
- Don't over-rehearse — this should feel like a working session, not a pitch

---

## 1. Recognition — "Here's What I Heard" (2–3 min)

Don't show anything yet. Just talk.

> "Before I show you anything, I want to make sure I got it right from our last conversation. You're running about 30 jobs at any given time across Keithville, Midland, and Pleasanton. Every two days, you and Don sit down with the spreadsheet and basically play Tetris — figuring out who's short, what's coming off a job, whether you can convert sizes, whether you need to pull from another district or sub-rent. That process takes you a few hours each time, and it's the thing that eats a third of your day."
>
> "The spreadsheet works — it's gotten you to $70M — but it can't answer basic questions fast. Like 'how many 7-inch 15K hydraulics are available in Midland right now?' or 'if I take this new job, what breaks?' You have to scroll and do the math in your head every time."
>
> "The other piece is your DMs have no visibility. They can't look at their own yard and know what's available — they wait for you to tell them. And the consignment tracking, the machine shop, the delivery tickets that are 4 days late into Basis — all of that feeds back into you being the bottleneck."
>
> "Did I get that right? Anything I'm missing?"

**Why this matters:** Delmar told you all of this in the discovery call. Saying it back to him proves you listened and frames everything that follows as "we built this to solve YOUR specific problems."

Let him respond. He'll probably add color or correct something. That's good — take notes. Then transition:

> "Alright, so what I want to do today is walk through the system we've been building and show you how these same workflows would look. I'm going to start with the thing you do most — a new job comes in and you need to figure out if you can take it."

---

## 2. New Work Order — Current vs. Leonidas (10–12 min)

### 2a. Set the Scene (Current State)

> "So a frac schedule comes in. Let's say Vital Energy calls — they've got a 6-well pad in the Wolfcamp, they need you out there April 14. What do you do today?"

Let Delmar describe it. He'll say something like: "I open the spreadsheet, add a new row, start filling in what they need — 8 wing valves, 6 masters, 4 zippers — and then I scroll down and see if the numbers go negative."

> "Right. And when the numbers go negative, that's when the real work starts — you're scanning every row looking for who's coming off a job, whether you can convert sizes, whether Keithville has surplus. That's the part that takes hours."

### 2b. Show the New Work Order Tab

Click **+ NEW WORK ORDER** tab on the Planning Board.

> "Here's the same thing in Leonidas. Same job — Vital Energy, Wolfcamp A-9H Pad, Midland, starting April 14."

Walk through the form top to bottom:

**Job Information section:**
> "Customer, well site, district, dates — same info you'd put in the spreadsheet. We also added lat/long so the system knows exactly where the job site is for logistics routing."

**Equipment Requirements section:**
> "Here's where it gets different from the spreadsheet. Instead of free-text columns, you're picking from a standardized equipment list — Wing Valve Hyd, 4-inch, 15K, quantity 8. Same equipment types you track today, just structured so the system can do math with it."

**Point out the Qty Needed inputs:**
> "These are editable — if the customer calls back and says they actually need 10 instead of 8, you just change the number and the system recalculates everything instantly."

**Point out the Available (Local) and Available (Company) columns:**
> "This is the part you can't do today without scrolling. The second you add a line item, the system tells you: 'You have 0 available locally in Midland, 0 available company-wide.' It's already checked every other job, every district, the machine shop — everything."

**Point out the Impact column:**
> "And right here — red means shortfall. Before you even save this work order, you can see: 'Adding this job puts me 8 short on wings, 6 short on masters, 1 short on Javelins.' Today you don't find that out until you scroll to the bottom of the spreadsheet and do the mental math."

**Scroll to Availability Impact section:**
> "And down here it summarizes the damage — this work order worsens 2 existing shortfalls and creates 3 new ones. It's telling you exactly what breaks and by how much."

**Pause and ask:**
> "Is this the kind of information you're looking for when you're evaluating whether to take a job? What's missing?"

### 2c. The Decision Point

> "So now you're looking at this and you have a choice. You can save it and let the system run the waterfall to figure out sourcing. You can save it as a draft if you need to call the customer back first. Or you can adjust the quantities — maybe you tell Vital Energy 'I can do 6 wings instead of 8, we'll convert some to 3-inch' — change the number right here, and the impact recalculates."

> "Today, that 'what if I change this?' conversation happens in your head while you're staring at the spreadsheet. Here it's instant."

---

## 3. Shortfall Resolution — "Robbing Peter to Pay Paul" (8–10 min)

### 3a. Transition

> "So let's say you accept the job. Now you've got shortfalls to solve. This is the part where you and Don spend hours — let me show you how the system handles it."

Click **SHORTFALL ALERTS** tab (the one with the red "4" badge).

### 3b. Walk Through the Shortfall Dashboard

> "Right now the system is tracking 4 open shortfalls across your fleet. Two are critical, two are in progress."

Point out the KPI cards at the top: 4 Open, 2 In Progress, 11 Resolved (30 days), -22 Total Units Short.

> "That '-22 total units short' number — today, you'd have to add up every negative number at the bottom of your spreadsheet to get that. Here it's one number, always current."

### 3c. Dive into the Wings Hyd 4 15 Shortfall

> "Let's look at the big one — Wings Hyd 4 15, short 6 units starting April 10. Here's the waterfall."

Walk down the waterfall table row by row:

> "Priority 1 — local owned in Midland. Exhausted, all 26 are deployed through April 22."
>
> "Priority 2 — local consignment. MAV has 4 deployed, 3E has 2 deployed. Nothing available."
>
> "Priority 3 — sub-rental locally. We can get 2 units from a rental company, 48-hour lead time."
>
> "Priority 4 — this is the one the system is recommending. Keithville has 12 owned surplus through April 20. If we pull 4, they still have 8 — and the system has already checked Keithville's schedule for the next 4 weeks to make sure pulling those 4 doesn't screw them."

**Point out the Schedule Check column:**
> "See this column? This is the part that takes you the most time today — 'If I pull from Keithville, does that create a problem for Keithville?' The system is doing that check automatically against the full 4-week rolling schedule. It's not just looking at today's surplus — it's looking at every job Keithville has coming up."

**Point out the recommendation box:**
> "And here's the bottom line: 'Transfer 4 owned from Keithville plus 2 sub-rental locally. Keithville retains 8 surplus through April 20. Estimated transfer cost: $2,850 scheduled freight.' You can approve it, modify it, or run a what-if."

**Ask Delmar:**
> "Is this how you think about it? Priority 1 through 6, exhaust local options before going cross-district? Is the order right?"

### 3d. Show the Masters Shortfall (Shop Insight)

Scroll to the second shortfall card — 7 15 SERVICE LESS MASTERS.

> "This one is different — masters are short company-wide. But look at this: 'Expediting 3 masters in Keithville shop (expected April 8) reduces shortfall to -5.' The machine shop queue feeds into the planning system. Today, the shop is a black hole — you know the count is down but you don't know when stuff is coming back. Here, the system can tell you 'if you push the shop to finish these 3 faster, you avoid a sub-rental.'"

> "And the capital flag at the bottom — serviceless masters have been at 95%+ utilization for 3 months. That's the system telling you it might be time to buy more, not just keep scrambling."

---

## 4. Quick Hits (5–7 min)

### 4a. Equipment Inventory — Component Detail

Click **Equipment Inventory** in the sidebar, then click **Wings Hyd 4 15** row.

> "This is the detail page for a single equipment type. Right now you'd have to scan the spreadsheet to figure out how many you have, where they are, who owns them. Here it's all in one place — 62 total, 44 owned, 14 consignment, 4 sub-rental, 97% utilization."

Scroll to the **14-Day Demand vs. Supply Forecast** table.

> "And this is the 14-day forecast for this specific equipment type. Day by day — demand, assets on hand, net position. You can see exactly when you go negative (April 10), how bad it gets (-6 for 9 days), and when it eases up (April 19). Today you'd have to calculate this in your head."

### 4b. Planning Board — Gantt View

Click **Planning Board**, make sure you're on the **Forecast Grid** tab.

> "This is your spreadsheet view — but structured. Every job, every district, 4-week rolling window. Click on a job to expand and see the equipment breakdown per line item, color-coded by ownership — green is owned, pink is consignment, checkered is sub-rental, red is shortfall. Same mental model you use today."

Click a job row to expand it.

> "BPX Permian Pad 7 — 8 equipment types, 46 units. You can see each line: 10 Wings Hyd 4 15 (owned), 6 Masters (owned), 4 Zippers (owned). The bottom rows show active job count and total units deployed per day."

### 4c. Mobile Check-In / Check-Out (if time allows)

Click **Check-Out / Check-In** in the sidebar.

> "Last thing — this is the mobile app your yard supervisors would use. When a loadout ships, they pull up the ticket on their phone, verify each line item — 10 wing valves, check, 6 masters, check — a second person confirms, and the system updates in real time. No handwritten delivery tickets, no 4-day lag into Basis."

> "And on check-in, if the quantity coming back matches what went out, it's a fast path — no serial number check needed. If there's a discrepancy — 9 came back instead of 10 — then the app forces a serial number drill-down to figure out what's missing."

---

## 5. Discussion (5–10 min)

Don't sell. Ask questions.

> "That's the core of it. Before we talk about building — a few questions:"

1. **"Did the waterfall priority order feel right? Local owned first, then consignment, then sub-rental, then cross-district?"** (This is the foundation of the entire system — if his mental model is different, we need to know now.)

2. **"When you're evaluating whether to take a new job, is there other information you'd want to see on that work order screen that wasn't there?"**

3. **"The schedule check on cross-district transfers — we're looking at the full 4-week window for the sending district. Is that far enough out, or do you sometimes need to think 6 or 8 weeks ahead?"**

4. **"For the mobile check-in/out — who would actually use it? Yard supervisors? DMs? Would your guys actually use a phone app, or is that a tough sell?"**

5. **"What's the first thing you'd want to see working? If we could build one piece of this and put it in front of you in 4–6 weeks, what would make the biggest dent in your day?"**

That last question is the most important. His answer tells you what Phase 1 should be.

---

## Things to Watch For

- **If he starts re-describing his spreadsheet in detail:** Let him. He's telling you what the system needs to match. Take notes.
- **If he says "yeah but what about...":** Write it down. These are requirements you missed.
- **If he asks about consignment reporting:** Show him you know about it (it's in the PRD) but it's not in the mockup yet. "That's Phase 2 — we wanted to nail the daily planning first."
- **If he asks about price:** Don't quote. Say "Let me put together a proposal based on what we land on today for Phase 1."
- **If he brings up Don (President):** Ask if Don should be in a future walkthrough. Don owns sales — his workflows are different.
- **If he seems overwhelmed:** Slow down. Focus on one screen. Ask "Is this too much at once?"

---

## After the Call

1. Send a follow-up email within 24 hours summarizing what he confirmed, what he corrected, and what new requirements surfaced
2. Update PRD-leonidas.md with any corrections
3. Update the mockup if he flagged anything visual
4. Draft Phase 1 scope based on his answer to "what would make the biggest dent"
