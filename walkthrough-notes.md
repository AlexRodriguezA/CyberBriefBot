# 🎬 CyberBrief — Video Walkthrough Companion Notes

Use these notes alongside the video walkthrough to follow along and understand what is happening at each stage.

---

## 📍 Timestamp Guide

| Section | What's Happening |
|---|---|
| 0:00 | Project overview and what you will build |
| 2:00 | Setting up n8n, OpenAI, and Telegram accounts |
| 8:00 | Building the Ingest layer (RSS feeds) |
| 18:00 | Normalize layer — why we standardize data |
| 24:00 | Aggregate — merging all sources |
| 27:00 | Clean & Filter — deduplication and date window |
| 38:00 | Score, Rank & Select — keyword scoring system |
| 52:00 | AI Analysis — connecting OpenAI |
| 61:00 | Telegram delivery — setting up the bot |
| 68:00 | Publishing and testing the full workflow |

---

## 🧠 Concept Checkpoints

### At the Ingest Layer — Stop and ask yourself:
- Why are we pulling from 5 different sources instead of just 1?
- What is an RSS feed and why do security sites use them?
- What might happen if one source goes down?

> **Answer:** Multiple sources ensure coverage. If BleepingComputer misses a story, TheHackersNews might have it. Redundancy is a core principle in both security and engineering.

---

### At the Normalize Layer — Stop and ask yourself:
- Why do all sources need the same field names?
- What field tells us when an article was published?
- What happens downstream if one article has no URL?

> **Answer:** Downstream nodes like the deduplicator and scorer expect consistent field names. If one source calls it `link` and another calls it `url`, the code breaks. Normalization solves this.

---

### At the Clean & Filter Layer — Stop and ask yourself:
- What is a duplicate story and why does it matter?
- Why do we filter articles older than 72 hours?
- What are URL tracking parameters and why do we strip them?

> **Answer:** The same story often appears on multiple sources. Deduplication prevents the AI from summarizing the same event twice. The 72-hour window keeps the brief focused on current threats, not last week's news. Tracking parameters (like `?utm_source=twitter`) create false duplicates of the same URL.

---

### At the Scoring Layer — Stop and ask yourself:
- Why does "zero-day" score 20 but "phishing" only scores 4?
- What makes something "actively exploited in the wild"?
- What is an APT and why is it high priority?

> **Answer:** Zero-days are unpatched vulnerabilities being exploited with no fix available — maximum urgency. Phishing is common and well-understood — lower urgency for experienced defenders. APT (Advanced Persistent Threat) means a nation-state or sophisticated attacker — high priority due to resources and intent.

---

### At the AI Layer — Stop and ask yourself:
- What is a system prompt and why does it matter?
- Why do we tell the AI NOT to invent facts?
- What does the AI actually receive as input?

> **Answer:** The system prompt defines the AI's role, rules, and output format. Without rules, the AI might exaggerate threats or make up CVEs. The AI receives your top 10 scored articles as structured JSON — the better the input, the better the briefing.

---

### At the Delivery Layer — Stop and ask yourself:
- Why do we split the message into chunks?
- What is a Telegram bot token and why must you keep it secret?
- What would happen if we sent the raw JSON to Telegram instead?

> **Answer:** Telegram has a 4096 character limit per message. The chunker splits long briefings into multiple messages. The bot token is like a password — anyone with it can send messages as your bot. Raw JSON would be unreadable — the formatting step makes it human-readable.

---

## 🔍 IOC Glossary

| Term | Definition | Example |
|---|---|---|
| **CVE** | Common Vulnerabilities and Exposures — a numbered ID for a known vulnerability | CVE-2026-1731 |
| **IOC** | Indicator of Compromise — artifact associated with an attack | IP, domain, hash |
| **APT** | Advanced Persistent Threat — sophisticated, often nation-state attacker | APT29 (Russia) |
| **RCE** | Remote Code Execution — attacker can run code on your system | Critical severity |
| **Zero-day** | Vulnerability with no patch available, actively exploited | Maximum urgency |
| **KEV** | CISA Known Exploited Vulnerabilities catalog | Must-patch list |
| **TTPs** | Tactics, Techniques, and Procedures — how attackers operate | MITRE ATT&CK |
| **MSRC** | Microsoft Security Response Center — Microsoft's vulnerability feed | Patch Tuesday |

---

## ✅ Completion Checklist

Use this to verify your build is complete:

### Accounts
- [ ] n8n account created
- [ ] OpenAI account created with API key
- [ ] Telegram bot created via @BotFather
- [ ] Chat ID retrieved from getUpdates URL

### Workflow Nodes
- [ ] Schedule Trigger (daily at 8am)
- [ ] 5 RSS feed nodes connected
- [ ] 5 Normalize Code nodes (one per source)
- [ ] Merge/Aggregate node (Append mode)
- [ ] CLEAN — Canonicalize URLs & Trim Content
- [ ] FILTER — Deduplicate Stories
- [ ] FILTER — Recent Intel (72h)
- [ ] ENRICH — Score (NEWS)
- [ ] SELECT — Top Intel (News)
- [ ] FORMAT — Briefing Payload (News)
- [ ] MSRC RSS feed + Normalize
- [ ] FILTER — MSRC Recent Window
- [ ] AGGREGATE — MSRC Daily Rollup
- [ ] MERG — SRC ALL (combines news + MSRC)
- [ ] FORMAT — Final Briefing Input
- [ ] Basic LLM Chain with system prompt
- [ ] OpenAI Chat Model subnode (gpt-4o-mini)
- [ ] FORMAT Telegram Chunks
- [ ] Telegram — Send a text message

### Testing
- [ ] Ran workflow manually end-to-end
- [ ] Received message in Telegram
- [ ] Briefing includes date header
- [ ] IOCs section appears in output
- [ ] Workflow published (active)

---

## 🐛 Common Errors & Fixes

| Error | Cause | Fix |
|---|---|---|
| `Status code 404` on CISA | Feed URL changed | Use `https://www.cisa.gov/cybersecurity-advisories/all.xml` |
| `Invalid character in entity name` | Broken RSS feed XML | Replace TheHackersNews URL with Feedburner version |
| `Insufficient quota` | No OpenAI credits | Add billing at platform.openai.com |
| `{"ok":true,"result":[]}` | Bot hasn't received a message | Message your bot first, then retry getUpdates |
| `top_news is empty array` | Pipeline not connected correctly | Check Merge node has both inputs connected |
| Telegram message not formatting | Parse Mode not set | Set Parse Mode to Markdown in Telegram node |

---

## 🚀 What to Do After Completion

1. **Let it run for a week** — you'll start recognizing threat patterns and recurring actors
2. **Check your IOCs section daily** — look up any CVEs on NVD to understand severity
3. **Try the stretch challenges** in the README
4. **Document it on your GitHub** — this is a strong portfolio project for security roles
5. **Add it to your resume** under projects: *"Built automated threat intelligence pipeline using n8n, OpenAI GPT, and Telegram delivering daily IOC-enriched security briefings"*
