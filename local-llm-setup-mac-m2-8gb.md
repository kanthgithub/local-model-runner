# Local LLM Setup — MacBook Pro M2, 8 GB RAM

A beginner-friendly guide to running a free, local AI coding assistant on a low-RAM Mac, and wiring it into Cursor (and optionally Claude Code).

> Audience: Python / AI student, new to terminals and local models.
> Goal: Unlimited, offline Python help directly inside Cursor's chat panel.

---

## 0. What you're about to build

```
   ┌──────────────┐        HTTP (localhost:11434)        ┌─────────────┐
   │   Cursor     │  ───────────────────────────────►   │   Ollama    │
   │ (Ask panel)  │                                      │  + model    │
   └──────────────┘                                      └─────────────┘
                                                                │
                                                                ▼
                                                         qwen2.5-coder:3b
                                                          (runs on your
                                                           M2's GPU)
```

- **Ollama** = the program that loads and runs the AI model on your Mac.
- **Model** = the actual "brain". You'll use a small 3B-parameter model that fits in 8 GB RAM.
- **Cursor** = your editor; it talks to Ollama as if it were OpenAI.

---

## 1. Reality check for 8 GB RAM

| What | RAM used | Works on 8 GB? |
|---|---|---|
| macOS + Cursor + browser idle | ~4–5 GB | always running |
| Small 3B model (recommended) | ~2–2.5 GB | yes |
| 7B model | ~4–5 GB | **no — will swap, freeze, fans** |
| 13B+ model | 8 GB+ | **don't even try** |

**Rule:** stick to **3B models** and close Chrome/Slack/Discord during long sessions.

---

## 2. Install Ollama (one time)

### 2.1 Make sure Homebrew is installed

Open the **Terminal** app (Cmd+Space → "Terminal") and paste:

```bash
brew --version
```

If you see a version number → skip to 2.2.
If you see "command not found", install Homebrew first:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

After it finishes, follow the on-screen instructions (it tells you to run two `echo ... >> ~/.zprofile` commands — copy-paste them).

### 2.2 Install Ollama

```bash
brew install ollama
```

### 2.3 Start the Ollama background server

```bash
brew services start ollama
```

This makes Ollama auto-start every time you log in. It listens at `http://localhost:11434`.

**To check it's running:**

```bash
curl http://localhost:11434
```

You should see `Ollama is running`. If you don't, jump to the [Debugging](#7-debugging--troubleshooting) section.

---

## 3. Download your first model

This downloads ~2 GB once and caches it forever (in `~/.ollama/models`).

```bash
ollama pull qwen2.5-coder:3b
```

This is the **single best small model for Python/coding** as of 2026. It's good at:
- Writing Python functions
- Explaining error messages
- Reviewing your code
- Answering "how do I…?" questions

### Optional second model (general chat / explanations)

```bash
ollama pull llama3.2:3b
```

Use this when you want plainer, more tutor-like explanations of concepts (e.g. "what is a decorator in Python?").

### Optional ultralight model (if 3B feels slow)

```bash
ollama pull gemma2:2b
```

Smaller, faster, slightly less smart. Good fallback when your Mac feels sluggish.

---

## 4. First test — does it work?

In Terminal:

```bash
ollama run qwen2.5-coder:3b
```

You'll get a `>>>` prompt. Type:

```
Write a Python function that returns the nth Fibonacci number using memoization.
```

Press Enter. You should see code stream out within ~1 second.

To exit: type `/bye` and press Enter (or Ctrl+D).

**If it printed code → you're done with the engine. Move to step 5 to wire it into Cursor.**

---

## 5. Connect it to Cursor

1. Open Cursor.
2. Press `Cmd + ,` (or click the gear icon) → **Cursor Settings**.
3. Go to the **Models** tab.
4. Scroll down to **OpenAI API Key**.
5. Click **Override OpenAI Base URL** and set:
   - **Base URL:** `http://localhost:11434/v1`
   - **API Key:** `ollama` (literally type the word `ollama` — any non-empty string works)
6. In the **Model Names** list (further up the page), click **Add model** and type:
   ```
   qwen2.5-coder:3b
   ```
   (the name must match exactly what `ollama list` shows)
7. Toggle that model **on**, toggle the cloud models (gpt-4, claude, etc.) **off** for now.
8. Click **Verify** next to the Base URL. It should say "Verified ✓".

### Using it

- Press `Cmd + L` to open the chat panel.
- Make sure the model dropdown at the top of the chat shows `qwen2.5-coder:3b`.
- Ask anything. The first message takes ~3 sec (the model loads into memory). After that it's snappy.

### What won't work on the local model

- **Tab autocomplete** — requires Cursor's hosted models.
- **Composer / multi-file agent edits** — also requires Cursor's hosted models.
- **Image input** — `qwen2.5-coder:3b` is text-only.

These are not bugs, just limitations of free local models. The free Cursor tier covers them.

---

## 6. (Optional) Connect to Claude Code in the terminal

> Skip this section if you're happy with Cursor's Ask panel. The setup below is brittle on 8 GB.

Claude Code expects Anthropic's API format, so you need a translator.

```bash
npm install -g @anthropic-ai/claude-code
npm install -g @musistudio/claude-code-router
```

Start the router (it spins up a local proxy that speaks Anthropic but routes to Ollama):

```bash
ccr code
```

The first run creates a config at `~/.claude-code-router/config.json`. Edit it to look like this:

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
    "default": "ollama,qwen2.5-coder:3b"
  }
}
```

Restart `ccr code` and you have a Claude-Code-like CLI powered by your local model.

**Warning for 8 GB Macs:** Claude Code's agentic loop does many tool calls in a row. A 3B model will often make mistakes mid-loop. Use it for chat, not for autonomous coding sessions.

---

## 7. Debugging & Troubleshooting

### 7.1 `ollama: command not found`

You haven't installed it, or your shell didn't pick up the new PATH.

```bash
brew install ollama
exec zsh                  # reload the shell
which ollama              # should print /opt/homebrew/bin/ollama
```

### 7.2 `Error: could not connect to ollama app`

The background server isn't running.

```bash
brew services start ollama
# or, foreground (closes when you close the terminal):
ollama serve
```

Then verify:

```bash
curl http://localhost:11434
# Expected:  Ollama is running
```

### 7.3 Port 11434 already in use

Something else is on that port, or Ollama crashed without releasing it.

```bash
lsof -i :11434            # see what's using the port
brew services restart ollama
```

### 7.4 `ollama pull` is super slow or hangs

The download server may be slow. Just leave it. If truly stuck:

```bash
# Ctrl+C to cancel, then retry. Ollama resumes from where it stopped.
ollama pull qwen2.5-coder:3b
```

Check your disk space too: `df -h ~/.ollama`. Each 3B model is ~2 GB.

### 7.5 Mac becomes unresponsive / fans loud while using a model

You're running out of RAM. The model is being **swapped to SSD**, which is 1000× slower than RAM.

**Fix:**
1. Quit Chrome, Slack, Discord, Spotify.
2. Use a smaller model: `ollama pull gemma2:2b` and switch Cursor to that.
3. Close any other Cursor / Code windows.
4. In a new terminal:

```bash
ollama ps                 # see which models are loaded in RAM
ollama stop qwen2.5-coder:3b   # unload it manually
```

Check memory pressure: open **Activity Monitor → Memory tab**. If "Memory Pressure" graph is yellow or red → too much running.

### 7.6 Cursor's "Verify" button fails

Go through this checklist:

| Symptom | Cause | Fix |
|---|---|---|
| "Connection refused" | Ollama not running | `brew services start ollama` |
| "Model not found" | Model name typo in Cursor | Run `ollama list`, copy exact name (including the `:3b` tag) |
| Hangs forever | Firewall / VPN intercepting localhost | Pause VPN, retry |
| 401 / 403 | Empty API key field | Type `ollama` (any non-empty string) |

### 7.7 The model gives short, weird, or repetitive answers

Three common causes:

1. **Context too long.** 3B models have small effective context. Don't paste 500-line files. Paste only the function you care about.
2. **Wrong model selected in Cursor's chat.** The dropdown at the top of `Cmd+L` panel — make sure it says `qwen2.5-coder:3b`, not a cloud model that's been disabled.
3. **Out of memory.** See 7.5. When RAM runs out, the model degrades.

### 7.8 First response after a pause is slow (~5–10 sec)

Normal. Ollama unloads the model after 5 minutes idle to free RAM. First message reloads it.

To keep it loaded longer (uses RAM):

```bash
OLLAMA_KEEP_ALIVE=2h ollama serve
```

(restart Ollama first: `brew services stop ollama` then run the command above)

### 7.9 How to see what's installed / loaded

```bash
ollama list               # all downloaded models
ollama ps                 # which models are currently in RAM
du -sh ~/.ollama/models   # how much disk they're using
```

### 7.10 Delete a model to free disk space

```bash
ollama rm llama3.2:3b
```

### 7.11 Update Ollama and models

```bash
brew upgrade ollama
ollama pull qwen2.5-coder:3b     # re-pulls the latest version
```

### 7.12 Completely uninstall

```bash
brew services stop ollama
brew uninstall ollama
rm -rf ~/.ollama                 # deletes all downloaded models (~few GB)
```

---

## 8. Daily usage cheat-sheet

```bash
ollama serve                       # start (only if not auto-started)
ollama list                        # what's installed
ollama ps                          # what's loaded right now
ollama run qwen2.5-coder:3b        # chat in terminal
ollama stop qwen2.5-coder:3b       # free RAM
ollama pull <name>                 # download a new model
ollama rm <name>                   # delete one
```

In Cursor: just `Cmd + L` → ask away.

---

## 9. Good prompts to learn with

Copy-paste these into Cursor's chat to get comfortable:

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
Write a Python script that reads a CSV file and prints the average of a column.
Use only the standard library.
```

```
What's the difference between a list and a tuple in Python? When should I use each?
```

```
Review this function and suggest improvements:
<paste your code here>
```

---

## 10. When to fall back to cloud models

A local 3B model is great for **learning, explaining, small snippets, and Q&A**. It is **not** great for:

- Writing entire apps in one shot
- Long agentic refactors across many files
- Reading huge files (>200 lines)
- Math-heavy reasoning
- Tasks needing the latest library docs (its training is frozen)

For those, switch the model dropdown in Cursor back to `gpt-4o-mini` (free tier) or use the free ChatGPT web app.

---

## 11. Where things live (so you know what to back up / clean up)

| Thing | Path |
|---|---|
| Ollama binary | `/opt/homebrew/bin/ollama` |
| Downloaded models | `~/.ollama/models` |
| Ollama logs | `~/.ollama/logs/server.log` |
| Cursor settings | `~/Library/Application Support/Cursor/User/settings.json` |
| Claude Code router config | `~/.claude-code-router/config.json` |

To check Ollama logs when something's broken:

```bash
tail -f ~/.ollama/logs/server.log
```

---

## TL;DR (literally the only commands you need to get started)

```bash
brew install ollama
brew services start ollama
ollama pull qwen2.5-coder:3b
ollama run qwen2.5-coder:3b "Hello, write me a Python hello world."
```

Then in Cursor: Settings → Models → Override OpenAI Base URL = `http://localhost:11434/v1`, API key = `ollama`, add model `qwen2.5-coder:3b`, click Verify. Done.
