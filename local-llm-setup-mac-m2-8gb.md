# Local AI on Your Mac — Beginner's Guide

A friendly, step-by-step guide to running your own free AI coding assistant on a **MacBook Pro M2 with 8 GB RAM**, written for someone who is **also just learning macOS**.

By the end you'll have:

- A small AI model running locally on your Mac (no internet needed after setup)
- It working **inside Cursor's chat panel** (your editor)
- It working **inside the Terminal** via a tool called `aider`
- (Optional) Claude Code skipping its forced login screen

No prior Mac or terminal experience required.

---

## Table of Contents

1. [Mac & Terminal 101 — the absolute basics](#1-mac--terminal-101)
2. [What you're about to build (the big picture)](#2-the-big-picture)
3. [Reality check for 8 GB RAM](#3-reality-check-for-8-gb-ram)
4. [Install Homebrew (the Mac app store for developers)](#4-install-homebrew)
5. [Install Ollama (the AI engine)](#5-install-ollama)
6. [Download your first AI model](#6-download-your-first-ai-model)
7. [Test it works](#7-test-it-works)
8. [**Setup A: Use it in Cursor (recommended for everyone)**](#8-setup-a--cursor)
9. [**Setup B: Use it in the Terminal with `aider` (recommended CLI)**](#9-setup-b--aider-in-terminal)
10. [Setup C (advanced): Claude Code with login bypass](#10-setup-c-claude-code-bypass-advanced)
11. [Daily commands cheat-sheet](#11-daily-cheat-sheet)
12. [Practice prompts](#12-practice-prompts)
13. [When to fall back to cloud models](#13-when-to-use-cloud-instead)
14. [Troubleshooting — everything that can break](#14-troubleshooting)
15. [File locations & uninstall](#15-file-locations--uninstall)
16. [TL;DR — just give me the commands](#16-tldr)

---

## 1. Mac & Terminal 101

Before any commands, three things you need to be comfortable with:

### 1.1 Opening Terminal

- Press `Cmd + Space` to open **Spotlight**.
- Type `Terminal` and press `Enter`.
- A black or white window opens with text like:

  ```
  yourname@MacBook ~ %
  ```

  The `%` is the **prompt**. It means "I'm ready for a command."

### 1.2 Running a command

A command is just text you type and then press **Enter**. To run any code block from this guide:

1. Click the small "copy" icon at the top-right of the code block (or select the text and `Cmd + C`).
2. Click into your Terminal window.
3. Paste with `Cmd + V`.
4. Press **Enter**.

Nothing happens until you press Enter.

### 1.3 Vocabulary you'll see in this guide

| Word | What it means |
|---|---|
| **Terminal / shell / CLI** | The black window you just opened. |
| **Command** | A line of text you type and run. |
| **Directory / folder** | Same thing. |
| **`~` (tilde)** | Shortcut for "your home folder" (`/Users/lakshmi.kanth`). |
| **`~/Documents`** | The Documents folder in your home folder. |
| **Install** | Download and set up a program. |
| **Path** | The full address of a file or folder, like `/Users/you/Desktop/file.txt`. |
| **Environment variable** | A setting that programs can read. Set with `export NAME=value`. |
| **Prompt password** | Sometimes a command asks for your Mac login password. Type it (you won't see dots — it's invisible) and press Enter. |

### 1.4 If you get lost

- `Ctrl + C` cancels a running command.
- Close Terminal and re-open it to start fresh. Nothing breaks.
- You **cannot** damage your Mac by running the commands in this guide.

---

## 2. The big picture

You're going to install three things:

```
   ┌──────────────────┐                        ┌────────────────────┐
   │     Cursor       │                        │      aider         │
   │  (your editor)   │                        │   (terminal CLI)   │
   └────────┬─────────┘                        └─────────┬──────────┘
            │                                            │
            └──────────────┐         ┌───────────────────┘
                           ▼         ▼
                      ┌─────────────────────┐
                      │       Ollama        │ ← the AI engine
                      │ http://localhost:11434
                      └──────────┬──────────┘
                                 │
                                 ▼
                       qwen2.5-coder:3b
                        (the AI brain,
                       runs on your M2)
```

- **Ollama** loads and runs the AI model on your Mac.
- **qwen2.5-coder:3b** is the model — the actual "brain". 3B = 3 billion parameters (small but smart enough for Python learning).
- **Cursor** and **aider** are two different ways for you to chat with the model.

Everything runs **on your laptop**. No data leaves your machine.

---

## 3. Reality check for 8 GB RAM

| What | RAM used | Works on 8 GB? |
|---|---|---|
| macOS + Cursor + browser idle | ~4–5 GB | always running |
| Small 3B model (recommended) | ~2–2.5 GB | yes |
| 7B model | ~4–5 GB | **no — Mac will freeze** |
| 13B+ model | 8 GB+ | **don't try** |

**Rules for an 8 GB Mac:**
1. Only use **3B-parameter** models.
2. Close Chrome / Slack / Discord while using the AI.
3. Don't run two models at once.

---

## 4. Install Homebrew

Homebrew is "the Mac app store for developers". It's a single command that lets you install other developer tools easily.

### 4.1 Check if you already have it

Paste this into Terminal and press Enter:

```bash
brew --version
```

- If you see something like `Homebrew 4.x.x` → **skip to section 5**.
- If you see `command not found: brew` → continue below.

### 4.2 Install it

Paste this into Terminal and press Enter:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

What to expect:
- It will ask for your Mac password — type it (invisibly!) and Enter.
- It takes 3–10 minutes. Lots of text scrolls by. **This is normal.**
- At the very end, it tells you to run **two extra commands** that start with `echo`. **Run them.** They make `brew` available in future Terminal windows.

### 4.3 Verify

Close Terminal, open a new Terminal (`Cmd + Space` → "Terminal"), and run:

```bash
brew --version
```

You should now see a version. If not → see [Troubleshooting §14.1](#141-brew-command-not-found-after-install).

---

## 5. Install Ollama

Ollama is the program that actually runs the AI model.

### 5.1 Install

```bash
brew install ollama
```

Takes 30 seconds.

### 5.2 Start it (and make it auto-start every login)

```bash
brew services start ollama
```

This runs Ollama in the background forever. You don't need to start it again after this — it survives reboots.

### 5.3 Verify

```bash
curl http://localhost:11434
```

Expected output:

```
Ollama is running
```

If you don't see that → [Troubleshooting §14.2](#142-ollama-not-running).

---

## 6. Download your first AI model

```bash
ollama pull qwen2.5-coder:3b
```

What to expect:
- ~2 GB download. Takes 2–10 minutes depending on your internet.
- Progress bar shows percentage. **Don't close Terminal until it says "success".**
- If it gets stuck, press `Ctrl + C` and run the command again — it resumes from where it stopped.

### Optional second model (for general questions / explanations)

```bash
ollama pull llama3.2:3b
```

Use this when you want plainer "explain this concept to me" style answers.

### What you now have

```bash
ollama list
```

Shows your installed models. You'll keep these until you delete them with `ollama rm <name>`.

---

## 7. Test it works

In Terminal:

```bash
ollama run qwen2.5-coder:3b
```

You'll see a `>>>` prompt. Type:

```
Write a Python function that returns the nth Fibonacci number.
```

Press Enter. Code should stream out within a second.

**To exit:** type `/bye` and press Enter (or press `Ctrl + D`).

If you got code → the engine works. Now pick your front-end.

---

## 8. Setup A — Cursor

**This is the easiest and the one most people should start with.** Cursor is your code editor; you just point it at Ollama instead of OpenAI.

### 8.1 Configure

1. Open Cursor.
2. Press `Cmd + ,` to open **Cursor Settings**.
3. Click **Models** in the sidebar.
4. Scroll to **OpenAI API Key** section.
5. Click **Override OpenAI Base URL** (or similar; UI may vary slightly).
6. Set:
   - **Base URL:** `http://localhost:11434/v1`
   - **API Key:** `ollama` (literally type the word `ollama`)
7. Scroll up to the **Model Names** list. Click **+ Add model** and type exactly:
   ```
   qwen2.5-coder:3b
   ```
8. Toggle that new model **ON**. Toggle cloud models (gpt-4, claude, etc.) **OFF** for now.
9. Click **Verify** next to the Base URL field — it should say "Verified ✓".

### 8.2 Use it

- Press `Cmd + L` to open the chat panel.
- The model dropdown at the top should show `qwen2.5-coder:3b`.
- Ask anything. First message takes ~3 sec (loading); after that it's snappy.

### 8.3 What does NOT work with the local model

- **Tab autocomplete** → needs Cursor's hosted models.
- **Composer / multi-file agent** → needs Cursor's hosted models.
- **Image input** → 3B model is text-only.

These aren't broken; they're features of Cursor's paid model. You can switch back to a cloud model anytime via the dropdown.

---

## 9. Setup B — `aider` in Terminal

If you want to use AI **from the Terminal** (not from Cursor), `aider` is the best option. It works with Ollama out of the box — **no login, no proxy, no environment variables**.

### 9.1 Install

```bash
brew install aider
```

(Takes ~1 minute.)

### 9.2 Run it inside a project folder

`aider` works on a folder of code. Navigate there first:

```bash
cd ~/Documents/my-python-project        # change to your folder
aider --model ollama/qwen2.5-coder:3b
```

You'll see a prompt. Type questions, paste errors, or say things like:

```
Create a file called hello.py that prints hello world.
```

`aider` will write the file for you, show you a diff, and ask if you want to keep it.

### 9.3 Make it the default model

So you don't have to type `--model ollama/qwen2.5-coder:3b` every time:

```bash
echo "model: ollama/qwen2.5-coder:3b" > ~/.aider.conf.yml
```

Now just run `aider` in any folder.

### 9.4 Useful in-aider commands (type these inside aider's prompt)

| Command | What it does |
|---|---|
| `/add <file>` | Add a file to the AI's context. |
| `/drop <file>` | Remove a file from context. |
| `/diff` | Show what aider changed. |
| `/undo` | Undo aider's last change. |
| `/help` | Full list of commands. |
| `/exit` | Quit. |

---

## 10. Setup C — Claude Code (bypass login) — *advanced*

Claude Code is Anthropic's terminal coding tool. When you run plain `claude`, it **forces you to log in** to an Anthropic account. Here's how to skip that and route it to your local model.

> **Honest warning:** Claude Code was designed around Anthropic's Claude model, which is *much* smarter than your local 3B model. Local + Claude Code = many broken tool calls and confused edits. **`aider` (Setup B) is a better fit for your hardware.** Only do this if you really want Claude Code's specific look-and-feel.

### 10.1 If you already triggered the login screen, clear its state

```bash
rm -rf ~/.claude ~/.config/claude
```

Safe — only removes Claude Code's stored credentials.

### 10.2 Install the router that translates Anthropic → Ollama

```bash
npm install -g @anthropic-ai/claude-code
npm install -g @musistudio/claude-code-router
```

If `npm` is not installed, get it via:

```bash
brew install node
```

### 10.3 Create the router config

```bash
mkdir -p ~/.claude-code-router
nano ~/.claude-code-router/config.json
```

`nano` is a tiny text editor inside the Terminal. Paste:

```json
{
  "Providers": [
    {
      "name": "ollama",
      "api_base_url": "http://localhost:11434/v1/chat/completions",
      "api_key": "ollama",
      "models": ["qwen2.5-coder:3b"]
    }
  ],
  "Router": {
    "default": "ollama,qwen2.5-coder:3b",
    "background": "ollama,qwen2.5-coder:3b",
    "think": "ollama,qwen2.5-coder:3b",
    "longContext": "ollama,qwen2.5-coder:3b"
  }
}
```

Save: `Ctrl + O`, `Enter`, then exit: `Ctrl + X`.

### 10.4 Launch — **never run plain `claude` again**

Always use:

```bash
ccr code
```

What this does:
1. Starts the router proxy on `localhost:3456`.
2. Sets `ANTHROPIC_BASE_URL=http://localhost:3456` and a dummy `ANTHROPIC_API_KEY`.
3. Launches `claude` with those env vars → login screen never appears.

### 10.5 Manual fallback (skip the router)

If you ever want plain `claude` pointing at a proxy yourself:

```bash
export ANTHROPIC_BASE_URL=http://localhost:3456
export ANTHROPIC_API_KEY=sk-not-a-real-key
claude
```

The login screen only triggers when `ANTHROPIC_API_KEY` is **unset or empty**. Set it to literally any string and the OAuth flow is skipped.

To make those `export` lines permanent, add them to `~/.zshrc`:

```bash
echo 'export ANTHROPIC_BASE_URL=http://localhost:3456' >> ~/.zshrc
echo 'export ANTHROPIC_API_KEY=sk-not-a-real-key' >> ~/.zshrc
```

---

## 11. Daily cheat-sheet

```bash
# Ollama
ollama list                         # what's installed
ollama ps                           # what's loaded in RAM right now
ollama run qwen2.5-coder:3b         # chat in terminal
ollama stop qwen2.5-coder:3b        # free RAM
ollama pull <model-name>            # download a new model
ollama rm <model-name>              # delete one
brew services restart ollama        # restart if weird

# aider
aider                               # start in current folder
aider --model ollama/llama3.2:3b    # use a different model once

# Claude Code (via router)
ccr code

# Cursor
# Just press Cmd + L
```

---

## 12. Practice prompts

Try these in Cursor's chat (`Cmd + L`) or in `aider`:

```
Explain what a Python decorator is, with one simple example.
```

```
Here is my code:

def factorial(n):
    return n * factorial(n-1)

Why does this crash? Fix it and explain.
```

```
Write a Python script that reads data.csv and prints the average of the "price" column. Use only the standard library.
```

```
What's the difference between a list and a tuple? When should I use each?
```

```
Review this function and suggest improvements:
<paste your function>
```

---

## 13. When to use cloud instead

A local 3B model is great for:
- Learning Python
- Explaining concepts
- Small snippets and bug fixes
- Q&A while you read tutorials

It is **not** great for:
- Writing entire apps in one shot
- Long agentic refactors across many files
- Reading huge files (> 200 lines at once)
- Math-heavy reasoning
- Anything needing very recent library docs

For those, switch Cursor back to `gpt-4o-mini` (free tier) or use the free ChatGPT web app. You can have both.

---

## 14. Troubleshooting

### 14.1 `brew: command not found` after install

The installer printed instructions about `echo ... >> ~/.zprofile`. You skipped them. Run this (Apple Silicon path):

```bash
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"
```

Then re-test:

```bash
brew --version
```

### 14.2 Ollama not running

`curl http://localhost:11434` returns nothing or "connection refused".

```bash
brew services restart ollama
# wait 3 seconds, then:
curl http://localhost:11434
```

Still broken? Run it in the foreground to see errors:

```bash
ollama serve
```

Keep that Terminal window open and try the curl command in a **new** Terminal window.

### 14.3 Port 11434 in use

```bash
lsof -i :11434
```

Tells you what's using it. Usually it's a zombie Ollama process.

```bash
brew services stop ollama
brew services start ollama
```

### 14.4 `ollama pull` is slow or stuck

- Stuck progress: press `Ctrl + C`, run the same command again — resumes.
- Out of disk space: `df -h ~`. You need ~2 GB free per model.

### 14.5 Mac freezing / fans loud while using AI

You're out of RAM. The model is **swapping to SSD** (very slow).

Fix in this order:
1. Quit Chrome, Slack, Discord, Spotify.
2. Unload the model from RAM: `ollama stop qwen2.5-coder:3b`
3. Switch to a smaller model: `ollama pull gemma2:2b` and use that in Cursor / aider.
4. Watch **Activity Monitor → Memory tab**. If "Memory Pressure" is yellow or red → still too much running.

### 14.6 Cursor "Verify" button fails

| Symptom | Fix |
|---|---|
| "Connection refused" | `brew services restart ollama` |
| "Model not found" | Run `ollama list`, copy the exact name (with `:3b` tag) |
| Hangs forever | Pause your VPN, retry |
| 401 / 403 error | API key field is empty — type `ollama` |

### 14.7 Model gives short / weird / repetitive answers

1. **Context too long** — don't paste 500-line files. Paste only the function.
2. **Wrong model selected** — check the dropdown in Cursor's chat panel.
3. **Out of memory** → see 14.5.

### 14.8 First response after a pause is slow (~5–10 sec)

Normal. Ollama unloads models after 5 min idle to save RAM. First message reloads it.

To keep it loaded longer (uses more RAM):

```bash
brew services stop ollama
OLLAMA_KEEP_ALIVE=2h ollama serve
```

### 14.9 Claude Code keeps asking for login despite env vars

- Did you set **both** `ANTHROPIC_API_KEY` and `ANTHROPIC_BASE_URL`?
- Did you set them in the **same** Terminal window where you run `claude`?
- Try clearing stored creds: `rm -rf ~/.claude ~/.config/claude` and restart.
- Easier path: just use `ccr code` instead of `claude`.

### 14.10 `aider` says "model not found"

Check the prefix is `ollama/`, not just the model name:

```bash
aider --model ollama/qwen2.5-coder:3b      # correct
aider --model qwen2.5-coder:3b             # wrong
```

### 14.11 How to read Ollama's own logs

```bash
tail -f ~/.ollama/logs/server.log
```

Errors show up here. `Ctrl + C` to exit tailing.

---

## 15. File locations & uninstall

| Thing | Path |
|---|---|
| Ollama binary | `/opt/homebrew/bin/ollama` |
| Downloaded models | `~/.ollama/models` |
| Ollama logs | `~/.ollama/logs/server.log` |
| Cursor settings | `~/Library/Application Support/Cursor/User/settings.json` |
| Claude Code state | `~/.claude` and `~/.config/claude` |
| Claude Code router config | `~/.claude-code-router/config.json` |
| aider global config | `~/.aider.conf.yml` |

### Free up disk space

```bash
ollama rm <model-name>          # delete one model
du -sh ~/.ollama/models         # see total size
```

### Full uninstall of Ollama

```bash
brew services stop ollama
brew uninstall ollama
rm -rf ~/.ollama                # removes all models (~few GB)
```

---

## 16. TL;DR

If you read nothing else, run these in order:

```bash
# 1. Install package manager (if you don't have it)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# 2. Install AI engine
brew install ollama
brew services start ollama

# 3. Download the model (~2 GB)
ollama pull qwen2.5-coder:3b

# 4. Test
ollama run qwen2.5-coder:3b "Write a Python hello world."

# 5. Install terminal AI assistant
brew install aider
echo "model: ollama/qwen2.5-coder:3b" > ~/.aider.conf.yml
```

Then in **Cursor**: Settings → Models → Override OpenAI Base URL = `http://localhost:11434/v1`, API key = `ollama`, add model `qwen2.5-coder:3b`, click Verify.

You're done. Unlimited local AI in both your editor and your terminal.

---

*Last updated: June 2026. If anything in this guide stops working, check the Troubleshooting section first, then ask your AI to help debug — yes, ask the AI you just installed.*
