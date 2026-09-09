# 🤖 AI Dev Team: Autonomous Multi-Agent Software Engineering

An autonomous, multi-agent software engineering system powered by Groq's ultra-low latency LLM inference. The system orchestrates specialized AI agents representing key software engineering roles to collaboratively design, implement, test, review, self-optimize, and deploy software applications with human-in-the-loop oversight.

---

## 🚀 Quick Start Guide

### 1. Prerequisites
- **Python 3.10+** installed
- **Git** installed and on your system PATH
- A **Groq API Key** (free tier available at [Groq Console](https://console.groq.com/keys))

### 2. Installation

Clone or open the repository, create a virtual environment, and install dependencies:

```bash
# Navigate to the repository root
cd ai-dev-team

# Create and activate virtual environment
python -m venv venv

# Windows (PowerShell):
.\venv\Scripts\Activate.ps1
# Windows (Command Prompt):
.\venv\Scripts\activate.bat
# Linux / macOS:
source venv/bin/activate

# Install project dependencies
pip install -r requirements.txt
```

---

## 🔑 How to Set Your `GROQ_API_KEY`

The system requires a Groq API key to query high-speed inference models. You can configure it using either a `.env` file (recommended) or environment variables:

### Option A: Using `.env` (Recommended)
1. Copy `.env.example` to `.env`:
   ```bash
   cp .env.example .env
   ```
2. Open `.env` in any text editor and paste your API key:
   ```ini
   GROQ_API_KEY=gsk_your_actual_groq_api_key_here
   ```

### Option B: Terminal Environment Variable
If you prefer not using a file, export the key in your terminal before running:

- **Windows PowerShell:**
  ```powershell
  $env:GROQ_API_KEY="gsk_your_actual_groq_api_key_here"
  ```
- **Windows Command Prompt (CMD):**
  ```cmd
  set GROQ_API_KEY=gsk_your_actual_groq_api_key_here
  ```
- **macOS / Linux:**
  ```bash
  export GROQ_API_KEY="gsk_your_actual_groq_api_key_here"
  ```

---

## 🖥️ How to Run the Dashboard

Launch the interactive Streamlit user interface:

```bash
# Standard launch
streamlit run dashboard/app.py

# Or via Python module (recommended if 'streamlit' command is not in PATH):
python -m streamlit run dashboard/app.py
```

Once started, open your web browser to:
👉 **`http://localhost:8501`** (or the port indicated in your terminal)

### Dashboard Highlights:
- **Responsive Background Execution:** When you launch a feature or fix request, the 5-agent pipeline runs in a background thread without locking or freezing the Streamlit UI.
- **Disk as Single Source of Truth:** All task states and telemetry are persisted to `data/tasks/{task_id}.json` and `data/agent_runs.jsonl`. The UI reads directly from disk, ensuring no state is lost across reloads.
- **Omni-Present Human Approval Counter:** A high-contrast badge (`🔔 X tasks need your approval`) appears in the sidebar whenever tasks await your review.
- **Prominent Human Approval Gate:** Placed right at the top of the task detail view, displaying the feature request, test pass status, security review status, and the final code diff with instant **Approve** / **Reject** buttons.
- **Team Health & Reflection Centerpiece:** Switch to the `📈 Team Health` tab to see live Plotly rolling success curves for each agent, an underperforming agent table highlighted in red, and the **"Run Reflection Now"** button with visual prompt diffs (`old vs new`) showing how the system gets smarter over time.

---

## ☁️ Deploying on Streamlit Community Cloud

You can deploy this application directly to [Streamlit Community Cloud](https://streamlit.io/cloud) in just a few clicks:

1. **New App Settings**:
   - **Repository:** Select your GitHub repo
   - **Branch:** `main` (or `master`)
   - **Main file path:** `dashboard/app.py`
2. **Secrets Configuration**:
   Streamlit Cloud does not use `.env` files. In your app's **App Settings ➔ Secrets**, define your Groq key:
   ```toml
   GROQ_API_KEY = "gsk_your_actual_groq_api_key_here"
   ```
   The application automatically detects and reads `st.secrets["GROQ_API_KEY"]` as a fallback when `.env` is absent.
3. **⚠️ Ephemeral Storage Warning**:
   On Streamlit Community Cloud, local filesystem folders (`workspace/` and `data/`) are **ephemeral** and get wiped whenever the container restarts, sleeps, or redeploys. Treat Streamlit Cloud deployments as an interactive demo environment unless persistent external storage (e.g., S3 or database) is wired up.

---

## 👥 What Each Agent Does (In Plain English)

Each agent in the AI Dev Team has a focused, single responsibility mirroring a real-world software engineering team:

| Agent Role | Persona | What It Does (Plain English) |
| :--- | :--- | :--- |
| **Product Manager** (`pm_agent`) | **The Requirements Analyst** | Takes your raw feature request or bug report and turns it into 3–6 concrete, measurable **acceptance criteria**. It defines exactly what "done" looks like so downstream agents can be verified objectively. |
| **Architect** (`architect_agent`) | **The System Blueprint Designer** | Inspects your repository's file tree, designs a technical implementation approach, and scopes down the **minimal files** that need to change or be created. It keeps context focused so the coder doesn't get overwhelmed. |
| **Coding Agent** (`coding_agent`) | **The Software Developer** | Reads the scoped files and writes production-ready code implementing the architect's plan and PM criteria. Outputs a standard unified diff (`git diff`) and applies it cleanly to your target workspace. |
| **Testing Agent** (`testing_agent`) | **The Automated QA Engineer** | Writes automated `pytest` test suites tailored to each acceptance criterion and verifies whether the newly generated code actually satisfies the requirements, reporting matched criteria or specific assertion failures. |
| **Review Agent** (`review_agent`) | **The Security & Quality Auditor** | Performs an in-depth security and code quality audit on the diff and test results. It checks for SQL/command injection, hardcoded secrets, path traversal, input validation, and code style. If issues are found, it routes code back to Coding for a retry (up to 2 retries); otherwise, it escalates to human approval. |
| **Manager Agent** (`manager_agent` / `orchestrator`) | **The Team Lead & Coordinator** | Manages pipeline transitions, enforces retry limits, logs run telemetry (`data/agent_runs.jsonl`), calculates rolling success rates, and flags agents whose reliability dips below 60%. |
| **Reflection Agent** (`reflection_agent`) | **The Metacognitive Prompt Optimizer** | The system's self-healing engine. When an agent underperforms, Reflection analyzes the failure logs, diagnoses the root cause, and autonomously rewrites the agent's system prompt to eliminate recurring errors. |
| **Human in the Loop** (`human`) | **The Final Gatekeeper** | You! Review the executive summary, test verification, security review, and code diff at the top of the task detail view, then click **Approve** (to keep the live code) or **Reject** (to close the task). |

---

## 🏗️ Pipeline Flow Diagram

```text
[Human Request] ──► [PM Agent] ──► [Architect Agent] ──► [Coding Agent]
                                                               │
                                                               ▼
[Human Approval] ◄── [Review Agent] ◄── [Testing Agent] ◄──────┘
       │                    │
       │ (Reject)           │ (Retry Loop, max 2)
       ▼                    ▼
 [Task Closed]        [Coding Agent]
```

---

## 📁 Repository Directory Structure

```text
ai-dev-team/
├── agents/                  # Specialized autonomous agents
│   ├── pm_agent.py          # Requirements & acceptance criteria synthesis
│   ├── architect_agent.py   # Blueprint planning & file scoping
│   ├── coding_agent.py      # Code synthesis & git diff application
│   ├── testing_agent.py     # Automated pytest generation & verification
│   ├── review_agent.py      # Security audit & code quality gatekeeper
│   ├── manager_agent.py     # Analytics engine & underperformance detector
│   ├── manager_loop.py      # Manual and scheduled reflection cycle runner
│   └── reflection_agent.py  # Metacognitive prompt rewriter
├── core/                    # System foundation & orchestration
│   ├── schema.py            # Pydantic v2 data models & AgentTaskObject contract
│   ├── groq_client.py       # Resilient Groq SDK wrapper with JSON extraction
│   ├── orchestrator.py      # End-to-end pipeline runner & resume hooks
│   └── storage.py           # Disk persistence for tasks, runs, & prompt versions
├── dashboard/
│   └── app.py               # Streamlit web dashboard
├── prompts/
│   └── versions/            # Versioned prompt history and changelogs per agent
├── data/
│   ├── tasks/               # JSON task records (single source of truth)
│   └── agent_runs.jsonl     # Execution telemetry and duration metrics
├── workspace/               # Target workspace for generated projects
├── requirements.txt         # Dependencies
├── pyproject.toml           # Packaging metadata
└── README.md                # Project documentation
```

---

## 🛡️ Fault Tolerance & Error Handling

- **Resilient Inference:** The shared Groq client automatically retries transient rate limits and server errors with exponential backoff.
- **Graceful Degradation:** If an agent encounters an unrecoverable error or invalid response, the pipeline safely transitions `task.status = "blocked"` and logs the exact diagnostic error into the task history, ensuring the Streamlit application never crashes.
- **Auditable History:** Every agent step logs an ISO 8601 UTC timestamp and explicit `success: true/false` flag to `data/agent_runs.jsonl` and the task JSON.

---

## 📄 License
MIT
