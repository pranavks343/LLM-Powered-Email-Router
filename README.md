# LLM-Powered Email Router

An [n8n](https://n8n.io) workflow that watches a Gmail inbox, classifies each incoming email with an LLM, labels it, and routes it to the right action: an AI-written reply, a draft, a team notification, or quiet archiving.

## How it works

```
Gmail Trigger (polls every minute)
        │
        ▼
Text Classifier (GPT-5 mini)
        │
        ├─ customer support ─► label ─► AI Agent ─► send reply
        ├─ finance/billing ──► label ─► notify billing team by email
        ├─ high priority ────► label ─► AI Agent1 (finance prompt) ─► create draft reply
        └─ promotion ────────► label ─► mark as read
```

| Category | Detected by | Action |
|---|---|---|
| `customer support` | Help requests, errors, access issues | Labels the email, then an AI agent **sends** a reply in the thread |
| `finance/billing` | Invoices, payments, refunds, subscriptions | Labels the email and emails the billing team a summary |
| `high priority` | Urgent or outage-related emails | Labels the email, then an AI agent writes a reply and saves it as a **draft** |
| `promotion` | Marketing and sales emails | Labels the email and marks it as read |

The classifier reads the subject and the Gmail snippet (the first part of the body), not the full message.

> **Warning:** the `customer support` branch sends replies automatically with no human review. Test it on a separate inbox first, or swap the "Reply to a message" node for "Create a draft".

## Requirements

- n8n (cloud or self-hosted) with the Gmail and LangChain nodes
- A Google account with Gmail OAuth2 set up in n8n
- An OpenAI API key

## Setup

1. In n8n, go to **Workflows → ⋯ → Import from File** and select `workflow.json`. Alternatively, use **Import from URL** with the raw file link.
2. Add your credentials and attach them to the nodes. They were removed from this export:
   - **Gmail OAuth2** on the Gmail Trigger and every Gmail node
   - **OpenAI API** on the "GPT 5 mini" and "OpenAI Chat Model" nodes
3. Create four Gmail labels, one per category, then open each **Add label to message** node (four in total) and select the matching label.
4. Edit the placeholder addresses:
   - **Send a message** node: change `billing@example.com` to your billing team's address.
   - Both **AI Agent** system prompts: change `support@example.com` to the address customers should contact when the bot can't answer.
5. Optional: customise the agents' system prompts (name, company, tone, sign-off) and the category descriptions in the Text Classifier.
6. Run once manually with a test email, then toggle the workflow to **Active**.

## Customisation

- **Model:** swap `gpt-5-mini` for another chat model in the two model nodes.
- **Categories:** add or edit categories in the Text Classifier, then wire a new branch for each.
- **Polling interval:** change `pollTimes` on the Gmail Trigger (every minute by default).
- **Reply behaviour:** the agents only answer from their prompt, with no knowledge base. Add a tool or vector store node to the agent if you need real product or policy facts.

## Notes

- Tested on a recent n8n 1.x release. Node versions are pinned in `workflow.json`.
- The workflow ships inactive. No secrets, credential IDs or sample data are included.

## License

MIT. See [LICENSE](LICENSE).
