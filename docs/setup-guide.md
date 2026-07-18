# Setup Guide

This guide explains how to reproduce this system from scratch using n8n and the connected services.

## Prerequisites

- An [n8n](https://n8n.io) instance (cloud or self-hosted)
- A Gmail account (for reading leads and sending replies)
- An [OpenAI API key](https://platform.openai.com) (GPT-4o mini access)
- An [Airtable](https://airtable.com) account
- A [Slack](https://slack.com) workspace (for team alerts)
- A [Trello](https://trello.com) account (for lead tracking boards)
- A Google account (for Calendar integration)
- A [Vapi](https://vapi.ai) account + [Twilio](https://twilio.com) account (for AI voice calls)

## Step 1: Airtable Setup

Create a base named **"AI Support Agent"** with a table called **"Conversations"** containing these fields:

| Field Name | Type |
|---|---|
| Customer Name | Single line text |
| Channel | Single line text |
| Subject | Single line text |
| Message | Long text |
| AI Reply | Long text |
| Category | Single select (Hot Lead, New Lead, Existing Client, Complaint, General Inquiry) |
| Lead Score | Single select (High, Medium, Low) |
| Phone Number | Single line text |
| Status | Single select (Open, Closed) |
| Date/Time | Date |
| Timezone | Single line text |
| Call Status | Single select — **add predefined options** (Scheduled, Rescheduled - Awaiting Response) to avoid typecast errors |
| Scheduled Call Time | Date |
| Calendar Event ID | Single line text |
| Call Attempt Count | Number |
| Call Summary | Long text |
| Deal Outcome | Single select |

Also create a second table for your **knowledge base** (pricing, services, FAQs) that the RAG workflow will use.

## Step 2: Import the Workflows

1. In n8n, go to **Workflows → Add Workflow → Import from File**
2. Import each file from the `workflows/` folder:
   - `main-support-agent.json`
   - `rag-knowledge-base.json`
   - `calling-agent-vapi.json`

## Step 3: Connect Credentials

For each workflow, reconnect the credentials to your own accounts:

- Gmail (OAuth2)
- OpenAI API key
- Airtable Personal Access Token
- Slack OAuth
- Trello API key/token
- Google Calendar OAuth2
- Vapi (Header Auth with your private key)

## Step 4: Configure the RAG Knowledge Base

Populate your Airtable knowledge base table with your company's actual pricing, services, and policies. The Main Support Agent will query this automatically whenever a customer asks about cost, timelines, or services — see `docs/ai-agent-prompt.md` for how this is enforced in the prompt.

## Step 5: Configure the Calling Agent

1. Create a Vapi Assistant and note its **Assistant ID**
2. Set up a Twilio phone number and import it into Vapi (note the **Phone Number ID**)
3. Update the HTTP Request node in `calling-agent-vapi.json` with your own Assistant ID and Phone Number ID
4. **Important:** Twilio trial accounts cannot call unverified/international numbers. A paid Twilio plan (Pay-as-you-go) is required for real outbound calls.

## Step 6: Test End-to-End

1. Send a test email that includes a budget, urgency, and a phone number (e.g. "I need an app built urgently, budget is $5000, call me at +1234567890")
2. Confirm:
   - The email is categorized as **Hot Lead**
   - A reply is sent back
   - A Slack/Gmail alert is sent to the team
   - An Airtable record is created
   - A Google Calendar event is booked
   - A Vapi call is triggered (requires a paid Twilio plan)

## Notes

- The **"Create a record"** Airtable node must remain a sequential ancestor of the Switch node (not parallel), otherwise downstream nodes referencing it will throw "no path back to node" errors.
- Enable **"Always Output Data"** on the Google Calendar "Get many events" node — otherwise an empty calendar will silently halt the workflow.
