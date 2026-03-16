# 🦾 Bare-AI-Agent: Autonomous Infrastructure Management
**Version:** 5.1.0-Enterprise (Hybrid Architect Edition)  
**Author:** Cian Egan  
**Date:** 2026-03-16  
**Repository:** [github.com/Cian-CloudIntCorp/bare-ai-agent](https://github.com/Cian-CloudIntCorp/bare-ai-agent)

Bare-AI-Agent is a multi-node, self-healing architecture designed to manage data pipelines and infrastructure integrity across Linux and Windows environments. The system now supports **dual AI engines** - choose between the sovereign Bare-AI-CLI or Google's Gemini-CLI.

---

## 🏛️ System Architecture

The fleet follows a strict role-based hierarchy to ensure safety and scalability:

### 1. The Architect (Dev Console)
**Primary Host:** `penguin` (Chromebook/Debian)  
**Role:** Central Command & Deployment.  
**Key Tools:**
- `bare`: Hybrid AI assistant with automatic engine detection (Bare-AI-CLI or Gemini-CLI)
  - `bare-gemini`: Force Gemini engine
  - `bare-sovereign`: Force Bare-AI-CLI engine
  - `bare-engine`: Show current active engine
- `bare-enroll`: The "Deployment Gun" used to push worker logic to remote nodes via SSH/Headscale
- `bare-status`: Local telemetry auditing tool

### 2. The Brain (Coordinator)
**Primary Host:** `bare-dc` (User: bare-ai-brain)  
**Role:** Autonomous fleet monitoring and decision-making.  
**Logic:** Runs high-frequency health checks and executes self-healing protocols.

### 3. The Workers (Fleet Nodes)
**Hosts:** `bare-rke2`, `bare-dc-headscale`, etc.  
**Role:** Payload execution and telemetry reporting.  
**Core Tool:** `bare-summarize` (Binary artifact for JSON telemetry harvesting)

---

## 🤖 Hybrid Engine Support

The Architect Console now supports two AI engines:

| Engine | Type | Use Case |
|--------|------|----------|
| **Bare-AI-CLI** | Sovereign, Local-First | Air-gapped environments, Vault integration, maximum control |
| **Gemini-CLI** | Cloud-Based | Google Cloud integration, latest models, broader knowledge |

**Automatic Detection:** The `bare` command intelligently detects which engine is available and routes accordingly (priority: Bare-AI-CLI → Gemini-CLI).

**Override:** `export BARE_ENGINE=gemini` or `export BARE_ENGINE=bare`

---

## 📜 The "Gold Standard" Naming Convention

To ensure enterprise-grade consistency, the repository follows these naming rules:

- **The Box (Installers):** Must have the `.sh` extension  
  *Example:* `setup_bare-ai-worker.sh`, `setup_bare-ai-dev.sh`

- **The Product (Tools):** Must have **NO extension**  
  *Example:* `bare-summarize`, `bare-enroll`, `bare-audit`  
  *Why:* This allows the underlying logic to be rewritten (e.g., from Bash to Python or Go) without breaking system calls.

- **The Repository:** `bare-ai-agent` (previously `Bare-ai`)

---

## 📁 Directory Structure

After installation, your local environment will have:
~/bare-ai-agent/ # Main repository (cloned from GitHub)
├── scripts/
│ ├── worker/ # Worker node installers & artifacts
│ │ ├── setup_bare-ai-worker.sh
│ │ └── bare-summarize
│ └── dev/ # Developer console installer
│ └── setup_bare-ai-dev.sh

~/.bare-ai/ # Runtime configuration (auto-created)
├── bin/ # Installed tools (added to PATH)
│ ├── bare-enroll
│ ├── bare-audit
│ └── bare-summarize
├── diary/ # Daily conversation logs
├── logs/ # JSON telemetry logs
├── config # Agent configuration (AGENT_ID, ENGINE_TYPE)
└── constitution.md # Core identity and operational rules

text

---

## 🚀 Quick Start: Setting Up the Architect Console

### On your Developer Machine (Penguin):

```bash
# 1. Clone the repository
git clone https://github.com/Cian-CloudIntCorp/bare-ai-agent.git ~/bare-ai-agent

# 2. Run the Architect setup
cd ~/bare-ai-agent/scripts/dev
./setup_bare-ai-dev.sh

# 3. Reload your shell
source ~/.bashrc

# 4. Verify installation
bare-status
bare-engine
🚀 Quick Start: Enrolling a New Worker
From the Architect Console (Penguin), run:

bash
bare-enroll <user@host_or_headscale_ip>
Example:

bash
bare-enroll bare-ai-brain@192.168.1.100
The worker node will prompt you to select your preferred AI engine during installation.

🔧 Daily Usage
As the Architect (on Penguin):
bash
# Start an AI session (auto-detects engine)
bare

# Force specific engine
bare-gemini      # Use Gemini
bare-sovereign   # Use Bare-AI-CLI

# Check engine status
bare-engine

# Deploy to a worker
bare-enroll bare-worker@10.0.0.25

# Check local telemetry
bare-status
Session Logging:
All AI conversations are automatically saved to ~/.bare-ai/diary/YYYY-MM-DD.md with engine-specific tagging (🤖 for Bare-AI, ✨ for Gemini).

🔒 Security Notes
The Architect runs on your local developer machine - never on production servers

Workers can run in Docker containers for enhanced isolation

All telemetry is logged locally in JSON format for audit trails

Engine choice is user-controlled - no data is sent to cloud unless you choose Gemini

📦 Dependencies
Component	Requirement	Notes
Bare-AI-CLI	Node.js, npm	npm install -g bare-ai-cli (coming soon)
Gemini-CLI	Node.js, npm	sudo npm install -g @google/gemini-cli
SSH	OpenSSH client	For remote enrollment
jq	JSON processor	For bare-status command
🆕 What's New in v5.1.0
✅ Dual engine support - Choose between Bare-AI-CLI and Gemini-CLI

✅ Automatic engine detection - No configuration needed

✅ Engine override aliases - bare-gemini and bare-sovereign

✅ Enhanced logging - Engine-specific tagging in diary entries

✅ Repository renamed - Now bare-ai-agent for clarity

✅ Path preservation - All existing scripts continue to work

📝 License
Apache-2.0 license

Key updates:
1. **Repository name changed** to `bare-ai-agent` throughout
2. **Version bumped** to 5.1.0 to match the hybrid engine support
3. **Added Hybrid Engine section** explaining the dual-engine capability
4. **Updated directory structure** to show `~/bare-ai-agent/` instead of `~/Bare-ai/`
5. **Added new commands**: `bare-engine`, `bare-gemini`, `bare-sovereign`
6. **Updated dependencies** to include both engine options
7. **Added "What's New" section** highlighting v5.1.0 features
8. **Updated date** to today (2026-03-16)