# CyberBrief — Automated Threat Intelligence Pipeline

> A beginner-friendly homelab project that teaches threat intelligence, automation, and AI integration using n8n, OpenAI, and Telegram.

---

## What You Will Build

A fully automated **Daily Cyber Threat Brief** that:
- Ingests live cybersecurity news from 5+ RSS sources
- Cleans, deduplicates, and filters articles automatically
- Scores and ranks stories by threat severity
- Uses AI (OpenAI GPT) to write a professional analyst briefing
- Extracts IOCs (Indicators of Compromise) from articles
- Delivers the briefing to your phone via Telegram every morning

---

## Learning Objectives

By completing this project, you will understand:

| Concept | What You Learn |
|---|---|
| **Threat Intelligence** | What IOCs are, how defenders consume threat intel, RSS feeds as intel sources |
| **Automation** | How to build no-code workflows with n8n |
| **AI Integration** | How to use LLMs (GPT) as an analyst assistant |
| **Data Pipelines** | Ingest → Normalize → Filter → Enrich → Deliver |
| **API Basics** | Connecting services using API keys and webhooks |

---

## 🧰 Tools & Accounts You Need

| Tool | Cost | Purpose |
|---|---|---|
| [n8n Cloud](https://n8n.io) | Free 14-day trial | Workflow automation platform |
| [OpenAI](https://platform.openai.com) | ~$5 credit | AI briefing generation |
| [Telegram](https://telegram.org) | Free | Delivery channel |

---

## 🗺️ Architecture Overview

```
Triggers (Schedule / Manual)
        │
        ▼
┌─────────────────────┐
│    INGEST LAYER     │  ← RSS feeds from 5 sources
│  TheHackersNews     │
│  BleepingComputer   │
│  KrebsOnSecurity    │
│  CISA               │
│  SANS ISC           │
└─────────────────────┘
        │
        ▼
┌─────────────────────┐
│  NORMALIZE LAYER    │  ← Standardize all fields
│  title, url,        │
│  source, content,   │
│  publishedAt        │
└─────────────────────┘
        │
        ▼
┌─────────────────────┐
│  AGGREGATE (MERGE)  │  ← Combine all sources
└─────────────────────┘
        │
        ▼
┌─────────────────────┐
│  CLEAN & FILTER     │  ← 3 steps:
│  1. Canonicalize    │    Strip tracking params
│  2. Deduplicate     │    Remove duplicate stories
│  3. Date Filter     │    Keep last 72 hours only
└─────────────────────┘
        │
        ▼
┌─────────────────────┐
│  SCORE, RANK,       │  ← 3 steps:
│     SELECT          │    Score by keywords
│  1. ENRICH Score    │    Select top 10
│  2. SELECT Top 10   │    Format payload
│  3. FORMAT Payload  │
└─────────────────────┘
        │
        ▼ (+ MSRC CVE Rollup)
┌─────────────────────┐
│  AGGREGATE ALL      │  ← Merge news + MSRC
└─────────────────────┘
        │
        ▼
┌─────────────────────┐
│  AI ANALYSIS        │  ← GPT writes the brief
│  FORMAT Input       │    + extracts IOCs
│  Basic LLM Chain    │
│  OpenAI Chat Model  │
└─────────────────────┘
        │
        ▼
┌─────────────────────┐
│  DELIVERY           │  ← Telegram bot
│  FORMAT Chunks      │    Split to fit limits
│  Send Message       │    Deliver to phone
└─────────────────────┘
```

---

## 📋 Step-by-Step Setup Guide

### Phase 1 — Accounts & Prerequisites

**Step 1: Create n8n Account**
1. Go to [n8n.io](https://n8n.io) and sign up for a free trial
2. Create a new workflow and name it `CyberBrief`

**Step 2: Create OpenAI Account**
1. Go to [platform.openai.com](https://platform.openai.com)
2. Sign up and navigate to API Keys
3. Create a new key named `CyberBrief`
4. Add $5 in credits under Billing (this lasts months at ~$0.01/run)

**Step 3: Create Telegram Bot**
1. Open Telegram and search for `@BotFather`
2. Send `/newbot`
3. Name your bot (e.g. `CyberBriefBot`)
4. Username must end in `_bot` (e.g. `CyberBriefBot_bot`)
5. Copy the **Bot Token** BotFather gives you
6. Message your new bot once (say "hi")
7. Visit this URL in your browser:
```
https://api.telegram.org/botYOUR_TOKEN/getUpdates
```
8. Find `"chat":{"id":` — that number is your **Chat ID**

---

### Phase 2 — Build the Workflow

> Import the workflow JSON (see `/workflow/cyberbrief.json`) OR build it manually following the steps below.

#### Node 1 — Schedule Trigger
- Trigger Interval: **Days**
- Days Between Triggers: **1**
- Trigger at Hour: **8am**

#### Node 2 — RSS Feed Nodes (5 total)
Add one RSS node per source:

| Node Name | Feed URL |
|---|---|
| TheHackersNews | `https://feeds.feedburner.com/TheHackersNews` |
| BleepingComputer | `https://www.bleepingcomputer.com/feed/` |
| KrebsOnSecurity | `https://krebsonsecurity.com/feed/` |
| CISA | `https://www.cisa.gov/cybersecurity-advisories/all.xml` |
| SANS | `https://isc.sans.edu/rssfeed_full.xml` |

#### Node 3 — Normalize Code Nodes
After each RSS node, add a Code node. Change the `source` name to match:

```javascript
return items.map(item => ({
  json: {
    title: item.json.title || '',
    url: item.json.link || item.json.url || '',
    source: 'BleepingComputer', // ← change this per source
    publishedAt: item.json.pubDate || item.json.isoDate || '',
    content: item.json.contentSnippet || item.json.summary || ''
  }
}));
```

#### Node 4 — Merge (Aggregate)
- Add a **Merge** node set to **Append** mode
- Connect all 5 normalize nodes to it

#### Node 5 — CLEAN (Canonicalize URLs & Trim Content)
```javascript
return $input.all().map(item => {
  let url = item.json.url || '';
  try {
    const u = new URL(url);
    u.search = '';
    url = u.toString();
  } catch(e) {}
  const content = (item.json.content || '').slice(0, 500).trim();
  return { json: { ...item.json, url, content } };
});
```

#### Node 6 — FILTER (Deduplicate Stories)
```javascript
const seen = new Set();
const unique = [];
for (const item of $input.all()) {
  const key = item.json.url || item.json.title;
  if (!seen.has(key)) {
    seen.add(key);
    unique.push(item);
  }
}
return unique;
```

#### Node 7 — FILTER (Recent Intel 72h)
```javascript
const cutoff = Date.now() - (72 * 60 * 60 * 1000);
return $input.all().filter(item => {
  const published = new Date(item.json.publishedAt).getTime();
  return published >= cutoff;
});
```

#### Node 8 — ENRICH Score (NEWS)
```javascript
const keywords = [
  { word: 'zero-day', score: 20 },
  { word: 'zero day', score: 20 },
  { word: 'actively exploited', score: 20 },
  { word: 'exploited in the wild', score: 20 },
  { word: 'in the wild', score: 18 },
  { word: 'remote code execution', score: 15 },
  { word: 'RCE', score: 15 },
  { word: 'critical', score: 8 },
  { word: 'ransomware', score: 8 },
  { word: 'nation state', score: 8 },
  { word: 'APT', score: 8 },
  { word: 'exploit', score: 7 },
  { word: 'CVE', score: 7 },
  { word: 'data leak', score: 7 },
  { word: 'vulnerability', score: 6 },
  { word: 'breach', score: 6 },
  { word: 'malware', score: 5 },
  { word: 'backdoor', score: 5 },
  { word: 'phishing', score: 4 },
  { word: 'patch', score: 4 },
];

const CRITICAL_FLAGS = [
  'zero-day', 'zero day', 'actively exploited',
  'exploited in the wild', 'in the wild'
];

return $input.all().map(item => {
  const text = `${item.json.title} ${item.json.content}`.toLowerCase();
  let score = 0;
  let isCritical = false;
  for (const kw of keywords) {
    if (text.includes(kw.word.toLowerCase())) score += kw.score;
  }
  for (const flag of CRITICAL_FLAGS) {
    if (text.includes(flag.toLowerCase())) { isCritical = true; break; }
  }
  return {
    json: {
      ...item.json, score, isCritical,
      priority: isCritical ? '🚨 CRITICAL' : score >= 15 ? '🔴 HIGH' : score >= 8 ? '⚠️ MEDIUM' : '🔵 LOW'
    }
  };
}).sort((a, b) => {
  if (a.json.isCritical && !b.json.isCritical) return -1;
  if (!a.json.isCritical && b.json.isCritical) return 1;
  return b.json.score - a.json.score;
});
```

#### Node 9 — SELECT Top Intel (News)
```javascript
const all = $input.all();
const critical = all.filter(i => i.json.isCritical);
const nonCritical = all.filter(i => !i.json.isCritical);
return [...critical, ...nonCritical.slice(0, Math.max(10 - critical.length, 3))];
```

#### Node 10 — FORMAT Briefing Payload (News)
```javascript
const items = $input.all();
const critical = items.filter(i => i.json.isCritical);
const high = items.filter(i => !i.json.isCritical && i.json.score >= 15);
const rest = items.filter(i => !i.json.isCritical && i.json.score < 15);

const formatItem = (item, i) =>
  `${i + 1}. [${item.json.priority}] ${item.json.title}
   Source: ${item.json.source} | Score: ${item.json.score}
   URL: ${item.json.url}
   Published: ${item.json.publishedAt}
   Summary: ${item.json.content?.slice(0, 300)}...`;

const allFormatted = [
  ...(critical.length ? ['=== 🚨 ACTIVE EXPLOITATION / ZERO-DAY ===', ...critical.map(formatItem)] : []),
  ...(high.length ? ['=== 🔴 HIGH PRIORITY ===', ...high.map(formatItem)] : []),
  ...(rest.length ? ['=== ⚠️ OTHER INTEL ===', ...rest.map(formatItem)] : []),
].join('\n\n');

return [{ json: { briefingPayload: allFormatted, articleCount: items.length, criticalCount: critical.length, generatedAt: new Date().toISOString() } }];
```

#### Node 11 — FORMAT Final Briefing Input
```javascript
const items = $input.all();
const topNews = items.filter(i => i.json.source !== 'MSRC' && i.json.title && !i.json.briefingPayload);
const briefingItem = items.find(i => i.json.briefingPayload);
const msrcRollup = items.find(i => i.json.source === 'MSRC') || null;

return [{
  json: {
    top_news: topNews.length > 0 ? topNews.map(i => ({
      title: i.json.title || '',
      source: i.json.source || '',
      url: i.json.url || '',
      score: i.json.score || 0,
      priority: i.json.priority || '',
      isCritical: i.json.isCritical || false,
      content: (i.json.content || '').slice(0, 300)
    })) : briefingItem ? [{ briefingText: briefingItem.json.briefingPayload }] : [],
    msrc_rollup: msrcRollup ? msrcRollup.json : null,
    generatedAt: new Date().toISOString()
  }
}];
```

#### Node 12 — Basic LLM Chain
- Source for Prompt: **Define below**
- Add System message (see `/prompts/system-prompt.txt`)
- Add User message: `Here is today's briefing input JSON: {{ JSON.stringify($json, null, 2) }}`
- Connect **OpenAI Chat Model** subnode with your API key, model: `gpt-4o-mini`

#### Node 13 — FORMAT Telegram Chunks
```javascript
const text = $input.first().json.text || '';
const chunks = [];
let current = '';
for (const line of text.split('\n')) {
  if ((current + '\n' + line).length > 4000) {
    chunks.push(current.trim());
    current = line;
  } else {
    current += '\n' + line;
  }
}
if (current.trim()) chunks.push(current.trim());
return chunks.map(chunk => ({ json: { message: chunk } }));
```

#### Node 14 — Telegram (Send a text message)
- Credential: Your Bot Token
- Chat ID: Your Chat ID number
- Text: `{{ $json.message }}`
- Additional Fields → Parse Mode: **Markdown**

---

## 🧠 Key Concepts Explained

### What is Threat Intelligence?
Threat intelligence is information that helps defenders understand **who is attacking, how, and what to do about it**. This pipeline automates the collection of that information from public sources.

### What are IOCs?
**Indicators of Compromise** are digital artifacts associated with an attack:
- **CVE numbers** — known vulnerabilities (e.g. CVE-2026-1731)
- **Malware names** — families like ransomware strains
- **Threat actor names** — APT groups, criminal organizations
- **Affected products** — software or hardware that needs patching

### What is an RSS Feed?
RSS (Really Simple Syndication) is a standardized format websites use to publish their latest content. Security news sites expose their articles as RSS feeds, which we can automatically ingest.

### Why Score Articles?
Not all security news is equally urgent. A zero-day being actively exploited requires immediate action. A general awareness article does not. The scoring system uses keyword weights to surface the most critical stories automatically.

### What does the AI do?
The AI (GPT) acts as an analyst assistant. It receives the top scored articles and writes a structured briefing in the format a SOC analyst would want to read — concise, actionable, and organized by threat category.

---

## 📁 Repository Structure

```
cyberbrief/
├── README.md                  ← This file
├── workflow/
│   └── cyberbrief.json        ← Import this into n8n
├── prompts/
│   └── system-prompt.txt      ← AI analyst system prompt
└── docs/
    └── walkthrough-notes.md   ← Video walkthrough companion notes
```

---

## 📜 License

MIT License — free to use, modify, and share.

---

## 🙌 Credits

Built as part of the **Homelab Experience** curriculum.
Inspired by the daily workflow of real SOC analysts.
