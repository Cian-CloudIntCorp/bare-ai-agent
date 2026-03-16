🏛️ PROJECT BARE: Architectural Handover & Protocol
Target Audience: Autonomous Infrastructure Deployment AI & Human Engineers
Repository: https://github.com/Cian-CloudIntCorp/bare-ai-agent
Version: v5.1.0-Enterprise

1. The Architectural Paradigm
Project Bare is a Level 4 Autonomous, Sovereign IPv6 Overlay Network. It operates on a strict MAPE-K Loop (Monitor, Analyze, Plan, Execute, Knowledge).
The architecture strictly separates the Control Plane from the Data Plane:
PlaneComponentHostResponsibilityControlbare-brainBrain host (e.g. bare-dc)Analyze, Plan, Execute reflexesControlbare CLI / Architect ConsolePenguin (Dev Machine)Human engineering, Git, fleet managementDataWorker VMsFleet nodesMonitor (bare-summarize), receive & execute reflexes

2. Repository Structure
~/Bare-ai/
├── ARCHITECTURE.md
├── README.md
├── SECURITY.md
├── constitution.md          # Brain rule logic (Control Plane only)
├── fleet.conf               # Fleet IP list (repo copy)
├── scripts/
│   ├── brain/
│   │   ├── bare-brain-compiled    # Lightweight brain (no Vault)
│   │   └── setup_bare-brain.sh   # Brain installer (Vault-integrated v5.0.1)
│   ├── dev/
│   │   └── setup_bare-ai-dev.sh  # Architect Console installer (Penguin)
│   ├── worker/
│   │   ├── bare-summarize         # Telemetry sensor artifact
│   │   └── setup_bare-ai-worker.sh # Worker node installer
│   └── windows_alpha/             # Windows node support (in development)

3. Component Descriptions
🧠 bare-brain (Control Plane)
The autonomous MAPE-K loop engine. Two variants exist:
VariantFileUse CaseVault-Integratedinstalled via setup_bare-brain.shProduction — fetches Gemini API key from HashiCorp Vault at runtime, never stored on diskCompiled (lightweight)bare-brain-compiledDev/testing — uses ambient GEMINI_API_KEY env var
Brain execution flow:

Harvest — SSH into each worker in fleet.conf, run bare-summarize, collect JSON telemetry
Circuit Breaker — Skip any worker already reflexed in the current hour
Analyze — Send telemetry + constitution to Gemini. Extract COMMAND: from response
Spinal Cord Fallback — If Gemini is unreachable, apply hardcoded rules (e.g. restart inactive rke2-server)
Reflex — SSH back into the worker and execute the fix command
Log — Append timestamped entry to ~/.bare-ai/reflex_history.log

🖥️ Architect Console (Penguin — Dev Machine)
Installed via scripts/dev/setup_bare-ai-dev.sh. Transforms a developer machine into the fleet control centre.

Installs bare-enroll — deploys the worker installer to remote nodes via SSH/SCP
Installs bare-audit — local telemetry check (alias for bare-summarize)
Writes the Architect constitution.md to ~/.bare-ai/
Installs the bare() hybrid CLI function with auto engine detection:

EngineCommandTriggerBare-AI-CLI (Sovereign)bare or bare-sovereignAuto-detected if bare-ai binary presentGemini-CLIbare or bare-geminiAuto-detected if gemini binary presentForce overrideexport BARE_ENGINE=bare|geminiManual
👷 Worker Node (bare-summarize)
The telemetry sensor. Harvests local node data and outputs structured JSON for the Brain to consume. Installed to ~/.bare-ai/bin/bare-summarize on each worker.
Workers are passive — they do not think, loop, or initiate. They only:

Report (scraped by bare-brain via SSH)
React (receive and execute reflex commands via SSH)


4. GitHub Pull Manifest (For VM Templates)
When constructing a VM template from this repository, strictly filter components:
✅ REQUIRED (Pull to VM Template)

bare-summarize — the telemetry sensor
scripts/worker/setup_bare-ai-worker.sh — worker installer
Public SSH keys (.pub) required for Brain→Worker access

❌ FORBIDDEN (Do NOT put on VM Template)

bare-brain / bare-brain-compiled — Control Plane only
setup_bare-brain.sh — Control Plane only
setup_bare-ai-dev.sh — Developer machine only
constitution.md — Brain rule logic, Control Plane only
fleet.conf — Brain targeting list, Control Plane only
Any private keys (id_ed12345), .env files, or API keys
~/.bare-ai/config/vault.env — Vault credentials, never on workers


5. Worker Node Security & Configuration Posture
The VM template must be prepared to receive commands from the Brain via the "No-Touch" SSH gap:

Identity: Create the standard bare-ai user
Access Control: Configure /etc/ssh/sshd_config to whitelist the user (AllowUsers bare-ai)
The Effector: Grant passwordless sudo for specific self-healing commands in /etc/sudoers:

   bare-ai ALL=(ALL) NOPASSWD: /bin/systemctl restart rke2-server

Artifact: Ensure bare-summarize is installed and executable at ~/.bare-ai/bin/bare-summarize


6. Fleet Configuration
FileLocationPurposeRepo copy~/Bare-ai/fleet.confVersion-controlled source of truthRuntime copy~/.bare-ai/fleet.confRead by bare-brain at execution time
Format — one IPv6/IPv4 address per line, comments with #:
# Example
100.64.0.3

7. End-State Goal
When a template is cloned into a live VM, it should:

Boot silently
Connect to the overlay network
Wait to be scraped by the centralised bare-brain service


It does not think. It only reports and reacts.


8. ⚠️ Critical Clarification: Developer Tools vs. Production Services
ToolTypePermitted on Workers?bare-brain autonomous loopProduction service❌ FORBIDDENbare / Gemini-CLI interactiveHuman dev CLI✅ PERMITTED (admin/debug use)bare / Bare-AI-CLI sovereignHuman dev CLI✅ PERMITTED (admin/debug use)bare-enrollDev machine only❌ Not applicable to workers
The autonomous bare-brain loop MUST NOT run on worker nodes. The interactive bare CLI IS PERMITTED on worker nodes for human administrative and debugging purposes only.

9. Vault Integration (Brain v5.0.1+)
The production brain fetches its Gemini API key from HashiCorp Vault at runtime using AppRole authentication:

Reads ~/.bare-ai/config/vault.env for VAULT_ROLE_ID, VAULT_SECRET_ID, VAULT_ADDR
Authenticates via POST /v1/auth/approle/login → receives a short-lived token
Reads secret from secret/data/bare-ai/brain → key loaded into memory as GEMINI_API_KEY
Key is never written to disk


The bare-brain-compiled variant skips Vault and uses an ambient GEMINI_API_KEY env var — suitable for dev/testing only.