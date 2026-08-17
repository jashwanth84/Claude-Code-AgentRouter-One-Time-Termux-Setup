# 🚀 Claude Code + AgentRouter — One-Time Termux Setup

Run Claude Code on Android through Termux + Ubuntu, using AgentRouter as the API provider.

**The goal of this project is simple:**

> Install everything once. When an AgentRouter API key reaches its credit/quota limit, change only the API key. No reinstallation required.

---

## ✨ Features

- 📱 Android + Termux
- 🐧 Ubuntu through PRoot-Distro
- ⚡ Node.js + npm
- 🔧 Git
- 🤖 Claude Code CLI
- 🌐 AgentRouter integration
- 🔑 Persistent API-key storage
- 🔄 Simple API-key rotation
- 📂 Android project/storage support
- 🔒 Local API-key protection
- 🚫 No repeated installation after API-key changes

---

## 🏗️ Architecture

```
Android
   │
   ▼
Termux
   │
   ▼
PRoot-Distro
   │
   ▼
Ubuntu
   │
   ├── Node.js
   ├── npm
   └── Git
        │
        ▼
   Claude Code
        │
        ▼
   AgentRouter
        │
        ▼
   API Key
```

---

## 📦 What Gets Installed

Only the required components:

1. Termux
2. PRoot-Distro
3. Ubuntu
4. Node.js
5. npm
6. Git
7. Claude Code

AgentRouter itself does not need to be installed as a package. It is the remote API service used by Claude Code.

---

## 📱 Requirements

- Android phone
- Termux
- Internet connection
- AgentRouter API key
- Sufficient storage

Root is **NOT** required.

---

## 🚀 ONE-TIME INSTALLATION

### 1. Update Termux

Open Termux:

```bash
pkg update -y && pkg upgrade -y
```

### 2. Install PRoot-Distro

```bash
pkg install -y proot-distro
```

### 3. Enable Android Storage

```bash
termux-setup-storage
```

Tap **Allow** when Android asks for permission.

Android storage will be available at:

```
/storage/emulated/0/
```

Example:

```
/storage/emulated/0/Download
/storage/emulated/0/Documents
/storage/emulated/0/Projects
```

### 🐧 4. Install Ubuntu

```bash
proot-distro install ubuntu
```

This is a **one-time installation**. Do not reinstall Ubuntu when your AgentRouter credits expire.

### 🛠️ 5. Install Node.js, npm and Git

```bash
proot-distro login ubuntu -- bash -c '
apt update
apt install -y nodejs npm git
'
```

Verify:

```bash
proot-distro login ubuntu -- bash -c '
node -v
npm -v
git --version
'
```

### 🤖 6. Install Claude Code

Install Claude Code once:

```bash
proot-distro login ubuntu -- bash -c '
npm install -g @anthropic-ai/claude-code@2.1.112
'
```

Verify:

```bash
proot-distro login ubuntu -- claude --version
```

### 🔑 7. Create API-Key Manager

Return to normal Termux.

Create the command directory:

```bash
mkdir -p ~/bin
```

Create the API-key manager:

```bash
cat > ~/bin/set-agent-key <<'EOF'
#!/data/data/com.termux/files/usr/bin/bash

KEY_FILE="$HOME/.agentrouter_key"

clear

echo "======================================"
echo "      AgentRouter API Key Manager"
echo "======================================"
echo

read -rsp "Enter new AgentRouter API key: " KEY
echo

if [ -z "$KEY" ]; then
    echo
    echo "ERROR: API key cannot be empty."
    exit 1
fi

printf '%s\n' "$KEY" > "$KEY_FILE"
chmod 600 "$KEY_FILE"

echo
echo "API key saved successfully."
echo "No reinstallation is required."
EOF

chmod +x ~/bin/set-agent-key
```

The API key is stored locally:

```
~/.agentrouter_key
```

### 🌐 8. Create Claude + AgentRouter Launcher

```bash
cat > ~/bin/claude <<'EOF'
#!/data/data/com.termux/files/usr/bin/bash

KEY_FILE="$HOME/.agentrouter_key"

if [ ! -f "$KEY_FILE" ]; then
    echo "No AgentRouter API key configured."
    echo
    echo "Run:"
    echo "  set-agent-key"
    exit 1
fi

KEY=$(cat "$KEY_FILE")

if [ -z "$KEY" ]; then
    echo "AgentRouter API key is empty."
    echo
    echo "Run:"
    echo "  set-agent-key"
    exit 1
fi

TARGET_DIR="${1:-/root}"

proot-distro login ubuntu -- bash -c "
cd \"$TARGET_DIR\" 2>/dev/null || cd /root

export ANTHROPIC_BASE_URL='https://co.agentrouter.org'
export ANTHROPIC_AUTH_TOKEN='$KEY'
export ANTHROPIC_MODEL='claude-opus-5'
export CLAUDE_CODE_USE_AUTH_TOKEN='true'

exec claude
"
EOF

chmod +x ~/bin/claude
```

> **Model note:** `claude-opus-5` is used here because it was the model shown as allowed by the AgentRouter token in the setup process. If your token allows a different model, change only `ANTHROPIC_MODEL`.

### 🔗 9. Add Commands to PATH

```bash
echo 'export PATH="$HOME/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

Verify:

```bash
which claude
which set-agent-key
```

Both should point to:

```
/data/data/com.termux/files/home/bin/
```

### 🔐 10. Add Your First API Key

```bash
set-agent-key
```

Paste your AgentRouter API key. The key is saved locally and protected with:

```bash
chmod 600 ~/.agentrouter_key
```

⚠️ **Never publish your API key.** Do not put it in:

- GitHub
- README files
- Screenshots
- Public Telegram messages
- YouTube descriptions
- Source code
- `.env` files committed to Git

### ▶️ 11. Start Claude

After the first setup:

```bash
claude
```

The launcher automatically configures:

- `ANTHROPIC_BASE_URL`
- `ANTHROPIC_AUTH_TOKEN`
- `ANTHROPIC_MODEL`
- `CLAUDE_CODE_USE_AUTH_TOKEN`

### 📂 12. Start Claude in a Project

For Android storage:

```bash
claude /storage/emulated/0/Projects/MyProject
```

Example:

```bash
claude /storage/emulated/0/Download/MyProject
```

---

## 🔄 API KEY / CREDITS EXPIRED

This is the main purpose of this setup.

Suppose:

```
API KEY A
   ↓
Credits finished ❌
```

**DO NOT reinstall anything.** You do not need to reinstall:

- ❌ Termux
- ❌ Ubuntu
- ❌ Node.js
- ❌ npm
- ❌ Git
- ❌ Claude Code

Simply run:

```bash
set-agent-key
```

Enter your new API key, then:

```bash
claude
```

Done. ✅

---

## 🔁 API-Key Rotation

**Key A**

```bash
set-agent-key   # Enter Key A
claude          # Start
```

**Key A reaches its limit**

```bash
set-agent-key   # Enter Key B
claude          # Start
```

**Key B reaches its limit**

```bash
set-agent-key   # Enter Key C
claude          # Start
```

Installation happens only once.

---

## 🧪 Verification

```bash
# Check the launcher
which claude

# Check Ubuntu
proot-distro list

# Check Claude Code
proot-distro login ubuntu -- claude --version

# Check Node.js
proot-distro login ubuntu -- node -v

# Check npm
proot-distro login ubuntu -- npm -v

# Check Git
proot-distro login ubuntu -- git --version
```

---

## 🆘 Troubleshooting

**"claude: command not found"**

```bash
export PATH="$HOME/bin:$PATH"
hash -r
source ~/.bashrc
claude
```

**No API key configured**

```bash
set-agent-key
```

**API key quota finished**

```bash
set-agent-key
claude
```

Do not reinstall.

**403 Model Error**

If AgentRouter returns:

```
403
Model not authorized
```

the API key may not have access to the selected model. Check your AgentRouter token's allowed models and update `ANTHROPIC_MODEL`. You do not need to reinstall Claude Code for a model-access error.

---

## 🔒 GitHub Security

If you publish this project on GitHub, add a `.gitignore` with:

```
.agentrouter_key
.claude_api_key
.env
.env.*
*.key
*.token
node_modules/
npm-debug.log*
.DS_Store
```

Never commit real API credentials.

---

## ⚡ Quick Commands

| Task | Command |
|---|---|
| Start Claude | `claude` |
| Change API key | `set-agent-key` |
| Start project | `claude /storage/emulated/0/Projects/MyProject` |
| Enter Ubuntu | `proot-distro login ubuntu` |
| Exit Ubuntu | `exit` |
| Check Claude | `proot-distro login ubuntu -- claude --version` |
| Check launcher | `which claude` |

---

## 🎯 Final Workflow

**First time**

```bash
# Install
pkg update -y && pkg upgrade -y
pkg install -y proot-distro
termux-setup-storage

# Ubuntu
proot-distro install ubuntu

# Development tools + Claude
proot-distro login ubuntu -- bash -c '
apt update
apt install -y nodejs npm git
npm install -g @anthropic-ai/claude-code@2.1.112
'

# Configure API key
set-agent-key

# Start
claude
```

**Every normal day**

```bash
claude
```

**When AgentRouter credits finish**

```bash
set-agent-key   # Paste the new key
claude
```

That's it. No reinstallation. 🚀

---

## 👤 Credits

Created and maintained by **AREPALLY JASHWANTH**

Project: *Claude Code + AgentRouter — One-Time Termux Setup*

---

## 📺 YouTube

**Infinity Code Lab Official**
AI, coding, Termux, developer tools, tutorials and technology content.

## 📢 Telegram

**Infinity codes free**
Get project updates, tools, tutorials and community updates.

---

## ⭐ Support

If this project helped you:

- ⭐ Star the repository
- 📺 Subscribe to the YouTube channel
- 📢 Join the Telegram channel
- 🐛 Report bugs
- 💡 Suggest improvements

---

## ⚠️ Disclaimer

This is an independent community setup. It is **not** an official Anthropic product.

AgentRouter is a third-party API service. Model availability, API limits, credits, authentication, pricing and supported models may change. Always follow the current terms and documentation of the services you use.

---

**❤️ Made for Android Developers**

Termux + Ubuntu + Claude Code + AgentRouter

Turn your Android phone into a portable AI coding environment.

Install once. Change the API key when needed. Keep coding. 🚀
