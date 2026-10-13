# Converse AI

Build voice agents, talk to them live, and control how each conversation unfolds, stage by stage.

Converse AI is a full-stack platform for designing voice agents and running them in real time. Each agent can follow a staged conversation script, remember what the caller has said, and trigger actions mid-call, such as saving a lead and sending you an email about it.

## What You Can Do

- Set up voice agents with their own prompts, voices, and settings
- Have the platform create the matching voice assistant for you
- Talk to an agent in the browser and watch the transcript appear live
- Break a conversation into stages that advance as the caller responds
- Keep track of details like names and emails as the call progresses
- Fire webhooks and tools when the caller asks for an action
- Review every past call with its full transcript
- Receive a Gmail alert each time a new lead is captured.

## Stack

| Layer | Technology |
|---|---|
| Frontend | React, Vite, React Router |
| Backend | Node.js, Express (CommonJS) |
| Database | PostgreSQL with Prisma |
| Voice | Vapi, Deepgram, OpenAI and ElevenLabs voices |
| Email | Nodemailer over Gmail SMTP |

## How It Fits Together

An agent's configuration is turned into layered prompts. The orchestration engine decides which stage a call is in, the memory layer supplies context, and the tool layer handles actions. The result is injected into the live voice session.

```
Agent config → Prompt layers → Orchestration → Memory → Tools → Live voice session
```

## Anatomy of a Call

1. The caller speaks and the speech is transcribed.
2. The backend updates the conversation state and extracts details worth remembering.
3. The prompt is rebuilt for the current stage.
4. Stage conditions are checked, and the call moves forward if one is met.
5. If the caller asks for an action, the matching tool runs, optionally through a webhook.
6. The assistant replies, and the cycle repeats.

## Lead Capture Walkthrough

A short call might go like this:

| Caller says | System records |
|---|---|
| "I'm Rahul" | name = Rahul |
| "I work at Infosys" | company = google |
| "rahul at gmail dot com" | email = rahul@gmail.com |
| "Please save my details" | `send_lead` runs, then `notify_lead` |

The lead is sent as a structured payload:

```json
{
  "name": "Rahul",
  "company": "google",
  "email": "rahul@gmail.com"
}
```

## Email Alerts

After a lead is saved, the `notify_lead` tool sends a Gmail message to the address you choose. It only runs once `send_lead` has succeeded.

Email is kept separate from lead storage. If sending fails, the lead stays saved.

Add the following to `backend/.env`:

```env
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your-email@gmail.com
SMTP_PASS=your-gmail-app-password
SMTP_FROM=your-email@gmail.com
NOTIFY_EMAIL_TO=recipient@example.com
NOTIFY_CHANNELS=email
```

Gmail requires an App Password here. Your normal login password will be rejected.

## Repository Layout

```
converse-ai/
├── frontend/
│   └── src/
│       ├── components/
│       ├── context/
│       ├── hooks/
│       ├── pages/
│       └── services/
├── backend/
│   ├── config/
│   ├── constants/
│   ├── controllers/
│   ├── middleware/
│   ├── prisma/
│   ├── routes/
│   ├── services/
│   └── server.js
└── README.md
```

Most of the logic lives in `backend/services/`:

| Service | Responsibility |
|---|---|
| `agentService` | Create, update, and delete agents and their assistants |
| `vapiService` | Communicate with the Vapi API |
| `promptService` | Assemble the layered prompt for each stage |
| `orchestrationService` | Run the stage flow and handle transitions |
| `orchestrationSchema` | Validate flow definitions |
| `conditionEvaluator` | Decide whether a transition condition is met |
| `callStateStore` | Hold the live state of active calls |
| `contextService` | Gather context from earlier conversations |
| `memoryService` | Extract and summarize conversation memory |
| `conversationService` | Store calls, messages, and transcripts |
| `toolRegistry`, `toolRouter`, `toolExecutor` | Define, detect, and run tools |
| `notificationService` | Send email alerts |

## Configuration

**Backend** (`backend/.env`)

```env
PORT=3001
FRONTEND_URL=http://localhost:5173
DATABASE_URL=
VAPI_API_KEY=
OPENAI_API_KEY=
DEEPGRAM_API_KEY=
```

Add the SMTP variables from the Email Alerts section.

**Frontend** (`frontend/.env`)

```env
VITE_API_URL=http://localhost:3001
VITE_VAPI_PUBLIC_KEY=
```

## Getting Started

```bash
git clone <repo-url>
cd converse-ai
```

**Start the backend**

```bash
cd backend
npm install
npx prisma generate
npx prisma migrate dev
npm run dev
```

The API runs at `http://localhost:3001`.

**Start the frontend**

```bash
cd frontend
npm install
npm run dev
```

The app runs at `http://localhost:5173`.

## Upcoming Work

- Document uploads and retrieval (RAG)
- Production deployment and scaling
- Call analytics and reporting
- User accounts with real authentication
- Support for multiple users and teams
- CRM integrations

## Goal

Converse AI aims to make structured, multi-step voice conversations easy to design and reliable to run. Voice is the interface, but the real focus is orchestration.
