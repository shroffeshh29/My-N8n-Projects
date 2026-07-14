# 🤖 AI Test Case Generator (N8N + Groq)

> A low-code automation that turns user stories into structured QA test cases using **n8n**, **Groq AI**, and a lightweight HTML frontend.

---

## 📸 What It Does

Paste a user story like:

> *"As a user, I want to reset my password via email so that I can regain access to my account if I forget it."*

...and instantly get back:

- ✅ **Positive test cases** – happy path scenarios
- ❌ **Negative test cases** – invalid input / error handling
- ⚠️ **Edge cases** – boundary & stress scenarios

All returned as structured JSON with **ID, Title, Type, Priority, Steps, and Expected Results**.

---

## 🏗️ Architecture

```
┌──────────────┐      POST (userStory + ticketId)      ┌──────────────┐
│   Frontend   │ ─────────────────────────────────────>│  N8N Webhook │
│  (index.html)│                                      │   (n8n.io)   │
└──────────────┘                                      └──────────────┘
                                                               │
                                                               ▼
                                                 ┌──────────────────────┐
                                                 │  Groq AI API Call  │
                                                 │ (llama-3.3-70b)    │
                                                 └──────────────────────┘
                                                               │
                                                               ▼
                                                 ┌──────────────────────┐
                                                 │  JSON Response       │
                                                 │  (testCases[])       │
                                                 └──────────────────────┘
                                                               │
                                                  ┌────────────┴────────────┐
                                                  ▼                         ▼
                                         ┌──────────────┐        ┌──────────────────┐
                                         │  Frontend    │        │ Slack Notify     │
                                         │  (display)   │        │ (optional)       │
                                         └──────────────┘        └──────────────────┘
```

### Components

| Component | Technology | Role |
|-----------|-----------|------|
| **Workflow Engine** | n8n (self-hosted / cloud) | Orchestrates the automation |
| **AI Model** | Groq (`llama-3.3-70b-versatile`) | Generates test cases via LLM |
| **Frontend** | Vanilla HTML + CSS + JS | User interface |
| **Notifications** | Slack Webhooks (optional) | Team alerts |

---

## 🚀 Quick Start

### Prerequisites

- [n8n](https://n8n.io/) installed (local, Docker, or cloud)
- A [Groq](https://console.groq.com/) API key (free tier available)
- (Optional) A Slack Incoming Webhook URL

### Step 1: Clone This Repo

```bash
git clone https://github.com/shroffeshh29/My-N8n-Projects.git
cd My-N8n-Projects
```

### Step 2: Configure Environment Variables

**⚠️ CRITICAL: Never commit credentials to Git.**

1. Copy the example environment file:

```bash
cp .env.example .env
```

2. Open `.env` and fill in your real values:

```env
# Your Groq API Key (from https://console.groq.com/keys)
GROQ_API_KEY=your_groq_api_key_here

# Your n8n webhook URL after deployment
N8N_WEBHOOK_URL=https://your-n8n-instance.up.railway.app/webhook/generate-test-cases

# Slack Webhook (optional) — get from https://api.slack.com/messaging/webhooks
SLACK_WEBHOOK_URL=your_slack_webhook_url_here
```

3. **Keep `.env` private** — it is already ignored by `.gitignore`.

### Step 3: Import the n8n Workflow

1. Open your n8n Editor.
2. Go to **Workflows → Import from File**.
3. Select `n8n-workflow.json` from this repo.
4. The workflow will appear with these nodes:
   - **Webhook** – listens for POST requests
   - **Normalize Input** – extracts `userStory` and `ticketId`
   - **Groq - Generate Test Cases** – calls Groq AI API
   - **Parse Groq Response** – cleans and parses LLM output
   - **Respond to Webhook** – returns JSON to frontend
   - **Slack Notify (optional)** – sends team notification

### Step 4: Add Credentials Inside n8n (Securely)

| Credential | Where to Set |
|------------|-------------|
| **Groq API Key** | n8n → Settings → Credentials → HTTP Header Auth → Name: `Authorization`, Value: `Bearer YOUR_GROQ_API_KEY` |
| **Slack Webhook** | n8n → Workflow → "Slack Notify" node → set URL from environment or credential store |

> **Why this way?** n8n encrypts credentials in its database. When you share the `n8n-workflow.json`, credentials are **not exported** — only the workflow structure is. Others can import the workflow but must supply their own API keys.

### Step 5: Activate the Workflow

1. Click **Activate** in n8n.
2. Copy the webhook URL (e.g., `https://.../webhook/generate-test-cases`).

### Step 6: Configure the Frontend

1. Open `frontend/index.html` in any code editor.
2. Update this line with your actual webhook URL:

```javascript
const N8N_WEBHOOK_URL = "https://your-n8n-instance.up.railway.app/webhook/generate-test-cases";
```

3. Open `frontend/index.html` in a browser (double-click or use Live Server).
4. Paste a user story and click **Generate test cases**.

---

## 🔒 Security & Credential Handling

### For You (Repository Owner)

- Your credentials live in:
  - `.env` file (ignored by Git)
  - n8n's internal encrypted credential store
- The exported `n8n-workflow.json` **does not contain** your API keys.
- The frontend uses a webhook URL — if you deploy publicly, consider adding basic auth or a token gate.

### For Others (Forkers / Contributors)

When someone else clones this repo:

- ❌ They **cannot** see your `.env` file — it is git-ignored.
- ❌ They **cannot** see your Groq API key — it lives in your n8n instance.
- ❌ They **cannot** see your Slack webhook — also in your n8n instance.
- ✅ They get the **full workflow logic** and can run it using **their own** API keys.

**Best Practices:**
- Rotate API keys if you ever accidentally commit one.
- Use n8n's built-in **Credential Store** instead of hardcoding secrets in workflow JSON.
- For production, add an **API Key gate** before the webhook node.

---

## 📁 Project Structure

```
My-N8n-Projects/
├── .env.example          # Template for environment variables
├── .gitignore            # Ensures .env and secrets never get committed
├── n8n-workflow.json     # Exportable n8n workflow (no credentials inside)
├── frontend/
│   └── index.html        # Standalone UI — just open in browser
└── README.md             # This file
```

---

## 🎯 Why This Matters for Your N8N Journey

| Skill You Learn | How This Project Helps |
|-----------------|------------------------|
| **Webhook Configuration** | Understand request/response nodes |
| **Credential Management** | Learn secure secret handling in n8n |
| **HTTP Request Nodes** | Call external AI APIs (Groq, OpenAI, etc.) |
| **Code Nodes (JS)** | Parse and sanitize LLM JSON responses |
| **Data Normalization** | Use Set nodes to shape incoming payloads |
| **Error Handling** | Graceful fallbacks when AI returns bad JSON |
| **Integration Patterns** | Connect frontend → automation → notification |

This is a **stepping stone** to bigger automations:
- Auto-generate JIRA tickets from test cases
- Trigger CI/CD pipelines based on test case output
- Build a full QA assistant with memory (vector DB + chat)

---

## ⚠️ Why This Won't Work "As-Is" at Enterprise Level

This project is designed for **learning, prototyping, and personal workflows**. Here's what enterprises require that this repo does **not** currently provide:

| Enterprise Requirement | Current Gap | Solution Path |
|----------------------|-------------|---------------|
| **Authentication & Authorization** | Webhook is open — anyone with the URL can hit it | Add OAuth2, API key middleware, or IP whitelisting |
| **Rate Limiting** | No throttling on Groq API calls | Add n8n **Wait** nodes or external API gateway |
| **Input Validation & Sanitization** | Minimal validation on user story input | Add schema validation (e.g., Zod, JSON Schema) |
| **Audit Logging** | No persistent logs of who generated what | Connect to DB / SIEM (Splunk, Datadog) |
| **Data Privacy (PII)** | User stories may contain sensitive data | Add PII detection node + data masking |
| **High Availability** | Single n8n instance, no queue mode | Deploy n8n in **Queue Mode** with Redis + Postgres |
| **Secret Rotation** | Manual credential updates | Integrate with HashiCorp Vault / AWS Secrets Manager |
| **Testing & CI/CD** | No unit tests for JS code nodes | Add Jest tests + GitHub Actions pipeline |
| **SLA Monitoring** | No alerting if Groq or n8n is down | Add health-check webhooks + PagerDuty |
| **Model Governance** | No prompt versioning or approval | Use Promptlayer / Weights & Biases for prompt management |

> **The goal is not to discard this project — it is to understand which layer to add next as you scale.**

---

## 🛠️ Customization Ideas

| Idea | How To |
|------|--------|
| Change AI model | Edit the Groq node → change `"model": "llama-3.3-70b-versatile"` to `"mixtral-8x7b-32768"` |
| Add more test types | Update the system prompt in the Groq node JSON |
| Save to Google Sheets | Add a **Google Sheets** node after "Parse Groq Response" |
| Email results | Add an **Email (SMTP)** node or integrate SendGrid |
| Deploy frontend | Host `index.html` on Vercel, Netlify, or GitHub Pages |
| Dockerize n8n | Use the official `n8nio/n8n` Docker image with `docker-compose.yml` |

---

## 🐛 Troubleshooting

| Problem | Likely Cause | Fix |
|---------|-----------|-----|
| "No test cases returned" | Groq returned malformed JSON | Check n8n execution logs; adjust temperature lower |
| `401 Unauthorized` from Groq | API key missing or invalid | Add credential in n8n Credentials store |
| Webhook not reachable | n8n not active / wrong URL | Activate workflow; copy correct webhook URL |
| CORS error in browser | Webhook origin restriction | Enable CORS in n8n webhook settings or host frontend on same domain |
| Slack not notifying | `SLACK_WEBHOOK_URL` not set | Add Slack credential or disable the Slack node |

---

## 📜 License

MIT — free to use, fork, and learn from.

---

## 💬 Questions?

Open an [issue](https://github.com/shroffeshh29/My-N8n-Projects/issues) or connect with me on [LinkedIn](https://www.linkedin.com/in/shroffeshh29).

> **Built with curiosity. Shared for the community.**
