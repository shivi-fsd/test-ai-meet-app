# 🚀 Meet AI — Full Project Setup Guide

A step-by-step guide to set up this AI-powered meeting platform on any machine.

---

## 📋 Prerequisites

Install these on the new system first:

| Tool | Version | Download |
|------|---------|----------|
| **Node.js** | v18+ | https://nodejs.org |
| **Git** | Latest | https://git-scm.com |
| **ngrok** | v3 | https://ngrok.com/download |

---

## Step 1 — Clone the Repository

```bash
git clone https://github.com/shivi-fsd/test-ai-meet-app.git
cd test-ai-meet-app
```

---

## Step 2 — Install Dependencies

```bash
npm install --legacy-peer-deps
```

> ⚠️ The `--legacy-peer-deps` flag is required due to a version conflict between `@stream-io/openai-realtime-api` and `@stream-io/node-sdk`.

---

## Step 3 — Create External Accounts & Get API Keys

You need accounts on **5 services**. Here's exactly what to get from each:

### 3a. Neon (PostgreSQL Database)
1. Go to [neon.tech](https://neon.tech) → Create free account
2. Create a new **Project**
3. Go to **Dashboard** → **Connection string**
4. Copy the `postgresql://...` connection string

### 3b. Better Auth (Authentication)
- No account needed — just generate a random secret:
```bash
# Run this in any terminal to generate a secret
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
```

### 3c. GitHub OAuth App (Social Login)
1. Go to GitHub → **Settings** → **Developer Settings** → **OAuth Apps** → **New OAuth App**
2. Set:
   - **Homepage URL**: `http://localhost:3000`
   - **Callback URL**: `http://localhost:3000/api/auth/callback/github`
3. Copy **Client ID** and **Client Secret**

### 3d. Google OAuth (Social Login)
1. Go to [console.cloud.google.com](https://console.cloud.google.com)
2. Create a new project → **APIs & Services** → **Credentials** → **Create OAuth Client ID**
3. Type: **Web application**
4. Add Authorized redirect URI: `http://localhost:3000/api/auth/callback/google`
5. Copy **Client ID** and **Client Secret**

### 3e. Stream Video (Video Calls + AI)
1. Go to [getstream.io](https://getstream.io) → Create free account
2. Create a new **App** → Select **Video & Audio**
3. Go to **App Settings**
4. Copy **API Key** and **Secret Key**
5. ⚠️ **Enable OpenAI Realtime**: In Stream dashboard, go to **AI Features** and enable it

### 3f. OpenAI (AI Tutor)
1. Go to [platform.openai.com](https://platform.openai.com)
2. Go to **API Keys** → Create new key
3. Make sure you have **credits** (Realtime API requires a paid account)

### 3g. ngrok (Webhook Tunnel)
1. Go to [dashboard.ngrok.com](https://dashboard.ngrok.com) → Create free account
2. Go to **Domains** → **New Domain** → Get a free static domain
3. Copy your **authtoken** from the ngrok dashboard

---

## Step 4 — Create the `.env` File

Create a file named `.env` in the project root:

```env
# Database (from Neon)
DATABASE_URL="postgresql://..."

# Auth
BETTER_AUTH_SECRET="your-generated-secret-here"
BETTER_AUTH_URL="http://localhost:3000"

# GitHub OAuth
GITHUB_CLIENT_ID="your-github-client-id"
GITHUB_CLIENT_SECRET="your-github-client-secret"

# Google OAuth
GOOGLE_CLIENT_ID="your-google-client-id"
GOOGLE_CLIENT_SECRET="your-google-client-secret"

# App URL
NEXT_PUBLIC_APP_URL="http://localhost:3000"

# Stream Video (same key used for both video and chat)
NEXT_PUBLIC_STREAM_VIDEO_API_KEY="your-stream-api-key"
STREAM_VIDEO_SECRET_KEY="your-stream-secret-key"
NEXT_PUBLIC_STREAM_CHAT_API_KEY="your-stream-api-key"
STREAM_CHAT_SECRET_KEY="your-stream-secret-key"

# OpenAI
OPENAI_API_KEY="sk-proj-..."
```

---

## Step 5 — Create the `ngrok.yml` File

Create a file named `ngrok.yml` in the project root:

```yaml
version: "3"
agent:
  authtoken: YOUR_NGROK_AUTHTOKEN

tunnels:
  meetai:
    proto: http
    addr: 3000
    domain: YOUR_NGROK_STATIC_DOMAIN
```

> Replace `YOUR_NGROK_AUTHTOKEN` and `YOUR_NGROK_STATIC_DOMAIN` with your values from the ngrok dashboard.

---

## Step 6 — Sync the Database Schema

This creates all the required tables in your Neon database:

```bash
npm run db:push
```

You should see: `[✓] No changes detected` or a list of applied changes.

---

## Step 7 — Configure Stream Webhook

1. Go to [getstream.io/dashboard](https://getstream.io/dashboard)
2. Select your App → **Webhooks**
3. Click **Add Webhook**
4. Set URL: `https://YOUR_NGROK_DOMAIN/api/webhook`
5. Enable these events:
   - ✅ `call.session_started`
   - ✅ `call.session_participant_left`
6. Save

---

## Step 8 — Run the Project

You need **two terminals** running simultaneously:

**Terminal 1 — Start the dev server:**
```bash
npm run dev
```
Wait until you see: `✓ Ready on http://localhost:3000`

**Terminal 2 — Start the ngrok tunnel:**
```bash
npm run dev:webhook
```
Wait until you see: `Session Status: online`

---

## Step 9 — Verify Everything Works

1. Open http://localhost:3000
2. Sign in with GitHub or Google
3. Go to **Agents** → Create a new AI agent with instructions
4. Go to **Meetings** → Create a new meeting using that agent
5. Click **Join Meeting** → You should see the call lobby
6. Click **Join** → The AI tutor should connect within a few seconds

---

## 🔧 Troubleshooting

| Problem | Solution |
|---------|----------|
| `DATABASE_URL is not defined` | Check `.env` file exists and has the correct value |
| `Maximum update depth exceeded` | Hard refresh the browser (Ctrl+Shift+R) |
| AI tutor not joining | Check Terminal 1 for `[Webhook]` logs; ensure ngrok is running |
| `ERR_NGROK_3200` | Start `npm run dev:webhook` in a second terminal |
| `ERR_NGROK_8012` | Ensure `npm run dev` is running on port 3000 |
| Auth redirect error | Double-check GitHub/Google OAuth callback URLs |
| Stream errors | Verify API Key and Secret Key match in Stream dashboard |

---

## 📁 Project Structure Overview

```
test-ai-meet-app/
├── src/
│   ├── app/              # Next.js App Router pages
│   │   ├── api/webhook/  # Stream webhook (AI tutor trigger)
│   │   └── (dashboard)/  # Protected dashboard pages
│   ├── db/               # Drizzle ORM schema & client
│   ├── hooks/            # React hooks (useConfirm, etc.)
│   ├── lib/              # Auth, Stream clients
│   ├── modules/          # Feature modules (meetings, agents, call)
│   └── trpc/             # tRPC router & client setup
├── .env                  # Environment variables (create this)
├── ngrok.yml             # ngrok tunnel config (create this)
└── drizzle.config.ts     # Database config
```
