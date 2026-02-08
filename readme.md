# 🛡️ SoliForge: Autonomous Smart Contract Security Agent

[![Powered by Qwen](https://img.shields.io/badge/AI-Qwen_Max-blue)](https://www.aliyun.com/)
[![Built with LangGraph](https://img.shields.io/badge/Agent-LangGraph-orange)](https://github.com/langchain-ai/langgraph)
[![Security](https://img.shields.io/badge/Security-Foundry_%2B_Slither-red)](https://getfoundry.sh/)
[![License](https://img.shields.io/badge/License-MIT-green)](LICENSE)

> **"Security on Autopilot."**
> SoliForge is an autonomous Red-Blue teaming system that detects, exploits, and fixes smart contract vulnerabilities automatically.

## 📖 Introduction

Smart contract security is a high-stakes "Dark Forest," where a single vulnerability can lead to millions in losses. Traditional static analysis tools are often noisy, and manual auditing is slow and expensive.

**SoliForge** transforms security auditing into a self-healing Continuous Integration process. It orchestrates a battle between two AI agents:

* **🔴 Red Agent (Attacker):** Analyzes code, writes executable **Foundry** exploits, and proves vulnerabilities exist.
* **🔵 Blue Agent (Defender):** Analyzes attack vectors and auto-patches the source code.

The system uses a **sandboxed Docker environment** to run regression tests until the contract is mathematically secure against all generated threats.

---

## 🚀 Key Features

* **🤖 Autonomous Red-Blue Teaming**: A stateful loop where AI agents attack and defend until convergence.
* **⚡ Auto-PoC Generation**:
    * Converts abstract Slither reports into executable Foundry scripts (`.t.sol`).
    * **Fuzzer Integration**: Automatically parses Fuzzer crashes (Counterexamples) and generates reproducible "replay" scripts (`Reproduce_Fuzz_Crash.t.sol`).
* **🛡️ Logic Inversion Validation**:
    * If an exploit runs **successfully** in Foundry -> The system flags it as a **FAILURE** (Vulnerable).
    * If an exploit **reverts/fails** -> The system flags it as a **PASS** (Secure).
* **📊 Real-time Mission Control**:
    * Visual "Attack Matrix" tracking active vs. mitigated threats.
    * Decoupled Fuzzing metrics (Failures / 500 Rounds).
* **🐳 Isolated Execution**: All compilations and tests run inside ephemeral Docker containers for safety and consistency.

---

## 🛠️ Architecture & Tech Stack

SoliForge is built on a Python backend orchestrated by **LangGraph**, managing a cyclic graph of agents.

### Backend

* **Core Logic**: Python 3.10+, FastAPI
* **Agent Orchestration**: LangGraph
* **LLM**: Alibaba Cloud **Qwen (DashScope)** (Optimized for code generation)
* **Database**: MySQL / SQLite (SQLAlchemy)
* **Security Engines**:
    * **Slither**: Static Analysis
    * **Foundry (Forge)**: Dynamic Fuzzing & Exploit Execution

### Frontend

* **Framework**: React 18 + Vite
* **Styling**: Tailwind CSS
* **Icons**: Lucide React
* **Visualization**: Custom Attack Matrix & Diff Viewer

### The Workflow

```mermaid
graph LR
    A[Start] --> B(Discovery: Slither + Fuzzer)
    B --> C(Red Agent: Weaponize Exploits)
    C --> D{Check Status}
    D -- Vulnerable --> E(Blue Agent: Fix Code)
    E --> F(Regression Validation)
    F --> B
    D -- Secure --> G[End Mission]
## ⚡ Getting Started

### Prerequisites

* **Docker** (Must be running; required for the Foundry sandbox)
* **Python 3.10+**
* **Node.js 18+**
* An API Key for **DashScope (Qwen)**

### 1. Backend Setup

```bash
# Clone the repository
git clone [https://github.com/your-username/soliforge.git](https://github.com/your-username/soliforge.git)
cd soliforge

# Create virtual environment
python -m venv venv
source venv/bin/activate  # or `venv\Scripts\activate` on Windows

# Install dependencies
pip install -r requirements.txt

# Configure Environment Variables
cp .env.example .env
# Edit .env and add your DASHSCOPE_API_KEY

# Run the backend
uvicorn src.main:app --reload --port 50550

```
### 2. Frontend Setup

```bash
cd frontend

# Install dependencies
npm install

# Start the dev server
npm run dev
```
Visit http://localhost:5173 to open Mission Control.

## 🧠 Engineering Highlights ("The Hacky Parts")

### 1. One-Shot Prompting for Reentrancy
LLMs often hallucinate invalid Reentrancy exploits (e.g., forgetting the `receive()` recursion). We injected a strict **"One-Shot Template"** into the Red Agent's system prompt (`temperature=0.1`). This forces the LLM to adhere to a specific `Attacker` contract structure, increasing compilation success rates from <30% to >90%.

### 2. Auto-PoC from Fuzzer
We don't just run `forge test`. The system parses the raw JSON output from Foundry, extracts the `counterexample` arguments from failed fuzz runs, and automatically generates a **Reproduction Script** (`Reproduce_Fuzz_Crash.t.sol`). This turns random probabilistic crashes into permanent, deterministic regression tests.

### 3. Logic Inversion in Validation
In a standard CI/CD pipeline, a "Passed Test" is good. In SoliForge's Red Team phase, a "Passed Test" means the exploit worked—which is **BAD** for security. We implemented a logic inversion layer in the `validate` node to correctly map Foundry results to Security Status (Active Threat vs. Mitigated).

---

## ⚠️ Current Limitations

> **Note on Generalization:**
> While SoliForge is designed as a general-purpose security agent, the current v1.0 prompt engineering is optimized for **Reentrancy & Bad Randomness scenarios** (e.g., Banking & Lottery contracts) to ensure high determinism and accuracy during the live demo.

* **Scope:** The Red Agent's "One-Shot" templates are currently tuned for logic errors and state inconsistencies.
* **Roadmap:** Future versions will implement **Dynamic Context Retrieval (RAG)** to index the entire Foundry documentation and Solidity patterns, allowing the agents to handle arbitrary contract logic and complex DeFi composability bugs without hardcoded templates.

---

## 📸 Screenshots

| Mission Control | Attack Matrix |
|:---:|:---:|
| ![Dashboard](./docs/dashboard.png) | ![Matrix](./docs/matrix.png) |
| *Real-time monitoring of Red/Blue agents* | *Visualizing active threats and fixes* |

*(Note: Please replace the image paths above with your actual screenshots)*

---

## 🤝 Contributing

Contributions are welcome! Please verify that all new features include unit tests and pass the Docker sandbox validation.

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.