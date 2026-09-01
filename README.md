# AI Post-Call Sales Automation

A system that turns raw, unstructured sales call notes into structured intelligence, scores the deal, and decides what should happen next, without ever letting the AI make that final call alone.

## The Problem

A sales call can go perfectly and still lose momentum the moment it ends. The rep moves on to the next call. The CRM doesn't update itself. Nobody remembers to send the proposal. A follow-up that was promised on the call quietly gets forgotten. And somewhere in all of that, the specific details the client actually said start disappearing, replaced by whatever the rep vaguely remembers a day later.

*Quick context for anyone unfamiliar with the tools mentioned below: a CRM (Customer Relationship Management system) is where a business keeps track of every customer and where that customer stands in the sales process, for example GoHighLevel. n8n and Zapier are automation platforms that connect different apps together so information moves between them automatically instead of someone copying and pasting it by hand.*

## What This Does

The moment call notes come in (captured through Granola and synced via Zapier), the system reads them and separates what was actually said from what was assumed or inferred. Budget the client explicitly stated gets captured as a fact. A number the rep guessed at gets marked as an interpretation. That distinction carries through the entire system.

From there, a structured scoring layer calculates how ready the deal is for a proposal and how healthy it looks overall, based on hard criteria, not AI opinion. A second AI step looks at those scores and recommends a next action. But that recommendation isn't final. A rules layer checks it against explicit thresholds before anything executes, and if the numbers don't support the AI's recommendation, it overrides it and routes the deal somewhere safer instead.

Depending on what the system decides, one of five things happens: a proposal gets drafted using the client's own words and sent for review, a request goes out flagging exactly what information is still missing, a follow-up reminder gets sent, an alert goes out that another call is needed, or the deal gets flagged for human review because something didn't add up. At the same time, the CRM updates itself: the contact record, the pipeline stage, and the relevant tags, all without anyone touching it manually.

## Architecture

```
Call Ends (Granola)
        ↓
Notes Synced to Sheet (Zapier)
        ↓
n8n Trigger
        ↓
AI 1 — Extract Call Intelligence
   (fact vs assumption, with evidence)
        ↓
Deterministic Scoring
   (Proposal Readiness + Deal Health)
        ↓
AI 2 — Recommend Next Action
        ↓
Rules Guard
   (validates or overrides the AI's recommendation)
        ↓
Execution Router
   ┌────────┬───────────┬──────────┬──────────────┬──────────────┐
   ↓        ↓           ↓          ↓              ↓
Proposal  Missing    Follow-Up  Schedule      Human Review
Generated  Info Alert  Reminder  Next Call     Flagged
   ↓
GoHighLevel
   (contact updated, tagged, pipeline stage moved)
```

*A "pipeline stage" is just where a deal currently sits in the sales process, for example "New Lead," "Proposal Sent," or "Won." A "tag" is a small label attached to a contact so the team can filter and find them later, for example "needs-info" or "proposal-ready."*

## Why the AI Doesn't Get the Final Say

Early versions of this system let a single AI call read the notes and directly decide what to do next. That worked most of the time, which was exactly the problem. When it was wrong, it was wrong confidently, and there was no check in place to catch it.

The current version splits the job into two AI steps with a deterministic layer in between. The first AI only extracts facts and is explicitly told not to make any decisions. A scoring step then calculates readiness and deal health from clear boolean conditions, not from the AI's judgment. The second AI recommends a next action based on those scores. Before that recommendation is allowed to execute anything, a rules layer checks it against fixed thresholds. If the AI recommends sending a proposal but the budget was never confirmed by the client, the rules layer overrides it and routes the deal to request missing information instead.

This means the system can be wrong about what to recommend, but it can't act on a recommendation that doesn't meet the actual conditions.

## Fact vs Assumption

Every extracted value for budget and timeline is classified as one of three things: explicitly stated by the client, interpreted or guessed by the rep, or not mentioned at all. Each classification comes with the exact evidence text that supports it. A number the client said out loud carries very different weight than a number the rep inferred from tone, and the system is built to never blur that line.

## Example Output

```
Client: John Smith, ABC Properties
Budget: $7,000/month — CLIENT_STATED
  Evidence: "We're spending around $7,000 a month right now"
Timeline: October — CLIENT_STATED
  Evidence: "We'd like to switch providers in October"
Decision Maker Present: Yes
Unresolved Objection: No

Proposal Readiness: 65/100
Deal Health: 50/100

AI Recommended: SEND_PROPOSAL
Rules Guard Result: OVERRIDDEN
Final Action: REQUEST_MISSING_INFORMATION
Reason: Decision maker confirmation and full scope were not
        sufficiently established to meet the proposal threshold.
```

## Tech Stack

**Granola** — an app that records sales calls and turns the conversation into written notes automatically, so the rep doesn't have to type everything by hand during the call.

**Zapier** — connects Granola to a tracked spreadsheet, so as soon as a call's notes are ready, they land somewhere the automation can pick them up.

**n8n** — the automation platform that runs the actual workflow: receiving the notes, calling the AI, checking the rules, and deciding what happens next.

**GPT** — the AI model used for reading the call notes and reasoning about what was said and what should happen next.

**PDFShift** — converts the generated proposal from a webpage-style document into an actual downloadable PDF.

**GoHighLevel** — the CRM where the client's information lives, and where the deal gets tagged, staged, and tracked going forward.

## What's Different About This One

Most AI automations in this space either do everything through a single AI call, or they skip the scoring step entirely and just let the model decide. This system treats the AI as a component with a specific, limited job at each stage, extraction, then recommendation, never both at once, and never without a rules layer standing between its recommendation and any real action. The salesperson still owns the deal. The system just makes sure nothing about it gets forgotten.
