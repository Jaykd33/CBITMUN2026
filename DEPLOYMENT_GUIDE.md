# CBITMUN 2026 — AI Chatbot Deployment Guide
## Zero to Live in ~45 Minutes | 100% Free

---

## What You're Deploying

```
cbitmun.in/
├── index.html              ← replace with new version (chatbot injected)
├── committees.html         ← replace with new version (redesigned)
├── chatbot/
│   ├── chatbot.css         ← new file
│   └── chatbot.js          ← new file

cloudflare-worker/          ← lives on Cloudflare, never on your web server
├── worker.js
└── wrangler.toml

ingestion/                  ← run once on your computer, then delete
├── ingest.js
└── package.json
```

---

## STEP 1 — Get Your 4 Free API Keys (15 min)

### A. Google Gemini API Key
Used for: AI chat answers (RAG mode) + embeddings

1. Go to https://aistudio.google.com
2. Sign in with any Google account (your personal Gmail is fine)
3. Click **"Get API key"** in the top left
4. Click **"Create API key"** → select **"Create API key in new project"**
5. Copy the key — it starts with `AIzaSy...`
6. **Save it as:** `GEMINI_API_KEY`

Free limits: 1,500 requests/day, 1M tokens/minute — more than enough.

---

### B. Groq API Key
Used for: General AI answers (fast Llama 3.3 70B)

1. Go to https://console.groq.com
2. Sign up (Google or GitHub login works)
3. Left sidebar → **"API Keys"**
4. Click **"Create API Key"** → name it `cbitmun`
5. Copy the key — it starts with `gsk_...`
6. **Save it as:** `GROQ_API_KEY`

Free limits: 14,400 requests/day — no credit card.

---

### C. Supabase Project (Vector Database)
Used for: Storing CBITMUN knowledge embeddings

1. Go to https://supabase.com
2. Sign up with GitHub
3. Click **"New Project"**
   - Name: `cbitmun-rag`
   - Database password: choose something strong, **save it**
   - Region: **Southeast Asia (Singapore)** — closest to Hyderabad
4. Wait ~2 minutes for it to spin up
5. Go to **Settings** (gear icon, bottom left) → **API**
6. Copy and save:
   - **Project URL** → save as `SUPABASE_URL`  (looks like `https://abcxyz.supabase.co`)
   - **anon / public key** → save as `SUPABASE_ANON_KEY`
   - **service_role key** → save as `SUPABASE_SERVICE_KEY` (keep this secret — only used for ingestion)

---

### D. Cloudflare Account (Worker + KV)
Used for: Backend proxy — keeps all API keys hidden from the browser

1. Go to https://cloudflare.com
2. Sign up for a free account (just an email, no credit card)
3. That's it for now — you'll use the CLI below.

---

## STEP 2 — Set Up Supabase Vector Database (5 min)

1. In your Supabase dashboard, click **"SQL Editor"** (left sidebar, table icon)
2. Click **"New query"**
3. Paste this entire SQL block and click **"Run"**:

```sql
-- Enable pgvector
CREATE EXTENSION IF NOT EXISTS vector;

-- Knowledge table
CREATE TABLE IF NOT EXISTS cbitmun_docs (
  id          BIGSERIAL PRIMARY KEY,
  page_path   TEXT NOT NULL,
  page_title  TEXT,
  content     TEXT NOT NULL,
  embedding   VECTOR(768),
  created_at  TIMESTAMP DEFAULT NOW()
);

-- Fast similarity search index
CREATE INDEX ON cbitmun_docs
  USING hnsw (embedding vector_cosine_ops)
  WITH (m = 16, ef_construction = 64);

-- Search function called by the Worker
CREATE OR REPLACE FUNCTION match_cbitmun_docs(
  query_embedding   VECTOR(768),
  match_count       INT     DEFAULT 4,
  similarity_threshold FLOAT DEFAULT 0.4
)
RETURNS TABLE (
  id          BIGINT,
  page_path   TEXT,
  page_title  TEXT,
  content     TEXT,
  similarity  FLOAT
)
LANGUAGE plpgsql AS $$
BEGIN
  RETURN QUERY
  SELECT
    d.id,
    d.page_path,
    d.page_title,
    d.content,
    1 - (d.embedding <=> query_embedding) AS similarity
  FROM cbitmun_docs d
  WHERE 1 - (d.embedding <=> query_embedding) > similarity_threshold
  ORDER BY d.embedding <=> query_embedding
  LIMIT match_count;
END;
$$;
```

You should see **"Success. No rows returned"** — that means it worked.

---

## STEP 3 — Run the Ingestion Script (10 min)

This embeds all CBITMUN knowledge into the vector database.
Run this **once now**, and again **whenever you update the website**.

```bash
# 1. Navigate to the ingestion folder
cd ingestion

# 2. Install the one dependency
npm install

# 3. Run with your keys (replace the values)
SUPABASE_URL="https://YOUR_PROJECT.supabase.co" \
SUPABASE_SERVICE_KEY="your_service_role_key" \
GEMINI_API_KEY="AIzaSy..." \
node ingest.js
```

**Windows (Command Prompt):**
```cmd
set SUPABASE_URL=https://YOUR_PROJECT.supabase.co
set SUPABASE_SERVICE_KEY=your_service_role_key
set GEMINI_API_KEY=AIzaSy...
node ingest.js
```

**What you'll see:**
```
🚀 CBITMUN RAG Ingestion Starting…
   19 chunks to embed and store.

🗑  Cleared existing entries.

[ 1/19] "CBITMUN 2026 — Overview"… ✅
[ 2/19] "About CBIT"… ✅
...
[19/19] "Social Responsibility Department (SRD)"… ✅

✨ Done! 19 succeeded, 0 failed.
   Your CBITMUN RAG database is ready.
```

It takes about **90 seconds** (rate-limited to respect Gemini free tier).

> **Note:** If a chunk fails, just re-run — existing entries are cleared first.

---

## STEP 4 — Deploy the Cloudflare Worker (10 min)

```bash
# 1. Install Wrangler CLI (Cloudflare's deploy tool)
npm install -g wrangler

# 2. Login to Cloudflare
wrangler login
# (opens browser, click Allow)

# 3. Navigate to worker folder
cd cloudflare-worker   # or wherever you put worker.js + wrangler.toml

# 4. Create the KV namespace for rate limiting + analytics
wrangler kv:namespace create "KV"
```

Copy the `id` value from the output. It looks like:
```
{ binding = "KV", id = "abc123def456..." }
```

Open `wrangler.toml` and paste it:
```toml
[[kv_namespaces]]
binding = "KV"
id      = "abc123def456..."   ← paste here
```

```bash
# 5. Add your secrets (you'll be prompted to paste each value)
wrangler secret put GEMINI_API_KEY
wrangler secret put GROQ_API_KEY
wrangler secret put SUPABASE_URL
wrangler secret put SUPABASE_ANON_KEY

# 6. Deploy
wrangler deploy
```

You'll get a URL like:
```
✅  Deployed to: https://cbitmun-chatbot.YOUR-SUBDOMAIN.workers.dev
```

**Copy this URL** — you need it in the next step.

---

## STEP 5 — Connect the Chatbot to Your Worker (2 min)

Open `chatbot/chatbot.js` and find line 11:

```javascript
var WORKER_URL = 'https://YOUR-WORKER.YOUR-SUBDOMAIN.workers.dev/chat';
```

Replace it with your actual Worker URL:
```javascript
var WORKER_URL = 'https://cbitmun-chatbot.your-subdomain.workers.dev/chat';
```

Save the file.

---

## STEP 6 — Copy Files to Your Website (5 min)

Copy these files/folders into your CBITMUN website root:

```
Your website root/
├── index.html          ← replace existing
├── committees.html     ← replace existing
└── chatbot/            ← new folder
    ├── chatbot.css
    └── chatbot.js
```

That's it. No changes to any other file.

---

## STEP 7 — Test It

1. Open your website in a browser
2. You should see a **gold pulsing button** in the bottom-right corner
3. Click it — ARIA opens
4. Try: *"What committees are there?"* → should answer from CBITMUN knowledge
5. Try: *"What is a position paper?"* → should switch to General AI mode

---

## Updating Website Content

Whenever you add or change content on the website:

```bash
# Re-run ingestion — takes 90 seconds
cd ingestion
SUPABASE_URL="..." SUPABASE_SERVICE_KEY="..." GEMINI_API_KEY="..." node ingest.js
```

No redeployment of anything else needed.

---

## Troubleshooting

| Problem | Fix |
|---|---|
| Chatbot doesn't appear | Check browser console for JS errors; confirm `chatbot/chatbot.js` path is correct |
| "Rate limit reached" error | Wait 1 hour — 30 requests/hour/IP limit. Increase it in `worker.js` line 14 |
| Worker returns 403 | Check `CORS` in worker.js — update `Access-Control-Allow-Origin` to your domain |
| Ingestion fails on a chunk | Re-run — the script clears and re-embeds everything |
| Groq answers are slow | Normal on first request (cold start). Subsequent ones are fast |
| `wrangler login` doesn't open browser | Run `wrangler login --no-browser` and follow the printed URL |

---

## Free Tier Limits Summary

| Service | Free Limit | Your Usage (est.) |
|---|---|---|
| Gemini API | 1,500 req/day | ~50 req/day |
| Groq | 14,400 req/day | ~50 req/day |
| Supabase | 500 MB + 50K rows | < 1 MB |
| Cloudflare Workers | 100,000 req/day | ~200 req/day |
| Cloudflare KV | 100K reads/day | ~400 req/day |

**You are nowhere near any limit.** All free, forever, for a conference of this scale.

---

## When Background Guides Are Released

In `committees.html`, find each committee's button:
```html
<a href="assets/bg-guides/disec.pdf" target="_blank" class="cmt-bg-btn soon">
```

1. Upload the PDF to `assets/bg-guides/disec.pdf`
2. Remove the `soon` class from the button:
```html
<a href="assets/bg-guides/disec.pdf" target="_blank" class="cmt-bg-btn">
```
3. The button becomes fully active with gold hover effect.

Do the same for each committee: `unhrc.pdf`, `imf.pdf`, `jpc.pdf`, `hcc.pdf`, `ip.pdf`.

---

*Built for CBITMUN 2026 · CBIT Hyderabad*
