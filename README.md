<!--
NOTE TO AUTHOR (delete before publishing):
While inspecting the exported JSON I found three wiring and configuration gaps. They are documented
accurately in the "Export Notes" block under "Limitations". Fix them in n8n, re-export, then remove that block.
1. "Facebook Message Webhook" has no outgoing connection and "Acknowledge Event" has no incoming connection.
2. The Facebook Graph API HTTP nodes (Seen, Typing, Send Response) carry no authentication in the export.
3. "Facebook Message Webhook" is exported with multipleMethods = true and an empty httpMethod list.
-->

<div align="center">

# 🤖 Smart AI Customer Support, Lead Generation & Human Handoff System

**An n8n workflow that answers Facebook Messenger customers from a business knowledge table, extracts and scores leads, and pauses the AI while a human takes over.**

<br>

![n8n](https://img.shields.io/badge/n8n-Workflow-EA4B71?style=for-the-badge&logo=n8n&logoColor=white)
![OpenAI](https://img.shields.io/badge/OpenAI-gpt--5--mini-412991?style=for-the-badge&logo=openai&logoColor=white)
![Messenger](https://img.shields.io/badge/Facebook-Messenger-0084FF?style=for-the-badge&logo=messenger&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-Code_Nodes-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black)
![Webhooks](https://img.shields.io/badge/Webhooks-Intake_%26_Handoff-2D3748?style=for-the-badge)
![Human in the loop](https://img.shields.io/badge/Human--in--the--Loop-Handoff-2EA44F?style=for-the-badge)

<br>

`AI Support` · `Lead Intelligence` · `Intent Routing` · `Human Handoff` · `Messenger Automation`

</div>

---

## 🧩 Project Snapshot

| 🧩 Area | ⚙️ Implementation | 🎯 Purpose |
|:--|:--|:--|
| **Automation** | n8n workflow, 24 nodes | Orchestrates the full support pipeline |
| **Channel** | Facebook Messenger via Graph API `v26.0` | Receives and answers customer messages |
| **AI** | n8n AI Agent + OpenAI `gpt-5-mini` | Reply, intent and lead extraction in one JSON object |
| **Knowledge** | n8n Data Table `messenger_business_data` | Grounds replies in active business rows |
| **Context** | Window buffer memory (50) + profile cache | Keeps conversation and customer state |
| **Lead Intelligence** | Score `0–100`, status, contact fields | Qualifies leads from conversation |
| **Routing** | JavaScript normalization + Switch node | Deterministic `human` / `sales` / `support` routes |
| **Human Handoff** | Timed AI pause + notification webhook | Escalates when a person is needed |
| **Status** | Export has `active: false` | Inactive until configured and activated |

---

## 🖼️ Workflow Overview

<img width="1256" height="534" alt="06 AI Customer Support, Lead Generation   Human Handoff System" src="https://github.com/user-attachments/assets/85cf0c50-0f8e-408f-858d-2b4d86ba5e21" />


The canvas has four zones: a webhook intake and verification area, a batching and validation area, an AI reasoning area (Agent with its model and memory), and a parse, persist, route and deliver area. The Switch node splits into human, sales, support and fallback outputs. Only the human output passes through the handoff notification.

---

## ✨ Core Capabilities

| 🧩 Capability | What It Does |
|:--|:--|
| 🔐 **Webhook verification** | Compares `hub.verify_token`, returns the challenge or a `403` |
| 🧹 **Message validation** | Accepts only events with message text that are not echoes |
| ♻️ **Duplicate protection** | Tracks message IDs for a 10-minute window |
| 🧵 **Message batching** | Waits 3 seconds, then combines a user's rapid messages |
| 🧠 **Conversation memory** | Window buffer of 50 messages per sender |
| 📚 **Knowledge grounding** | Injects active Data Table rows into the agent prompt |
| 🧾 **Structured AI output** | One JSON object with reply, intent, route, lead data |
| 👤 **Profile persistence** | Merges and upserts customer and lead fields |
| 📈 **Lead scoring** | Clamped 0–100 score with a configurable qualified threshold |
| 🤝 **Human handoff** | Timed AI pause plus a webhook notification |
| 📤 **Response delivery** | Cleans and length-limits text, then sends via Graph API |

---

## 🧠 How It Works

| # | Stage | System Behavior | Primary Component |
|:-:|:--|:--|:--|
| 1 | **Verify** | Validates the Meta verification request | `Is Token Valid?` |
| 2 | **Acknowledge** | Returns `EVENT_RECEIVED` with HTTP `200` | `Acknowledge Event` |
| 3 | **Validate** | Requires text and rejects echo events | `Filter Valid Messages` |
| 4 | **Guard and batch** | Checks handoff pause and duplicates, stores the message | `Store Message for Batching` |
| 5 | **Signal** | Sends `mark_seen`, then waits 3 seconds | `Send Seen Indicator`, `Wait 3 Seconds` |
| 6 | **Collect** | Combines the batch, attaches the cached profile | `Retrieve Batched Messages` |
| 7 | **Gate** | Continues only when `skip` is `false` | `Has Messages to Process?` |
| 8 | **Prepare** | Sends `typing_on`, loads active knowledge rows | `Send Typing Indicator`, `Load Business Knowledge Base` |
| 9 | **Reason** | Generates a reply and structured decision | `AI Agent` |
| 10 | **Normalize** | Parses JSON, fixes intent, route, score, status | `Parse AI Decision & Lead Data` |
| 11 | **Persist** | Upserts the customer and lead record | `Upsert Customer & Lead Profile` |
| 12 | **Route** | Branches on the normalized route | `Intent Router` |
| 13 | **Escalate** | Posts handoff context to the human webhook | `Notify Human Agent` |
| 14 | **Deliver** | Formats and sends the reply | `Format Response`, `Send Response to User` |

---

## 🏗️ Architecture Overview

| 🧱 Layer | 🎯 Responsibility | ⚙️ Implementation |
|:--|:--|:--|
| **Channel intake** | Receive Meta events | Two Webhook nodes on `facebook-messenger-webhook` |
| **Verification** | Validate the subscription handshake | `If` node, `respondToWebhook` nodes |
| **Message processing** | Filter, dedupe, batch | `If` and `Code` nodes, workflow static data |
| **Conversation context** | Multi-turn memory | Window Buffer Memory, 50 messages |
| **Business knowledge** | Supply factual context | Data Table `get`, filtered by `active = true` |
| **AI reasoning** | Reply and classify | AI Agent with `gpt-5-mini` |
| **Deterministic control** | Normalize AI output | JavaScript `Code` node |
| **Customer profile** | Persist lead state | Data Table `upsert` plus static-data cache |
| **Routing** | Select the branch | `Switch` node, four outputs |
| **Human handoff** | Notify and pause | HTTP Request plus static-data pause |
| **Delivery** | Send the reply | HTTP Request to Graph API `v26.0` |

---

## 💬 Facebook Messenger Integration

| 🔌 Element | Behavior in the Workflow |
|:--|:--|
| **Verification webhook** | Path `facebook-messenger-webhook`, responds via a Respond to Webhook node |
| **Token check** | `hub.verify_token` is compared with the placeholder `YOUR_VERIFY_TOKEN_HERE` |
| **Valid token** | Returns the `hub.challenge` value as text |
| **Invalid token** | Returns `Verification failed` with HTTP `403` |
| **Message webhook** | Same path, configured for message events |
| **Event acknowledgement** | Returns `EVENT_RECEIVED` with HTTP `200` |
| **Extracted fields** | Sender ID, page ID (recipient), message text, timestamp, message ID |
| **Echo filtering** | Drops events where `is_echo` is `true` |
| **Sender actions** | `mark_seen` and `typing_on` via `POST /v26.0/me/messages` |
| **Reply endpoint** | `POST /v26.0/me/messages`, `messaging_type: RESPONSE` |

---

## 🧵 Message Batching & Conversation Context

The session key is `senderId:pageId`. It scopes batches, handoff pauses, duplicate checks and the cached profile.

| 🧩 Mechanism | Behavior |
|:--|:--|
| **Duplicate tracking** | Message IDs are kept in static data and pruned after 10 minutes |
| **Batch storage** | Each message is appended to a per-session batch |
| **Batch window** | A 3-second Wait node precedes retrieval |
| **Combination** | Messages are sorted by timestamp and joined with spaces |
| **Batch consumption** | The batch is deleted once retrieved, so a later execution finds nothing and skips |
| **Profile attachment** | The cached profile is added to the item for the prompt |
| **Memory** | Window Buffer Memory keyed by sender ID, 50-message window |

This keeps a burst of short messages (for example "hi", "price?", "for 2 people") together as one AI turn instead of several. The workflow does not claim any latency figure.

---

## 🤖 AI Customer Support Agent

| 🧩 Element | Configuration |
|:--|:--|
| **Agent** | n8n AI Agent, prompt type `define`, input is the combined message |
| **Chat model** | OpenAI Chat Model, `gpt-5-mini` |
| **Memory** | Window Buffer Memory, 50 messages |
| **Tools** | None connected |
| **Context in prompt** | Active knowledge and config rows, existing customer profile |
| **Reply constraints** | Concise, professional, under 1900 characters |

**Required JSON output:**

| Field | Meaning |
|:--|:--|
| `response` | Customer-facing reply |
| `intent`, `route` | Classification and proposed route |
| `needs_human`, `handoff_reason` | Escalation decision and reason |
| `lead` | `name`, `email`, `phone`, `interest`, `budget`, `timeline` |
| `lead_score`, `lead_status` | Qualification signals |
| `knowledge_used`, `confidence` | Provenance and confidence |

### 🧭 AI reasoning vs. deterministic control

| 🧠 AI Reasoning | ⚙️ Deterministic Workflow Control |
|:--|:--|
| Writes the reply | Strips code fences and extracts the JSON object |
| Proposes intent, route and score | Validates intent against a fixed list |
| Extracts lead fields | Recomputes route, clamps score, merges profile |
| Suggests a handoff | Decides, persists and enforces the handoff pause |

The agent has no tools and cannot trigger workflow actions. Its output is treated as a proposal that the `Parse AI Decision & Lead Data` Code node normalizes before anything is routed or stored.

---

## 📚 Business Knowledge Base

The Data Table `messenger_business_data` holds knowledge, configuration and customer records.

| 🧩 Element | Behavior |
|:--|:--|
| **Loaded rows** | Rows where `active = true`, all returned |
| **Row format in prompt** | `[record_type] category or record_key: content` |
| **`[kb]` rows** | Business facts |
| **`[config]` rows** | Configuration values read by the Code node |
| **Empty table** | Prompt states no knowledge is configured and forbids invented facts |
| **Customer rows** | Written with `active = false`, so they stay out of the prompt |

Grounding is done by placing the active rows directly in the system prompt, with rules against inventing prices, policies, stock, delivery times, appointments or guarantees. It is prompt-context injection, not a vector search or retrieval pipeline.

---

## 👤 Customer & Lead Intelligence

| 🗂️ Profile Field | Source |
|:--|:--|
| `name`, `email`, `phone`, `interest`, `budget`, `timeline` | AI lead extraction, merged with the prior profile |
| `intent`, `route`, `lead_status` | Normalized by JavaScript |
| `lead_score` | Clamped and merged (see scoring) |
| `last_message`, `last_response` | Latest combined message and reply |
| `needs_human`, `handoff_reason` | Handoff decision |
| `updated_at` | ISO timestamp |

**Preservation:** every lead field uses "new value, otherwise previous value". A new message without a name or email does not erase what is already known.

**Two stores:** the profile is cached in workflow static data, which is what the next turn reads. It is also upserted into the Data Table, matched on `record_type = customer`, `user_id` and `page_id`. This is a lead and profile record, not a full CRM.

---

## 🧭 Intent Routing

| 🏷️ Intent | Normalized Route |
|:--|:--|
| `sales`, `product_inquiry`, `pricing`, `booking` | `sales` |
| `order`, `support`, `faq`, `spam`, `other` | `support` |
| `human_request`, `complaint`, `refund` | `human` |
| Any unknown value | Treated as `other`, then `support` |
| Any intent with `needs_human = true` or AI `route = human` | `human` |

| 🔧 Layer | Role |
|:--|:--|
| **1. AI classification** | Proposes `intent`, `route`, `needs_human` |
| **2. JavaScript normalization** | Applies the intent list and override rules above |
| **3. Switch-based routing** | Four outputs: `human`, `sales`, `support`, `fallback` |

Only `human` passes through `Notify Human Agent`. Sales, support and fallback go straight to `Format Response`.

---

## 📈 Lead Qualification & Scoring

| 📏 Aspect | Behavior |
|:--|:--|
| **Range** | 0–100, clamped |
| **Prompt factors** | Clear purchase intent, specific product or service interest, budget, timeline |
| **Prior score** | The higher of the new and previous score is kept |
| **Threshold** | `qualified_lead_score` config row, default `70`, clamped 0–100 |

**Status assignment, in order:**

1. `human_handoff` when a human is required
2. `qualified` when the merged score meets the threshold
3. `support` when the route is support
4. Otherwise `qualified` if previously qualified, else `new`

The score is a heuristic from the AI instructions. The workflow makes no claim that it predicts conversion or revenue.

---

## 🤝 Human Handoff

| 🧩 Element | Behavior |
|:--|:--|
| **Triggers** | AI sets `needs_human` or `route = human`, or intent is `human_request`, `complaint` or `refund` |
| **Prompt guidance** | Also covers refund or complaint decisions, payment, legal or security concerns, and missing business facts |
| **Pause window** | `handoff_pause_minutes` config row, default `30`, clamped 5–1440 |
| **AI pause** | While active, new messages are skipped at storage and at retrieval |
| **Expiry** | Expired pauses are removed on the next message |
| **Reset** | A later non-handoff turn clears the stored pause |
| **Notification** | POST to `human_handoff_webhook_url` from the config rows |
| **Payload** | Event `messenger_human_handoff`, IDs, intent, reason, lead data, message, AI reply |
| **Customer message** | AI-written reassurance, with a fixed fallback text if empty |

The workflow does not define who receives the notification or prove that a human team exists. That depends on the endpoint you configure.

---

## 📤 Response Delivery

| 🧩 Step | Behavior |
|:--|:--|
| **Empty reply, human needed** | `Thanks for your message. A human support representative will take over from here.` |
| **Empty reply, other** | `Thanks for your message. How can I help you today?` |
| **Length limit** | Over 1900 characters is cut to 1897 plus `...` |
| **Markdown cleanup** | Removes bold, italic, backticks and code fences |
| **Delivery** | `POST /v26.0/me/messages`, recipient = sender ID, `messaging_type: RESPONSE` |
| **Final node** | `Success`, an empty Set node |

---

## 🗄️ Data & Persistence

| 💾 Store | Contents | Used For |
|:--|:--|:--|
| **n8n Data Table** `messenger_business_data` | Knowledge, config and customer rows | Grounding, settings, profile records |
| **Workflow static data** | Message batches, recent message IDs, handoff sessions, profile cache | Batching, dedupe, pause, fast profile reads |
| **Window Buffer Memory** | Last 50 messages per sender | Conversation context for the agent |

Customer rows are written to the Data Table but are not read back by the workflow. The static-data cache supplies the profile on later turns.

---

## 🛡️ Safety & Reliability

**Implemented in the workflow:**

| 🛡️ Control | Mechanism |
|:--|:--|
| Verification handshake | Token compare, `403` on mismatch |
| Invalid input filtering | Requires text, drops echoes |
| Duplicate suppression | Message ID window of 10 minutes |
| Handoff pause | Static-data session with expiry |
| AI JSON fallback | Invalid output becomes a human-handoff response |
| Knowledge-only policy | Prompt forbids invented business facts |
| Intent normalization | Fixed intent list, unknown becomes `other` |
| Score and confidence guards | Score clamped 0–100, confidence clamped 0–1 |
| Reply length guard | 1900-character cap |
| Non-blocking errors | Several nodes use continue-on-error |

**General deployment practices (not implemented in the JSON):** request signature validation, authenticated handoff endpoint, credential rotation, execution-log review.

---

## 🧱 Key Workflow Components

<details>
<summary><b>Full node inventory (24 nodes)</b></summary>

<br>

| 🧩 Component | ⚙️ n8n Type | 🎯 Responsibility |
|:--|:--|:--|
| Facebook Verification Webhook | Webhook | Receives the Meta verification request |
| Facebook Message Webhook | Webhook | Receives message events |
| Is Token Valid? | If | Compares the verify token |
| Respond with Challenge | Respond to Webhook | Returns `hub.challenge` |
| Respond Forbidden | Respond to Webhook | Returns `403` |
| Acknowledge Event | Respond to Webhook | Returns `EVENT_RECEIVED` |
| Filter Valid Messages | If | Requires text, excludes echoes |
| Store Message for Batching | Code | Handoff check, dedupe, batch append |
| Send Seen Indicator | HTTP Request | `mark_seen` sender action |
| Wait 3 Seconds | Wait | Batching window |
| Retrieve Batched Messages | Code | Combines messages, attaches profile |
| Has Messages to Process? | If | Skips empty or paused turns |
| Send Typing Indicator | HTTP Request | `typing_on` sender action |
| Load Business Knowledge Base | Data Table | Loads active rows |
| Conversation Memory | Window Buffer Memory | 50-message context |
| OpenAI Chat Model | OpenAI Chat Model | `gpt-5-mini` |
| AI Agent | AI Agent | Reply and structured decision |
| Parse AI Decision & Lead Data | Code | Parse, normalize, merge, set pause |
| Upsert Customer & Lead Profile | Data Table | Persists the profile row |
| Intent Router | Switch | `human` / `sales` / `support` / fallback |
| Notify Human Agent | HTTP Request | Posts the handoff payload |
| Format Response | Code | Fallbacks, cleanup, length cap |
| Send Response to User | HTTP Request | Sends the Messenger reply |
| Success | Set | Terminal node |

</details>

---

## ⚙️ Configuration

<details>
<summary><b>Configuration reference</b></summary>

<br>

| ⚙️ Setting | Location | Notes |
|:--|:--|:--|
| Verify token | `Is Token Valid?` | Placeholder `YOUR_VERIFY_TOKEN_HERE`, replace it |
| OpenAI credential | `OpenAI Chat Model` | Configure inside n8n, no secret in the export |
| Facebook access | Three Graph API HTTP nodes | No authentication in the export, add it in n8n |
| Data Table | `messenger_business_data` | Must exist before running |
| `business_name` | `[config]` row | Included in the handoff payload |
| `qualified_lead_score` | `[config]` row | Default `70` |
| `handoff_pause_minutes` | `[config]` row | Default `30`, range 5–1440 |
| `human_handoff_webhook_url` | `[config]` row | Required for human notifications |

**Data Table columns used:** `record_type`, `record_key`, `category`, `content`, `active`, `user_id`, `page_id`, `customer_name`, `email`, `phone`, `interest`, `budget`, `timeline`, `intent`, `route`, `lead_score`, `lead_status`, `last_message`, `last_response`, `needs_human`, `handoff_reason`, `updated_at`.

Config rows are read as `category` (or `record_key`) for the key and `content` for the value.

</details>

---

## 🚀 Setup & Import

1. **Import** the workflow JSON into n8n.
2. **Create** the Data Table `messenger_business_data` with the columns above.
3. **Add** `[kb]` knowledge rows and `[config]` rows, all with `active = true`.
4. **Configure** an OpenAI credential on `OpenAI Chat Model`.
5. **Add** Facebook Page access authentication to the three Graph API HTTP nodes.
6. **Replace** `YOUR_VERIFY_TOKEN_HERE` with your own token.
7. **Set** `human_handoff_webhook_url` to your notification endpoint.
8. **Check** the message webhook accepts POST and is connected to `Acknowledge Event` (see Export Notes).
9. **Register** the n8n webhook URL in your Meta app and test verification.
10. **Send** test messages through each scenario below.
11. **Activate** the workflow deliberately. The export is inactive.

Dependencies: an n8n instance with Data Tables and AI Agent nodes, an OpenAI account, a Meta app with a Facebook Page, and an optional human notification endpoint.

---

## 🧪 Validation Scenarios

> These are **expected test scenarios**, not recorded execution results.

<details>
<summary><b>Validation matrix</b></summary>

<br>

| 🧪 Scenario | Expected Behavior |
|:--|:--|
| Invalid verify token | `403 Verification failed` |
| Valid verify token | Challenge value returned |
| Message without text | Filtered out |
| Echo event | Filtered out |
| Duplicate message ID | Skipped |
| Rapid consecutive messages | Combined into one AI turn |
| Support question in knowledge base | `support` route, grounded reply |
| Pricing or product inquiry | `sales` route |
| Sales intent with budget and timeline | Higher score, possible `qualified` |
| Explicit human request | `human` route, notification, AI paused |
| Complaint or refund | Forced `human` route |
| Fact missing from knowledge | Prompt directs a human handoff |
| Malformed AI JSON | Fallback reply, human route |
| Message during active handoff | Skipped without an AI reply |
| Returning customer | Known fields preserved |
| Reply delivery | Cleaned text sent via Graph API |

</details>

---

## 🧯 Failure Handling

| ⚠️ Situation | Behavior |
|:--|:--|
| Invalid verification token | `403` response |
| Missing text or echo event | Execution ends silently |
| Duplicate delivery | Marked `skip`, ends at the gate |
| Invalid AI JSON | Safe fallback with human handoff |
| Empty AI reply | Fixed fallback text |
| No business data | Prompt forbids invented facts |
| Handoff required | Pause set, notification sent |
| Provider errors | Seen, typing, send, notify, load and upsert nodes continue on error |
| Blank handoff URL | Notification request has no target, and the error is not blocking |

---

## 🔐 Security & Secret Handling

- Keep OpenAI and Facebook credentials in **n8n credential storage**, never in the README or repository.
- Replace the verification placeholder with a private token and do not commit it.
- Protect webhook endpoints with the authentication options available to you.
- Review the handoff endpoint URL before deployment.
- The export contains no real secrets, and no security certification is claimed.

---


---

## 🏁 Final Takeaway

This project shows an AI-assisted Messenger support flow built from clearly separated concerns. Replies are grounded in business knowledge held in an n8n Data Table, with conversation memory and a cached customer profile supplying context. The AI returns structured JSON, but JavaScript parses, validates and normalizes it before routing, so intent, score and handoff decisions stay deterministic. Lead data is merged without losing earlier details and persisted to a Data Table. Human escalation sends context to a webhook and pauses the AI for a configurable window. The result is a readable example of n8n-based orchestration across Facebook Messenger, OpenAI, Data Tables and code-driven safeguards.

<div align="center">

`AI Support` · `Lead Intelligence` · `Human Handoff` · `Messenger Automation`

</div>
