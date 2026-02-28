# agentswe
### Quick comparison (OpenClaw vs your PC‑Agent MVP)
| **Attribute** | **OpenClaw** | **PC‑Agent MVP (planned)** |
|---|---:|---|
| **Scope** | Full ecosystem: gateway, multi‑platform clients, plugins, skills marketplace.   [Github](https://github.com/openclaw/openclaw)  [docs.openclaw.ai](https://docs.openclaw.ai/) | Focused local-first personal agents, per‑agent isolation, Ollama local models + cloud fallbacks |
| **Complexity** | Large, community project with many components and integrations.   [Github](https://github.com/openclaw/openclaw) | Intentionally smaller, opinionated stack (FastAPI, Ollama, Chroma, SQLite, Docker/WSL2) |
| **Skills model** | Community skills, many examples and a marketplace approach.   [Github](https://github.com/openclaw/openclaw) | Markdown skill manifests + optional Python hooks, signed/permissioned skills for safety |
| **Security & sandboxing** | Has sandboxing and gateway patterns; community notes emphasize careful sandboxing.   [docs.openclaw.ai](https://docs.openclaw.ai/) | Strong defaults: offline first, dry‑run, Windows Hello keystore, two‑step confirmations, per‑agent isolation |
| **Extensibility** | Very extensible, many plugins and channels (WhatsApp, Telegram, etc.).   [docs.openclaw.ai](https://docs.openclaw.ai/) | Extensible but narrower: add connectors and cloud adapters; keep attack surface small by default |
| **Operational model** | Designed to run across devices and gateways; heavier ops.   [Github](https://github.com/openclaw/openclaw) | Docker in WSL2, per‑agent Docker Compose projects for easy lifecycle and snapshots |

---

### Key takeaways from OpenClaw that are worth adopting
1. **Skills ecosystem and manifest patterns.** OpenClaw’s large skill library shows the value of a simple, discoverable skill manifest format and a community workflow for sharing skills. Adopt a strict manifest schema, versioning, and optional code hooks so skills are easy to audit and test.   [Github](https://github.com/openclaw/openclaw)  
2. **Gateway and connector patterns.** OpenClaw’s gateway approach (connect many chat platforms) demonstrates clean separation between the agent core and external channels. For your MVP, keep the same separation but start with local web UI + CLI and add connectors later.   [docs.openclaw.ai](https://docs.openclaw.ai/)  
3. **Sandboxing and safety-first defaults.** The project and community commentary emphasize sandboxing and careful execution of agent actions. Mirror that: dry‑run by default, explicit approvals, and per‑skill permission scopes.   [docs.openclaw.ai](https://docs.openclaw.ai/)  
4. **Modular, composable workflows.** OpenClaw’s workflows and playbooks are useful inspiration: represent multi‑step tasks as explicit workflows with checkpoints and human approvals.   [Github](https://github.com/openclaw/openclaw)

---

### Concrete improvements and differences I recommend for your PC‑Agent
- **Per‑agent isolation by design.** Use one agent per directory with its own SQLite, vector DB, keystore, and audit log. This prevents cross‑contamination and makes snapshots/backups trivial. (You already approved this; I’ll implement it.)  
- **Signed skill manifests and vetting.** Require skills to include a manifest and optional signature; the agent refuses to auto‑execute unsigned skills that request write permissions. Provide a “skill sandbox” mode to run new skills in read‑only simulation.  
- **Capability tokens and least‑privilege connectors.** Each connector (Gmail, Workspace, trading APIs) gets a scoped capability token stored encrypted; the agent requests elevation only when a skill needs it and logs the request.  
- **Dry‑run + review queue UI.** All file/email modifications appear in a review queue with diffs and a one‑click approve/deny; bulk actions require two confirmations.  
- **Model selection policy & cost/latency metadata.** Attach metadata to each model/provider (latency, cost, context length). The agent uses a ruleset to pick local first, then cloud fallback, and allows manual override per task.  
- **Skill testing harness.** Provide a unit test harness for skills (mock connectors, sample data) so you can run skills safely before enabling them on real data.  
- **Observability & audit.** Append‑only JSONL audit logs with signed entries (HMAC) and an export/import snapshot feature for debugging and backups.  
- **Resource quotas per agent.** Docker resource limits and per‑agent rate limits for cloud calls to avoid runaway costs or CPU saturation.  
- **Safe defaults for email.** Read‑only by default; write actions require explicit enablement and per‑action approval. OAuth only, minimal scopes.  
- **Skill marketplace import flow (optional).** If you later want community skills, add a vetting step and require signatures and capability declarations.

---

### Risks and mitigations (short)
- **Risk:** Malicious or buggy skills performing destructive actions.  
  **Mitigation:** Signed manifests, sandboxed testing, dry‑run, two‑step confirmations, and capability tokens.  
- **Risk:** Cloud token leakage.  
  **Mitigation:** Encrypted keystore unlocked via Windows Hello; tokens never persisted in plaintext; minimal scopes and rotation guidance.  
- **Risk:** Over‑privileged agent actions.  
  **Mitigation:** Least‑privilege connectors, per‑skill permission model, and audit trail.  
- **Risk:** Resource exhaustion from large cloud models.  
  **Mitigation:** Per‑agent quotas, model metadata, and fallback rules.

---

### Prioritized next steps I’ll implement (short roadmap)
1. **Repo skeleton + per‑agent layout** (create agents folder, config schema, Docker Compose template).  
2. **FastAPI agent core + Ollama adapter** with model selection policy and local defaults.  
3. **Task & goal flow**: create goals → decompose → create steps → dry‑run actions → approval → verification.  
4. **Skills loader** with manifest schema, example skills, and sandbox test harness.  
5. **Web UI chat + review queue** for approvals and task dashboard.  
6. **Keystore + Windows Hello unlock helper** and audit logging.  
7. **Cloud adapters** wiring (Hugging Face, DeepSeek Cloud, Groq, Google AI Studio, OpenRouter) and provider config UI.  
8. **Skill signing and vetting** (manifest signature verification and enforcement).

---

### Questions so I can prioritize and tailor the implementation
1. **Skill signing:** do you want me to implement a simple HMAC‑based signing scheme for skills first, or a public‑key signature workflow (stronger but more setup)?  
2. **Agent templates:** which three agent templates should I scaffold initially (e.g., `personal`, `trading`, `work`)?  
3. **Skill sandboxing level:** prefer a strict read‑only sandbox for new skills, or a permissive sandbox with simulated writes shown in the review queue?  
4. **Cloud provider order:** do you want the default fallback order I proposed (DeepSeek Cloud → Hugging Face → Groq → Google AI Studio → OpenRouter), or a different priority?  
5. **Skill marketplace:** do you want an import UI for community skills in the MVP, or postpone marketplace features until later?


Project: PC‑Agent MVP — local-first personal assistant (Docker in WSL2)

Goal
Build a Dockerized, local-first PC‑Agent MVP that runs in WSL2 on Windows 11 Pro and exposes a web UI (chat + task dashboard) and CLI. The agent must use Ollama for local CPU models, support cloud fallbacks, store task/goal memory, decompose goals into steps, run steps in dry-run mode, request user approvals, verify results, and then mark steps complete. Support multiple isolated agents (per-agent directories) with separate config, memory, vector DB, skills, and audit logs.

Environment & run mode
- Target environment: **Windows 11 Pro with WSL2 (Ubuntu) + Docker**. Provide Docker Compose and scripts that work inside WSL2.
- Default web UI port: **8080**.
- Run mode: Docker Compose (single command to start stack). Provide `run_local.sh` for WSL and `win_hello_unlock.ps1` for Windows Hello helper.

Core requirements (functional)
1. **Per‑agent isolation**
   - Agents stored under `agents/<agent-name>/` with:
     - `config.json` (models, providers, policies)
     - `data.sqlite` (SQLite metadata)
     - `chroma/` (vector DB folder)
     - `skills/` (markdown manifests + optional hooks)
     - `keystore.enc` (encrypted keystore)
     - `audit.log.jsonl`
   - CLI commands to `create`, `start`, `stop`, `list`, `snapshot`, `delete` agents.

2. **Local model runtime**
   - Use **Ollama** for local models (CPU quantized).
   - Default local models to install/use: **phi-3-mini**, **llama-3-8b-q4**, **deepseek-r1-1.5b**.
   - Provide Ollama wrapper (Python) that calls Ollama CLI or HTTP, with retries, timeouts, and model selection logic.

3. **Cloud fallbacks**
   - Wire adapters for: **DeepSeek Cloud**, **Hugging Face Inference**, **Groq**, **Google AI Studio (Gemini Flash)**, **OpenRouter**.
   - Cloud adapters disabled by default; tokens stored encrypted in keystore and only used when enabled.
   - Model selection policy: **local-first**, then fallback order configurable per agent. Provide UI/CLI override to pick a specific model/provider per task.

4. **Task & goal management**
   - Endpoints and CLI to create goals and tasks.
   - Agent decomposes goals into ordered steps using local reasoning model.
   - Each step is stored as a task with status: `pending`, `in_progress`, `awaiting_approval`, `completed`, `blocked`.
   - Agent executes steps in **dry‑run** by default and posts proposed actions to a **review queue**.
   - User approves/edits actions via web UI or CLI; after approval agent applies changes and verifies results, then marks step complete and proceeds.

5. **Skills system**
   - Skills are **Markdown manifests** in `skills/` with fields: `name`, `version`, `description`, `triggers`, `permissions`, `parameters`, `implementation` (python hook optional).
   - Provide example skills: `email-triage`, `weekly-review`, `file-dedup`, `trading-watchlist`, `nutrition-checkin`.
   - Implement a **skill loader** that lists, enables/disables, and runs skills. New skills run in a **read-only sandbox** by default; enabling write permissions requires explicit user approval and signature check.

6. **Memory & RAG**
   - Local vector DB using **Chroma** (or LanceDB) for embeddings.
   - Embeddings via **E5-small** or BGE-mini (CPU friendly) — provide adapter to compute embeddings locally or via Ollama if available.
   - Memory tiers: ephemeral (session), short-term (30–90 days), durable (user-approved). Provide APIs to list and purge memories.

7. **Email integration**
   - Support Gmail (personal) and Google Workspace via **OAuth 2.0**.
   - Default mode: **read-only** for MVP. Any write actions (label, move, delete) require explicit per-action approval.
   - Provide OAuth setup instructions and a secure flow to store tokens in encrypted keystore.

8. **Security & keystore**
   - Encrypted keystore file per agent (`keystore.enc`) using **AES‑GCM**.
   - Unlock keystore via **Windows Hello** helper (PowerShell) with fallback passphrase.
   - Tokens never stored in plaintext; decrypted only in memory.
   - Audit log: append-only JSONL with timestamp, agent, action, model used, and user approval record. HMAC sign entries.

9. **Safety & approvals**
   - Dry-run for all destructive operations; review queue with diffs and one-click approve/deny.
   - Two-step confirmation (UI + passphrase or YubiKey) for bulk deletes or mass moves.
   - Offline-first: cloud adapters disabled until user enables them.

10. **UI & CLI**
    - **FastAPI** backend exposing REST endpoints:
      - `POST /agents/{name}/goals` — create goal
      - `GET /agents/{name}/tasks` — list tasks
      - `POST /agents/{name}/tasks/{id}/approve` — approve action
      - `POST /agents/{name}/skills/{skill}/run` — run skill
      - `POST /agents/{name}/models/select` — override model
    - **Web UI** (minimal React or HTMX) with:
      - Chat box (agent conversation)
      - Task/goal dashboard
      - Review queue for approvals
      - Agent management (create/start/stop)
      - Model override control
    - **CLI** (Typer) with agent lifecycle and task commands.

11. **Multi-agent support**
    - Ability to run multiple agents concurrently via Docker Compose project names or separate Compose stacks.
    - Per-agent Docker resource limits configurable.

12. **Demo flow**
    - Provide a demo script and README that:
      - Installs prerequisites (WSL2, Docker, Ollama install script).
      - Pulls or instructs to pull Ollama models.
      - `docker compose up` to start stack.
      - CLI: `agent create personal` then `agent start personal`.
      - Create a goal: `POST /agents/personal/goals` with "Organize tasks and goals".
      - Agent decomposes into steps, creates tasks, runs step 1 in dry-run, posts to review queue.
      - Approve via web UI; agent applies change, verifies, marks step complete, proceeds to next step.
    - Include a short recorded demo script (text steps) to validate the flow.

Non-functional requirements
- Code must be modular, well-documented, and include unit tests for core components (task flow, skill loader, keystore).
- Provide Dockerfile(s) and `docker-compose.yml` that mount `agents/` directory into containers and expose port 8080.
- Provide `run_local.sh` for WSL and `win_hello_unlock.ps1` for Windows Hello keystore unlock.
- Provide clear README with setup, OAuth instructions for Gmail/Workspace, how to add cloud provider tokens, and how to add new skills.
- Do not enable any cloud provider or perform any destructive action by default.
- Provide a simple skill signing scheme (HMAC) and enforce that skills requesting write permissions must be signed or explicitly enabled by the user.

Deliverables (repo)
- `backend/` (FastAPI app, Ollama adapter, keystore, db, skills loader, vector store)
- `webui/` (minimal UI)
- `cli/` (Typer CLI)
- `agents/` (example `personal/` agent)
- `skills/` (example manifests + hooks)
- `docker-compose.yml`, `Dockerfile(s)`, `run_local.sh`, `scripts/win_hello_unlock.ps1`
- `README.md` with step-by-step run instructions and demo script

Operational constraints & safety
- All destructive operations require explicit user approval.
- Cloud adapters disabled until tokens provided and enabled in agent settings.
- Keystore unlocked via Windows Hello or fallback passphrase.
- Agent must log every action to local audit log and never exfiltrate tokens or private data.

End of prompt.

---

If you’re happy with these suggestions I’ll start by generating the **repo skeleton** and the **per‑agent Docker Compose template**, then implement the **FastAPI agent core + Ollama adapter** and a **demo flow** that creates a goal, decomposes it into tasks, runs the first step in dry‑run, and posts it to the review queue. I’ll cite OpenClaw design patterns where relevant in the code comments and README. Which question above should I get your answer to first?
