<div align="center">

# Text2DB

**Chat with your database in natural language.**

Ask questions in plain English (or Spanish), and Text2DB turns them into SQL,
runs them against your database, and explains the results — always showing you
the exact query it executed.

[Download the latest release »](../../releases/latest)

</div>

---

> **This repository is the distribution & download page.** It hosts the packaged
> releases of Text2DB. The application source code is private and is **not**
> published here.

---

## Table of contents

- [What is Text2DB](#what-is-text2db)
- [Download & install](#download--install)
- [What the app contains](#what-the-app-contains)
- [Supported databases](#supported-databases)
- [Supported LLM providers](#supported-llm-providers)
- [Security model](#security-model)
- [Privacy & telemetry](#privacy--telemetry)
- [System requirements](#system-requirements)
- [Getting started](#getting-started)
- [Roadmap & future improvements](#roadmap--future-improvements)
- [License](#license)

---

## What is Text2DB

Text2DB is a **cross-platform desktop application** (Windows, macOS, Linux) that
lets you talk to a relational database in natural language. You configure a
connection, type a question in a chat window, and the app:

1. Reads your database schema (tables, columns, types, keys).
2. Asks a Large Language Model (LLM) to plan and write the SQL.
3. **Validates** the generated SQL — only read-only `SELECT` queries are allowed.
4. Executes it against your database.
5. Shows you the answer **together with the exact SQL that ran** and a results table.

It is designed for **analysts and developers** who want quick answers from their
data without writing SQL by hand — while keeping full transparency and control
over what actually gets executed.

**Bring your own key (BYO key):** Text2DB does not resell LLM access. You plug in
your own provider API key (or run a local model), so there is no recurring
subscription cost baked into the tool, and you stay in control of where your data
goes.

---

## Download & install

Grab the installer for your platform from the
**[Releases page](../../releases/latest)**:

| Platform | File | Notes |
|---|---|---|
| **Windows** | `Text2DB-<version>-win-x64.exe` | NSIS installer. Lets you choose the install directory and creates desktop/Start-menu shortcuts. |
| **macOS** | `Text2DB-<version>-mac-arm64.dmg` | Apple Silicon (arm64). Open the DMG and drag the app to Applications. |
| **Linux** | `Text2DB-<version>-linux-x64.AppImage` | Make it executable (`chmod +x`) and run it. |

### A note on security warnings

The current releases are **not yet code-signed (Windows) or notarized (macOS)**.
On first launch you may see a SmartScreen / Gatekeeper warning. This is expected
for an unsigned app — choose **"More info → Run anyway"** (Windows) or
**right-click → Open** (macOS) to proceed. Code signing and notarization are on
the [roadmap](#roadmap--future-improvements).

You will be asked to accept the **End User License Agreement (EULA)** during
installation.

---

## What the app contains

A single self-contained desktop app. Everything needed to run is bundled in the
installer — no extra runtimes, no command line, no Docker.

| Area | What's included |
|---|---|
| **Chat interface** | Conversational UI with message history, a "thinking" indicator, and a live reasoning ticker for models that expose it. |
| **Multi-database support** | Connect to PostgreSQL, MySQL, or MariaDB. Manage multiple saved connections and switch the active one. |
| **Multi-provider LLM** | Anthropic, OpenAI, Google Gemini, and local Ollama — pick the provider and model in settings. |
| **Schema introspection** | Automatically reads your tables, columns, types, and primary keys and gives the model the context it needs. |
| **Iterative reasoning** | The model explores your schema and refines its query over multiple steps before answering, instead of guessing in one shot. |
| **SQL safety validator** | Every query is parsed into an AST and checked before it runs. Only single-statement `SELECT`s pass. |
| **Read-only enforcement** | A connection check verifies the database user cannot write. |
| **Results table** | Paginated, sortable results (up to 1,000 rows per query in the UI). |
| **SQL transparency** | The exact SQL that ran is shown alongside every answer. |
| **Encrypted secrets** | API keys and database credentials are encrypted at rest using the OS keychain (Electron `safeStorage`), with an encrypted fallback when unavailable. |
| **Token & cost visibility** | Per-turn token usage is tracked and displayed so you can keep an eye on consumption. |
| **Bilingual UI** | English and Spanish, switchable in-app. |
| **Persistent chat history** | Conversations are saved locally between sessions. |
| **Security audit log** | Blocked/rejected SQL attempts are recorded locally for review. |

---

## Supported databases

| Engine | Driver | Status |
|---|---|---|
| **PostgreSQL** | `pg` | ✅ Supported |
| **MySQL** | `mysql2` | ✅ Supported |
| **MariaDB** | `mysql2` | ✅ Supported |

> DB2 and MongoDB are planned for a later phase (they require dedicated drivers
> and, for MongoDB, a different query language and validator).

---

## Supported LLM providers

You bring your own API key (or run a model locally). Configure the provider and
model in the **"LLM Model"** settings tab.

| Provider | Mode | Example models | Notes |
|---|---|---|---|
| **Anthropic** | Cloud, native tool-calling | Claude Opus / Sonnet / Haiku | Full agentic flow. |
| **OpenAI** | Cloud, native tool-calling | GPT-5.x family, GPT-4o, o3-mini | Full agentic flow. |
| **Google Gemini** | Cloud, native tool-calling | Gemini 3.x / 2.5 family | Supports reasoning summaries ("thoughts"). |
| **Ollama** | **Local**, no tool-calling | Any locally installed model | Fully private — data never leaves your machine. Uses a simpler generate-validate-execute flow. |

> **Privacy note:** With a cloud provider, parts of your schema (and sometimes
> sample data) are sent to that provider to generate the query. **Only with a
> local model (Ollama) does your data never leave your machine.** Choose
> accordingly for sensitive databases.

---

## Security model

Security is treated as non-negotiable. Several independent layers protect your data:

1. **Read-only by design.** You're guided to connect with a read-only database
   user, and the app verifies the user cannot perform writes.
2. **AST-based SQL validation.** Generated SQL is parsed (not regex-matched) and
   rejected unless it is a **single `SELECT` statement**. Blocked:
   `INSERT`/`UPDATE`/`DELETE`/`DROP`/`ALTER`/`TRUNCATE`/`GRANT`/`REVOKE`/`CALL`,
   multiple statements, modifying CTEs, `FOR UPDATE`/`FOR SHARE` locking clauses,
   `SHOW` statements, and a denylist of dangerous engine functions.
3. **Confined tool surface.** The model can only read your schema and run
   read-only queries — it has no file-system or shell access.
4. **Prompt-injection awareness.** Values read from the database are treated as
   untrusted data, not as instructions to the model.
5. **Encrypted credentials.** API keys and DB passwords are encrypted at rest
   via the OS keychain.
6. **Local audit log.** Rejected SQL is logged locally so you can see what the
   model attempted.

> ⚠️ **Text2DB is an assistance tool, not a source of truth.** LLMs can produce
> answers that look correct but answer a different question than the one you
> asked. Always review the SQL shown before acting on the results. See the
> [EULA](#license) for full terms.

---

## Privacy & telemetry

- **Your data stays between you and your chosen LLM provider.** Text2DB does not
  proxy or store your database contents.
- **Anonymous usage telemetry** is optional and **consent-based** — you're asked
  on first launch. It sends only anonymous usage metadata (app version, OS,
  provider/engine names, query latency buckets, error categories). It **never**
  sends your database data, SQL text, credentials, or chat content.
- Telemetry can be declined and is fully disabled if the build ships without
  analytics credentials configured.

---

## System requirements

- **Windows** 10/11 (x64), **macOS** (Apple Silicon / arm64), or **Linux** (x64).
- A reachable **PostgreSQL, MySQL, or MariaDB** database with a read-only user.
- One of:
  - An **API key** for Anthropic, OpenAI, or Google Gemini, **or**
  - A local **[Ollama](https://ollama.com)** installation with a model pulled.

---

## Getting started

1. **Install** the app for your platform (see [Download & install](#download--install)).
2. **Add a connection** in the *Connections* tab — host, port, database, and a
   **read-only** user. Test it.
3. **Configure an LLM** in the *LLM Model* tab — pick a provider, paste your API
   key (or point at your local Ollama), and choose a model.
4. **Ask a question** in the chat, e.g. *"How many customers signed up last month?"*
5. **Review** the answer, the SQL it ran, and the results table.

---

## Roadmap & future improvements

These are the directions we plan to take Text2DB. Top priority is **reducing
token usage and cost**, followed by broader provider/database coverage.

### 💰 Lower token consumption & cost

- **Smarter schema context.** Inject only relevant tables instead of the whole
  schema; let the model pull table detail on demand (it already can, but the
  initial context can be trimmed further).
- **Prompt caching.** Reuse the cached system prompt / schema across turns
  (e.g. Anthropic & Gemini prompt caching) so repeated context isn't re-billed.
- **Schema summarization & embeddings.** Pre-compute compact schema descriptions
  and use semantic retrieval to send only the tables a question actually needs.
- **Smaller-model routing.** Route simple questions to cheaper/faster models and
  escalate to a stronger model only when needed.
- **Result trimming.** Continue tuning how many rows/columns are fed back to the
  model between tool iterations.
- **A visible budget/cost meter** with per-session and per-question cost
  estimates and optional limits.

### 🔌 More LLM providers & integrations

- Additional cloud providers: **Mistral**, **Cohere**, **xAI (Grok)**,
  **DeepSeek**, **Groq**, and **OpenRouter** (one key, many models).
- **Azure OpenAI** and **Amazon Bedrock** for enterprise deployments.
- **Tool-calling for local models** (Ollama / llama.cpp) so local mode reaches
  feature parity with cloud providers.
- Per-connection provider/model overrides.

### 🗄️ More databases

- **SQLite**, **SQL Server**, **DB2**, **Snowflake**, **BigQuery**.
- **MongoDB** (requires a different query language and a dedicated validator).

### 🧠 Deeper analysis (the long-term differentiator)

- **Multi-step analysis:** chaining queries, cross-referencing datasets, and
  generating reports — not just single answers.
- **Charts & visualizations** generated from results.
- **Export** results to CSV / Excel.
- A **manual SQL editor** to tweak the generated query before running it.
- **Saved queries** and reusable analysis templates.

### 📦 Distribution & trust

- **Code signing (Windows)** and **notarization (macOS)** to remove install
  warnings.
- **Auto-update** so new versions install seamlessly.
- Published release notes and changelog per version.

> Have a request? Open an issue on this repository — provider and database
> requests help us prioritize.

---

## License

Text2DB is **free to use** under its
[End User License Agreement (EULA)](EULA.md). The application is proprietary and
closed-source; it bundles third-party open-source components, each under its own
license.

- The app is **licensed, not sold**, for personal or internal use, free of charge.
- Redistribution, reverse engineering, and derivative works are not permitted
  without prior written authorization.
- The software is provided **"as is"**, without warranty. Always verify results
  before relying on them.

Copyright © 2026 Iñigo Gastesi. All rights reserved.
