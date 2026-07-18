# AI Agent Configuration

This document explains how the core AI Agent (in `main-support-agent.json`) is configured to categorize leads, generate replies, and stay grounded in accurate company information.

## Model

- **Model:** OpenAI GPT-4o mini
- **Output:** Structured JSON, enforced via a Structured Output Parser node

## Output Format

The AI Agent always returns exactly four fields:

```json
{
  "reply": "Natural, professional customer-facing response",
  "category": "HOT_LEAD | NEW_LEAD | EXISTING_CLIENT | COMPLAINT | GENERAL_INQUIRY",
  "lead_score": "High | Medium | Low",
  "phone_number": "Extracted phone number, or empty string if none provided"
}
```

Enforcing structured output means every downstream node (Airtable logging, Switch routing, Calling Agent triggering) can reliably read these fields without extra parsing logic.

## Categorization Rules

| Category | Trigger Conditions |
|---|---|
| **Hot Lead** | Specific budget mentioned AND urgency AND clear buying intent |
| **New Lead** | Interested, asking about services/process/timeline, no budget yet |
| **Existing Client** | References a previous project or ongoing work |
| **Complaint** | Frustrated, reporting a problem, or threatening to leave |
| **General Inquiry** | Everything else — greetings, unrelated topics |

## Lead Scoring

- **High** — specific budget + urgency + buying intent
- **Medium** — engaged, asking detailed questions, no budget/urgency yet
- **Low** — casual greeting or general question, no buying signal

## Knowledge Base Grounding (RAG)

The AI Agent is instructed to **always call the `company_knowledge_base` tool** before answering any question involving pricing, cost, budget, timelines, services, or policies — rather than answering from memory or guessing. This prevents hallucinated pricing or outdated information from reaching customers. If the tool returns nothing relevant, the agent falls back to a safe default response asking for more project details.

## Memory / Conversation Context

Before generating a reply, the workflow checks Airtable for the sender's previous conversations. If history exists, the AI Agent is instructed to reference it naturally (rather than repeating an identical reply) so returning customers feel like they're talking to the same "person" each time.

## Design Principles

1. **Never guess on facts that matter** — pricing and policy questions are always grounded via the knowledge base tool.
2. **Structured output over free text** — every reply is accompanied by machine-readable metadata (category, score, phone number) so the rest of the pipeline can act on it automatically.
3. **Consistent persona** — the agent always identifies as "Alex," a support agent for the company, keeping tone professional and on-brand across every interaction.
