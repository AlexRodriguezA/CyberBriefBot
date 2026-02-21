# 📦 How to Add This Project to GitHub

Follow these steps to publish your CyberBrief project to GitHub so students can find and clone it.

---

## Step 1 — Create a GitHub Account
If you don't have one, go to [github.com](https://github.com) and sign up for free.

---

## Step 2 — Create a New Repository
1. Click the **+** icon in the top right → **New repository**
2. Repository name: `cyberbrief`
3. Description: `Automated daily cybersecurity threat intelligence pipeline using n8n, OpenAI, and Telegram`
4. Set to **Public** so students can access it
5. Check **Add a README file**
6. Click **Create repository**

---

## Step 3 — Upload Your Files

### Option A — GitHub Web Interface (easiest for beginners)
1. Open your repo on GitHub
2. Click **Add file** → **Upload files**
3. Drag and drop all files from this project
4. Click **Commit changes**

### Option B — GitHub Desktop App
1. Download [GitHub Desktop](https://desktop.github.com)
2. Clone your new repo to your computer
3. Copy all project files into the folder
4. Click **Commit to main** → **Push origin**

### Option C — Git Command Line
```bash
git clone https://github.com/YOURUSERNAME/cyberbrief.git
cd cyberbrief
# copy all project files here
git add .
git commit -m "Initial commit - CyberBrief threat intelligence pipeline"
git push origin main
```

---

## Step 4 — Recommended Folder Structure

Make sure your repo looks like this before pushing:

```
cyberbrief/
├── README.md
├── workflow/
│   └── cyberbrief.json        ← Export from n8n
├── prompts/
│   └── system-prompt.txt
└── docs/
    └── walkthrough-notes.md
```

---

## Step 5 — Export Your n8n Workflow JSON

To get the `cyberbrief.json` file:
1. Open your workflow in n8n
2. Click the **...** menu (top right of canvas)
3. Click **Download**
4. Rename the file to `cyberbrief.json`
5. Place it in the `workflow/` folder of your repo

> ⚠️ **IMPORTANT:** Before uploading, open the JSON file and **remove any API keys or tokens**. Search for your OpenAI key and Telegram token and delete those values. Students will add their own credentials.

---

## Step 6 — Add a Good Description and Topics

On your GitHub repo page:
1. Click the **gear icon** next to About
2. Add a description
3. Add topics: `cybersecurity`, `threat-intelligence`, `n8n`, `automation`, `openai`, `telegram`, `homelab`

This helps students find it through search.

---

## Step 7 — Share With Students

Give students this starter checklist:
1. Go to your GitHub repo URL
2. Click the green **Code** button → **Download ZIP** (easiest for beginners)
3. Follow the README.md step by step
4. Reference `docs/walkthrough-notes.md` while watching the video

---

## 🔒 Security Reminder for Students

Never commit real API keys or tokens to GitHub. If you accidentally push a key:
1. Immediately revoke it in the provider's dashboard (OpenAI, Telegram BotFather)
2. Generate a new key
3. Remove it from your git history

Use environment variables or n8n's built-in credential manager instead of hardcoding secrets.
