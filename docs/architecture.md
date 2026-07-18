# System Architecture

## Overview

This system consists of three independent n8n workflows that work together as one connected pipeline. Each workflow can be understood, tested, and modified independently, but they share data through Airtable and Execute Workflow calls.

## Components

| Component | File | Role |
|---|---|---|
| Main Support Agent | `workflows/main-support-agent.json` | Entry point — reads emails, categorizes them, replies, routes |
| RAG Knowledge Base | `workflows/rag-knowledge-base.json` | Provides accurate pricing/service answers via retrieval-augmented generation |
| Calling Agent | `workflows/calling-agent-vapi.json` | Books calendar slots and triggers AI voice calls for hot leads |

## Data Flow


┌─────────────────┐
│  Gmail Trigger    │  New email arrives
└────────┬─────────┘
↓
┌─────────────────┐
│  Build Memory     │  Pulls prior conversation history (if any) from Airtable
│  Context           │
└────────┬─────────┘
↓
┌─────────────────┐
│  AI Agent          │  Categorizes message + generates reply
│  (GPT-4o mini)     │◄──── Queries RAG Knowledge Base for pricing/service questions
└────────┬─────────┘
↓
┌─────────────────┐
│  Create Record     │  Logs conversation in Airtable (Conversations table)
│  (Airtable)        │
└────────┬─────────┘
↓
┌─────────────────┐
│  Switch Node       │  Routes based on category
└────────┬─────────┘
↓
┌─────┴─────┬─────────┬───────────┬─────────────┐
↓           ↓         ↓           ↓             ↓
Hot Lead   New Lead  Existing   Complaint    General
│           │      Client        │         Inquiry
│           │         │          │             │
↓           ↓         ↓          ↓             ↓
Reply +    Reply +   Reply +    Escalate      Reply
Alert +    Trello    Trello     to team       in-thread
Check                Card
Phone
│
↓ (if phone number present)
┌─────────────────┐
│  Calling Agent     │  Execute Workflow call
│  (Sub-workflow)    │
└────────┬─────────┘
↓
┌─────────────────┐
│  Check Calendar    │
│  Availability      │
└────────┬─────────┘
↓
┌─────┴─────┐
↓           ↓
Slot Free   Slot Busy
│           │
↓           ↓
Book Call   Send Reschedule

Update    Email + Update
Airtable    Airtable
Trigger
Vapi Call


## Categorization Logic

Every incoming message is classified into exactly one category:

- **Hot Lead** — mentions a specific budget AND urgency AND clear buying intent
- **New Lead** — interested and exploring, no budget/urgency yet
- **Existing Client** — references a previous project or ongoing work
- **Complaint** — frustrated, reporting a problem, or threatening to leave
- **General Inquiry** — everything else (greetings, unrelated questions)

Each message also receives a **Lead Score** (High / Medium / Low) to help the team prioritize follow-up.

## Data Storage (Airtable)

All conversations are logged in a single **"Conversations"** table, which acts as a lightweight CRM. Key fields include:

- Customer Name, Channel, Subject, Message, AI Reply
- Category, Lead Score, Phone Number, Status
- Call Status, Scheduled Call Time, Calendar Event ID, Call Summary, Deal Outcome (populated later by the Calling Agent)

This means the team always has a single source of truth for every customer interaction, whether it was handled purely by email or escalated to a phone call.

## External Services Used

| Service | Purpose |
|---|---|
| n8n | Workflow orchestration |
| OpenAI (GPT-4o mini) | Message understanding, categorization, reply generation |
| Airtable | CRM database + knowledge base storage |
| Gmail | Reading incoming leads, sending replies |
| Slack | Real-time team alerts for hot leads and complaints |
| Trello | Lead tracking boards |
| Google Calendar | Checking availability, booking calls |
| Vapi + Twilio | AI-powered outbound voice calls |
