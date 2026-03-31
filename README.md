# 🤖 Inbound Lead Qualification Agent — n8n

An AI-powered automation that qualifies inbound leads in real time, routes them to the right sales rep, and sends personalized email responses — all without any manual effort.

---

## 📌 Overview

When companies market themselves well, they receive far more inbound leads than they can realistically handle — many of which are not a good fit. This project automates the **lead qualification process** for **Big Boy Recruits**, a Dallas-based IT recruitment firm.

The agent:
- Accepts form submissions from potential leads
- Scrapes and researches the lead's company via **Relevance AI**
- Uses an **AI Agent** (powered by DeepSeek via Ollama) to qualify or reject the lead
- If **qualified** → classifies into *SaaS* or *Agency*, notifies the sales team, and sends a confirmation email
- If **not qualified** → sends a polite rejection email automatically

---

## 🗂️ Workflows

This project consists of **two n8n workflows**:

| File | Description |
|---|---|
| `Lead_Qualification_Agent__Form_Submissions_.json` | Main workflow — handles form submission, company research, and AI qualification |
| `Qualified_Lead_Classifier_AND_Notifier.json` | Sub-workflow — classifies qualified leads (SaaS vs Agency) and sends notifications |

---

## ⚙️ How It Works

```
Lead submits form
       ↓
Relevance AI scrapes company website
       ↓
AI Agent analyzes lead + company info
       ↓
  ┌────┴────┐
Qualified?  Not Qualified?
  ↓              ↓
Sub-workflow   Rejection email
  ↓            sent to lead
Classify:
SaaS or Agency
  ↓
Notify sales team (email)
  ↓
Send confirmation email to lead
```

---

## 🛠️ Setup Instructions

### Prerequisites

- [n8n](https://n8n.io/) (self-hosted or cloud)
- [Ollama](https://ollama.com/) running locally with `deepseek-v3.1:671b-cloud` model
- A **Gmail** account connected via OAuth2 in n8n
- A **Relevance AI** account with the company researcher agent set up

---

### Step 1 — Install & Run n8n

#### Option A: Using npm
```bash
npm install n8n -g
n8n start
```

#### Option B: Using Docker (recommended)
```bash
docker run -it --rm \
  --name n8n \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  docker.n8n.io/n8nio/n8n
```

Open your browser and navigate to:
```
http://localhost:5678
```

---

### Step 2 — Import the Workflows

1. In n8n, click **"Add workflow"** → **"Import from file"**
2. Import `Lead_Qualification_Agent__Form_Submissions_.json` first
3. Then import `Qualified_Lead_Classifier_AND_Notifier.json`
4. Note the **Workflow ID** of the Notifier workflow (visible in the URL) — you'll need it in Step 5

---

### Step 3 — Connect Your Credentials

In n8n, go to **Settings → Credentials** and add the following:

#### Gmail (OAuth2)
1. Click **"Add credential"** → Search **Gmail OAuth2**
2. Follow the OAuth flow to connect your Google account
3. Make sure the credential name matches `Gmail account` (or update it in the workflow nodes)

#### Ollama
1. Click **"Add credential"** → Search **Ollama**
2. Set the base URL to your Ollama server (e.g., `http://localhost:11434`)
3. Name it `Ollama account Real`

---

### Step 4 — Configure the Relevance AI HTTP Request

In the **`Lead_Qualification_Agent__Form_Submissions_`** workflow, click the **HTTP Request** node and update the URL to:

```
https://app.relevanceai.com/form/d7b62b/2024dcc2-fbb4-4260-92b3-edfce443909f?version=latest&primaryColor=%23000000
```

> **Note:** Make sure you are using the **v1** endpoint on Relevance AI, **not v2**.

The request body should send the company URL from the form:
```json
{
  "company_url": "{{ $json['Company website'] }}"
}
```

Also insert your **Relevance AI API key** into the HTTP Request node headers if prompted.

---

### Step 5 — Link the Two Workflows

In the **`Lead_Qualification_Agent__Form_Submissions_`** workflow:

1. Click the **"Call 'Qualified Lead Classifier AND Notifier'"** tool node
2. Under **Workflow**, select or paste the ID of your imported `Qualified_Lead_Classifier_AND_Notifier` workflow
3. Confirm the input fields map correctly:
   - `Name` → Lead's name
   - `Email` → Lead's email
   - `Message` → Their request
   - `Company info` → AI-summarized company details
   - `Qualified` → Boolean from AI decision

---

### Step 6 — Update Email Recipients

In the **`Qualified_Lead_Classifier_AND_Notifier`** workflow:

- Open the **"New Agency lead"** Gmail node → update `sendTo` to your sales team's email
- Open the **"New SaaS lead"** Gmail node → update `sendTo` to your sales team's email

In the **`Lead_Qualification_Agent__Form_Submissions_`** workflow:

- The rejection email is sent automatically to `{{ $('On form submission').item.json.Email }}` — no changes needed

---

### Step 7 — Activate Both Workflows

1. Open each workflow
2. Toggle the **Active** switch in the top-right corner
3. Copy the **Form Trigger URL** from the `On form submission` node and share it with your team or embed it on your website

---

## 🧪 Testing

1. Open the Form Trigger URL in a browser
2. Fill in the form with a test company (try a SaaS company URL)
3. Watch the execution log in n8n
4. Check your inbox for the sales notification and the lead's inbox for the confirmation/rejection email

---

## 📁 Project Structure

```
├── Lead_Qualification_Agent__Form_Submissions_.json   # Main workflow
├── Qualified_Lead_Classifier_AND_Notifier.json        # Sub-workflow (classifier + notifier)
└── README.md
```

---

## 🔑 Key Technologies

| Technology | Role |
|---|---|
| **n8n** | Workflow automation platform |
| **DeepSeek v3.1 (via Ollama)** | AI model for qualification & classification |
| **Relevance AI** | Company website scraper / researcher |
| **Gmail (OAuth2)** | Sending emails to leads and sales team |

---

## 📬 Email Templates

**Rejection Email (to unqualified lead):**
> Thanks for your interest in Big Boy Recruitment services! As we specialize in recruitment for software businesses such as SaaS and development agencies, we're not a good fit based on your company's industry. Please let us know if you'd like to be connected with one of our partners.

**Sales Notification (internal):**
> New [SaaS/Agency] Lead: `{{ Company Info }}`

**Confirmation Email (to qualified lead):**
> We are pleased to inform you that your request has been accepted. Further details regarding next steps will be shared shortly.

---

## ⚠️ Important Notes

- Always use the **v1** Relevance AI endpoint, not v2
- The Ollama model (`deepseek-v3.1:671b-cloud`) must be running and accessible from your n8n instance
- Both workflows must be **active** for the full automation to work
- Credentials (Gmail, Ollama) must be reconnected after import — they do not transfer between n8n instances

---

## 📄 License

This project is open for personal and commercial use. Attribution appreciated but not required.
