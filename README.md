# AI Lead Response Automation

> An automated system that instantly replies to every incoming lead and books a follow-up call within the hour for high-priority prospects — so no opportunity is ever missed.

---

## 🎥 Demo

*(Demo video will be added here)*

---

## The Problem This Solves

Most businesses lose potential customers not because their product is bad, but because:

- Leads email in and **no one replies for hours** — by then, the prospect has already gone to a competitor
- **High-value leads get buried** in a crowded inbox along with spam and casual inquiries
- Follow-up calls get **forgotten** because there's no system tracking who needs a call and when
- Support quality is **inconsistent** — some customers get fast, detailed replies; others get ignored

This system fixes all of that automatically, with zero manual effort after setup.

## What It Does

1. **Reads every incoming email** the moment it arrives
2. **Understands the message** using AI — figures out if it's a hot lead, a casual question, an existing client, or a complaint
3. **Replies instantly** with a relevant, professional response — pulling accurate pricing and service details from a knowledge base instead of guessing
4. **Flags high-priority leads** to the team instantly via Slack and email
5. **Automatically books a call** on the calendar and triggers an AI voice call for serious, high-budget leads — within the hour
6. **Logs everything** in a organized database (Airtable) so the team always has full visibility

## Why It Matters

| Without This System | With This System |
|---|---|
| Replies take hours (or never happen) | Replies happen in seconds |
| Hot leads mixed in with everything else | Hot leads flagged and prioritized automatically |
| Manual follow-up calls, easily forgotten | Calls booked and triggered automatically |
| No record of conversations | Every interaction logged and searchable |

---

## How It Works (Technical Overview)

This system is built using **n8n** (workflow automation), **OpenAI GPT-4o mini** (understanding + reply generation), **Airtable** (data + knowledge base), **Vapi + Twilio** (AI voice calling), and **Gmail/Slack/Trello** (communication channels).

It consists of three connected components:

### 1. Main Support Agent (`workflows/main-support-agent.json`)
Watches the inbox, categorizes every message (Hot Lead / New Lead / Existing Client / Complaint / General Inquiry), generates a reply using AI, and routes each category to the right action (client reply, team alert, CRM logging).

### 2. RAG Knowledge Base (`workflows/rag-knowledge-base.json`)
Whenever a customer asks about pricing, timelines, or services, the AI Agent queries this knowledge base instead of guessing — ensuring every answer is accurate and consistent.

### 3. Calling Agent (`workflows/calling-agent-vapi.json`)
For Hot Leads with a phone number, this checks calendar availability, books a slot, and triggers an AI-powered voice call (or sends a reschedule email if the slot is busy).
