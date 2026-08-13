# GTS Agent (Generate Test Scenarios / Test Cases)

## What Is This Agent?

The **GTS agent** is a smart helper that automatically writes **software test cases for
Workday** — specifically for Workday **HCM (Human Capital Management) Business Processes**
like *Hire*, *Change Job*, *Terminate Employee*, and so on.

Think of it like this: normally a test analyst has to read how a business process works,
imagine every situation that could happen ("what if the manager approves?", "what if the
step is skipped?"), and then hand-write dozens of detailed test cases. That is slow and
easy to get wrong.

The GTS agent does that work for you. You tell it the name of a business process, and it:

1. Looks up how that process is built (either from a **library of pre-written scenarios**
   or from the **live process definition inside Workday**).
2. Asks an **AI model (Claude)** to turn that information into complete, ready-to-use test
   cases.
3. Hands you back a **spreadsheet (CSV file)** you can download.

**Why it exists:** to save testers hours of manual work and to make sure test coverage is
consistent and thorough.

**What problem it solves:** writing Workday test cases by hand is slow, repetitive, and
error-prone. This automates it.

**Who should use it:**
- Workday QA / test analysts
- Business process consultants
- Anyone who needs Workday HCM test cases quickly

---

## Quick Summary

| Item | Details |
|------|---------|
| **Purpose** | Automatically generate Workday HCM test cases from business process names |
| **Primary users** | Workday QA testers, test analysts, business consultants |
| **Inputs** | One or more Business Process names (e.g. `"Hire"`, `"Change Job"`); a base search payload; a conversation ID; a file base-name |
| **Outputs** | A downloadable **CSV** file of test cases + a summary of what was processed |
| **External systems** | **Quasar** (knowledge base / vector search), **Workday** (custom report / RaaS), **Claude** (AI model via an Anthropic-compatible HTTP endpoint), **Canvas** (the chat UI) |

---

## Key Features

### 1. Generate test cases from a scenario library
- **Description:** Pulls curated, pre-written test scenarios for a business process from the
  Quasar knowledge base and asks Claude to expand each into a full formal test case.
- **Tool:** `wdquasar_reposcenarios`
- **Example usage:** Ask the agent for test cases for `["Hire", "Promote Employee"]`.
- **Expected outcome:** A CSV where each row is a test case (scenario, security role, test
  data, steps, expected result), one download link, and a per-process summary.

### 2. Generate test cases directly from the live Workday process definition
- **Description:** Finds the process inside Workday, downloads its **step-by-step definition
  (XML)**, extracts every step, then builds an exhaustive **scenario matrix** of test cases.
- **Tools:** `wd_bp_quasar_to_workday_xml` (step 1) → `wd_bp_quasar_xml_to_csv_pll` (step 2)
- **Example usage:** Ask for full test coverage of `"Hire"` when you want every path tested.
- **Expected outcome:** A CSV covering every step of the process — including
  TRUE/FALSE condition paths, minimum/maximum paths, completion, alternate routing, and
  approval outcomes.

### 3. Parallel AI processing (fast)
- **Description:** When there are many scenarios, the work is split into chunks and sent to
  Claude **at the same time** (in parallel) instead of one after another.
- **Where:** `wdquasar_reposcenarios` and `wd_bp_quasar_xml_to_csv_pll` use a thread pool
  (`MAX_WORKERS = 8`, `CHUNK_SIZE = 40`).
- **Expected outcome:** Large jobs finish in roughly the time of a single AI call instead of
  many sequential calls.

### 4. Plain-English test cases (no confusing system jargon)
- **Description:** The AI prompts strictly forbid raw Workday system syntax (e.g. `1=0?`) in
  the output. Everything is translated into business language a human can read.
- **Expected outcome:** Test cases that testers and business users can follow directly.

### 5. Automatic "long answer" handling
- **Description:** If the AI's answer gets cut off because it is too long, the agent
  automatically says *"Continue from exactly where you stopped"* until the full table is
  produced.
- **Expected outcome:** No truncated / half-finished test-case tables.

### 6. Deliver results back into the chat
- **Description:** Posts the result message (with the download link) back into the Canvas
  conversation.
- **Tool:** `workday_send_msg_to_canvas_wd`
- **Expected outcome:** The user sees the answer in the same chat where they asked.

---

## How the Agent Works

Below is the complete end-to-end flow, in plain language.

1. **User interaction** — In the Canvas chat, the user asks for test cases and provides one
   or more business process names (plus, behind the scenes, a search payload, a conversation
   ID, and a file name).

2. **Request processing** — The relevant tool starts up. It reads its credentials from the
   platform secret store (using `core.get_secret`) and parses the list of business process
   names (accepts a JSON array, a JSON string, or a comma-separated string).

3. **Decision making (which path?)**
   - **Library path** → `wdquasar_reposcenarios`: use pre-written scenarios.
   - **BP-definition path** → `wd_bp_quasar_to_workday_xml` then
     `wd_bp_quasar_xml_to_csv_pll`: build test cases from the live Workday definition.

4. **Integrations** — The agent authenticates and calls the external systems it needs:
   - **Quasar** for search / lookup.
   - **Workday** to download the process definition XML (BP-definition path only).
   - **Claude** to generate the test cases.

5. **Data retrieval** — It collects the scenarios (library path) or the extracted process
   steps (BP-definition path) that will be the AI's "source of truth."

6. **Response generation** — Claude turns the source data into a **Markdown table** of test
   cases. Helper functions clean the text and convert the table into a pandas **DataFrame**,
   then into a **CSV** file.

7. **Output delivery** — The CSV is saved to artifact storage (`core.write_artifact`), a
   temporary **download link** is created (`core.generate_presigned_url`), and the result is
   returned. `workday_send_msg_to_canvas_wd` can then post it back into the chat.

### The two paths side by side

**Library path (1 tool):**
```
BP names → Quasar search (scenario library) → filter matches
        → Claude (parallel, per chunk) → CSV of test cases → download link
```

**BP-definition path (2 tools):**
```
Step 1: BP names → Quasar lookup (reference ID) → Workday report (definition XML)
        → Claude extracts steps → save XML_Details_<id>.txt

Step 2: read XML_Details_<id>.txt → split per BP → Claude (parallel) builds
        scenario matrix → CSV of test cases → download link
```

---

## Architecture Diagram
<img width="962" height="666" alt="image" src="https://github.com/user-attachments/assets/82a0d7ea-2014-46de-966a-4b1ed79af7d3" />

## Architecture Explanation

| Component | Purpose | Responsibilities | Inputs | Outputs | Dependencies |
|-----------|---------|------------------|--------|---------|--------------|
| **User (Canvas chat)** | The person requesting test cases | Provide BP names and parameters | — | A chat request | Canvas UI |
| **`wdquasar_reposcenarios`** | Library-path generator | Search Quasar, filter scenarios, call Claude in parallel, build CSV | `payload`, `BusinessProcessName`, `conversationid`, `fileinitialname` | CSV + download URL + summary | Quasar, Claude, `core` |
| **`wd_bp_quasar_to_workday_xml`** | BP-definition step 1 | Resolve reference ID via Quasar, fetch definition XML from Workday, extract steps via Claude, save a text file | `bp_input_json`, `filename`, `conversationid` | `XML_Details`-style `.txt` file + download URL | Quasar, Workday, Claude, `core` |
| **`wd_bp_quasar_xml_to_csv_pll`** | BP-definition step 2 | Read the step file, split per BP, generate scenario-matrix test cases via Claude in parallel, build CSV | `conversationid` | CSV + download URL + summary | Claude, `core` |
| **`workday_send_msg_to_canvas_wd`** | Chat responder | Authenticate to Canvas and post the result message | `conversation_id`, `message`, `environment`, `recipient_id` | Confirmation / error | Canvas API, `core` |
| **`core` (platform SDK)** | Platform glue | Secrets, artifact read/write, presigned URLs, per-conversation state | — | — | Host platform |
| **Quasar** | Knowledge base & vector search | Return scenarios / reference IDs | Auth token + query | JSON results | — |
| **Workday** | System of record | Return the BP definition XML via a custom report | Basic auth + reference ID | XML | — |
| **Claude** | AI model | Turn source data into a Markdown test-case table | Prompt | Markdown text | — |
| **Canvas** | Conversational UI | Show the result to the user | Token + message | Delivered message | — |

**How information flows:** the user's request enters a tool → the tool authenticates and
gathers source data from Quasar and/or Workday → Claude converts that data into a Markdown
table → the table becomes a CSV stored as an artifact → a download link is generated →
`workday_send_msg_to_canvas_wd` delivers it back to the chat.

---

## Project Structure

Actual contents of this agent folder:

```text
GTS/
├── wdquasar_reposcenarios          # Library path: scenarios -> test cases CSV (parallel)
├── wd_bp_quasar_to_workday_xml     # BP-definition step 1: Workday XML -> extracted steps .txt
├── wd_bp_quasar_xml_to_csv_pll     # BP-definition step 2: steps .txt -> test cases CSV (parallel)
├── workday_send_msg_to_canvas_wd   # Posts the result back into the Canvas chat
└── README.md                       # This document
```

> **Note:** the four tool files have **no `.py` extension**, but they are plain Python
> modules. Each file defines one primary function whose name matches the file name, plus
> private helper functions (prefixed with `_`).

| File | What it does | Why it exists | How it fits in |
|------|--------------|---------------|----------------|
| `wdquasar_reposcenarios` | Generates test cases from the pre-authored scenario library | Fast path when curated scenarios already exist | Standalone end-to-end generator |
| `wd_bp_quasar_to_workday_xml` | Extracts the real process steps from Workday | Needed when you want coverage derived from the live definition | Feeds its `.txt` output into the next tool |
| `wd_bp_quasar_xml_to_csv_pll` | Builds the full scenario matrix from extracted steps | Turns raw steps into exhaustive test cases | Second half of the BP-definition path |
| `workday_send_msg_to_canvas_wd` | Delivers the result into chat | Users interact through Canvas | Final delivery step |

---

## Technology Stack

Detected from the source code (`import` statements and usage):

| Technology | Where seen | Why it is used |
|------------|-----------|----------------|
| **Python** | All tool files | Implementation language |
| **`requests`** | All tools | HTTP calls to Quasar, Workday, Claude, Canvas |
| **`pandas`** | Generation tools | Build data tables and export CSV |
| **`concurrent.futures` (ThreadPoolExecutor)** | `wdquasar_reposcenarios`, `wd_bp_quasar_xml_to_csv_pll` | Run many AI calls in parallel (imported locally inside the function) |
| **`json`, `re`, `io`, `copy`, `datetime`, `uuid`, `typing`** | Various | Parsing, regex table cleanup, buffers, timestamps, IDs, type hints |
| **`core` (platform SDK)** | All tools | Secrets, artifact storage, presigned URLs — provided by the host platform |
| **Claude (Anthropic-compatible API)** | Generation tools | The AI model that writes the test cases (`/anthropic/v1/messages`, header `anthropic-version: 2023-06-01`) |
| **Quasar API** | `wdquasar_reposcenarios`, `wd_bp_quasar_to_workday_xml` | Vector search / knowledge base |
| **Workday RaaS (custom report)** | `wd_bp_quasar_to_workday_xml` | Source of the process definition XML |

> **Frameworks:** No web framework, LLM orchestration framework (LangChain / Semantic
> Kernel), containerization, or CI/CD configuration was found in this folder.
> **Unable to determine from repository analysis** what host platform registers and runs
> these tools (the `core` module is provided externally). *(Assumption: based on the URL
> paths `atr-gateway/...` and `acnopenai`, this appears to run on an Accenture GenAI
> platform, but this is not confirmed by files in the repo.)*

---

## Prerequisites

To run these tools you need the environment that supplies the `core` module and the
external services they call.

- **Language runtime:** Python 3 (uses f-strings, type-annotations; `pandas` `applymap`).
- **Python packages:** `requests`, `pandas`. *(A `requirements.txt` / dependency file was
  **not** found — Unable to determine exact pinned versions from repository analysis.)*
- **Host platform:** must provide the `core` SDK
  (`get_secret`, `get_artifact`, `write_artifact`, `generate_presigned_url`,
  `get_local_value`, `set_local_value`).
- **Cloud / accounts / API access (via secrets):**
  - Quasar access (secret `wd-tool-secrets`)
  - Claude access (secret `SCA_WA_Claude_Opus_4_6`)
  - Workday access (secret `wd_dnt_10`) — BP-definition path only
  - Canvas access (secrets `workday`, `workday_prod`, `workday_CUI`, `workday_stgtest`,
    `convoui_cui_dev`) — for the chat reply tool
- **Network requirements:** outbound HTTPS to the Quasar, Workday, Claude, and Canvas
  endpoints.

---

## Installation Guide

> **Important:** These files are **platform tools**, not a standalone app. There is **no
> `main()`, server, Dockerfile, or startup script** in this folder — Unable to determine a
> local standalone run process from repository analysis. The steps below describe getting
> the code and the (inferred) way it is consumed.

**Step 1 — Clone the repository**
```bash
git clone <your-repository-url>
cd Workday-GenAI-Testing-main/Workday-GenAI-Testing-main/GTS
```

**Step 2 — Install dependencies**
```bash
pip install requests pandas
```
*(No `requirements.txt` exists in the repo; install the packages the code imports. The
`core` module is provided by the host platform and is not pip-installable from here.)*

**Step 3 — Configure secrets / environment**
These tools do **not** read OS environment variables directly. They read named **secrets**
through `core.get_secret(...)`. You must register the following secrets in the host
platform's secret store (see the [Environment Variables](#environment-variables) table).

**Step 4 — Validate installation**
Confirm the host platform can import the tool file and that each required secret resolves.
*(No validation script is included — Unable to determine an official validation command from
repository analysis.)*

**Step 5 — Run the agent**
Register/invoke each tool through the host GenAI platform. See
[Running the Agent](#running-the-agent).

---

## Environment Variables

These tools use **named secrets** (via `core.get_secret`) rather than OS environment
variables. Each secret is a key holding a set of fields.

| Secret name | Required | Fields used (from code) | Used by | Description |
|-------------|----------|--------------------------|---------|-------------|
| `wd-tool-secrets` | Yes | `quasar_baseurl`, `quasar_username`, `quasar_password` | `wdquasar_reposcenarios`, `wd_bp_quasar_to_workday_xml` | Quasar base URL and login for search/auth |
| `SCA_WA_Claude_Opus_4_6` | Yes | `Endpoint`, `api_key`, `deployment_name` | all generation tools | Claude endpoint, API key, and model name |
| `wd_dnt_10` | Yes (BP-definition path) | `baseurl`, `username`, `password` | `wd_bp_quasar_to_workday_xml` | Workday base URL + Basic-auth login for the custom report |
| `workday` | Yes (default) | `url`, `username`, `password` | `workday_send_msg_to_canvas_wd` | Default Canvas environment credentials |
| `workday_prod` | If `environment=prod` | `url`, `username`, `password` | `workday_send_msg_to_canvas_wd` | Production Canvas |
| `workday_CUI` | If `environment=demo` | `url`, `username`, `password` | `workday_send_msg_to_canvas_wd` | Demo Canvas |
| `workday_stgtest` | If `environment=stagetest` | `url`, `username`, `password` | `workday_send_msg_to_canvas_wd` | Stage-test Canvas |
| `convoui_cui_dev` | If `environment=dev` | `url`, `username`, `password` | `workday_send_msg_to_canvas_wd` | Dev Canvas |

**Actual secret values:** "Unable to determine from repository analysis" (they live in the
platform's secret store, not in the code — which is correct and secure).

Non-secret constants hard-coded in the source (facts, not secrets):

| Constant | Value | Where |
|----------|-------|-------|
| Quasar auth path | `/atr-gateway/identity-management/api/v1/auth/short-token?useDeflate=true` | Quasar tools |
| Quasar search path | `/api/v2/acnopenai/searchvectors` | Quasar tools |
| Scenario library metadata | `HCM_Test_Scenario_Library_Consolidated.xlsx` | `wdquasar_reposcenarios` |
| BP-definition metadata | `BP_Definition_HCM.xlsx` | `wd_bp_quasar_to_workday_xml` |
| Quasar index | `wd_TSEndtoEndTesting_idx` | `wd_bp_quasar_to_workday_xml` |
| Workday report path | `/ccx/service/customreport2/accenture_dpt10/madhavi.vangara/JN_Business_Process_Definition_Detail` | `wd_bp_quasar_to_workday_xml` |
| Claude messages path | `/anthropic/v1/messages` | generation tools |
| `CHUNK_SIZE` | `40` | `wdquasar_reposcenarios` |
| `MAX_WORKERS` | `8` | `wdquasar_reposcenarios`, `wd_bp_quasar_xml_to_csv_pll` |
| Claude `max_tokens` | `16000` (reposcenarios) / `8000` (xml extract & csv step) | generation tools |
| Step-file name pattern (input to step 2) | `artifacts/XML_Details_<conversationid>.txt` | `wd_bp_quasar_xml_to_csv_pll` |

---

## Running the Agent

> These are functions invoked by the host GenAI platform, not CLI programs. The following
> shows how each tool is called conceptually (Python-style), using the parameters defined in
> the code.

**Local execution / development / production:** "Unable to determine from repository
analysis." No local runner, dev-mode flag, production server, Dockerfile, or Kubernetes
manifest exists in this folder. The tools run wherever the host platform executes them.

**Library path — generate test cases from the scenario library**
```python
wdquasar_reposcenarios(
    payload='{"index": "...", "search_limit": "...", "output": "..."}',  # base Quasar payload (JSON string)
    BusinessProcessName='["Hire", "Change Job"]',                        # JSON array or comma list
    conversationid="abc-123",
    fileinitialname="hire_testcases"
)
# -> {"file_name": ..., "download_url": ..., "successful_bps": N, "summary": [...]}
```

**BP-definition path — step 1 (extract steps from Workday)**
```python
wd_bp_quasar_to_workday_xml(
    bp_input_json='{"bp_list": ["Hire"]}',   # JSON with bp_list / BusinessProcessName, or comma list
    filename="XML_Details",                  # base name for the extracted-steps text file
    conversationid="abc-123"
)
# -> writes artifacts/XML_Details_abc-123.txt  and returns a download URL
```

**BP-definition path — step 2 (build the test-case matrix)**
```python
wd_bp_quasar_xml_to_csv_pll(
    conversationid="abc-123"   # must match step 1 so it can find XML_Details_abc-123.txt
)
# -> {"file_name": "TestCaseGenerated_abc-123.csv", "download_url": ..., "summary": [...]}
```

**Deliver the result to chat**
```python
workday_send_msg_to_canvas_wd(
    conversation_id="abc-123",
    message="Here is your test-case file: <download_url>",
    environment="demo",          # "" (default 'workday'), prod, demo, stagetest, dev
    recipient_id="user-xyz"      # required for stagetest/prod/demo
)
```

**Container-based execution / test execution:** "Unable to determine from repository
analysis." No containerization files and no test suite were found.

---

## Usage Examples

### Example A (non-technical) — "Give me Hire test cases from the library"
- **Input:** In chat, the user asks for test cases for the *Hire* process.
- **What happens internally:** `wdquasar_reposcenarios` logs into Quasar, searches the
  `HCM_Test_Scenario_Library_Consolidated.xlsx` library for *Hire* scenarios, keeps the ones
  that truly match, splits them into chunks of 40, and asks Claude (8 chunks at a time) to
  write a proper test case for each. The results are combined into one spreadsheet.
- **Output:** A CSV file (e.g. `hire_testcases_20260812_101500_abc-123.csv`) with columns
  like *Business Process Name, Test Scenario, Security Role, Test Data/Inputs, Scenario ID,
  Test Steps, Expected Result*, plus a download link.

### Example B (technical) — "Full coverage of Hire from the live Workday definition"
- **Input:** `bp_input_json = '{"bp_list": ["Hire"]}'`, `conversationid = "abc-123"`.
- **What happens internally (step 1):** `wd_bp_quasar_to_workday_xml` searches Quasar
  (`BP_Definition_HCM.xlsx`, index `wd_TSEndtoEndTesting_idx`) to find the reference ID for
  *"Hire (default definition)"*, calls the Workday custom report
  `JN_Business_Process_Definition_Detail` to download the definition **XML**, and asks Claude
  to extract every `wd:Steps_group` into a 10-column Markdown table. It saves
  `artifacts/XML_Details_abc-123.txt`.
- **What happens internally (step 2):** `wd_bp_quasar_xml_to_csv_pll` reads that file, splits
  it per business process, and (in parallel) asks Claude to build a full **scenario matrix** —
  one TRUE and one FALSE row per business condition, plus MIN-PATH, MAX-PATH, completion,
  alternate-routing, and approval rows — every step listed in order as EXECUTE or SKIP.
- **Output:** `TestCaseGenerated_abc-123.csv` with columns *Business Process Name, Test Case
  ID, Test Scenario, Security Role, Test Data / Inputs, Test Case Description, Test Steps,
  Expected Result*, plus a download link and a chunk-by-chunk summary.

### Example C — Deliver to chat
- **Input:** the download URL from Example A or B.
- **What happens internally:** `workday_send_msg_to_canvas_wd` gets a short-lived token for
  the chosen environment and posts the message to `/atr-gateway/genai/messages`.
- **Output:** `"Message successfully sent to the Canvas."` (or an error message).

---

## Dependencies

| Dependency | Purpose | Why it is needed | Criticality |
|------------|---------|------------------|-------------|
| `core` (platform SDK) | Secrets, artifacts, presigned URLs, local state | Nothing works without it | **Critical** |
| `requests` | HTTP client | All external calls (Quasar, Workday, Claude, Canvas) | **Critical** |
| `pandas` | Data tables + CSV export | Builds and writes the output files | **Critical** (generation tools) |
| `concurrent.futures` | Parallel execution | Speeds up large AI jobs | High (performance) |
| `json`, `re`, `io`, `copy`, `datetime`, `uuid`, `typing` | Std-lib utilities | Parsing, table cleanup, buffers, IDs, timestamps | Medium |
| Claude API | The AI model | Actually writes the test cases | **Critical** |
| Quasar API | Knowledge base | Source scenarios / reference IDs | **Critical** |
| Workday RaaS | System of record | Source of the definition XML (BP-definition path) | **Critical** (that path only) |

> **Version pinning:** "Unable to determine from repository analysis" — no dependency
> manifest file is present.

---

## APIs and Integrations

| Integration | Endpoint(s) (from code) | Auth | How it is used |
|-------------|-------------------------|------|----------------|
| **Quasar – auth** | `POST /atr-gateway/identity-management/api/v1/auth/short-token?useDeflate=true` | username/password → short-token | Get an API token for search |
| **Quasar – search** | `POST /api/v2/acnopenai/searchvectors` | `apiToken` header | Retrieve scenarios (library) or reference IDs (definition) |
| **Workday – custom report** | `GET .../ccx/service/customreport2/accenture_dpt10/madhavi.vangara/JN_Business_Process_Definition_Detail` | HTTP Basic auth | Download the BP definition XML for a reference ID |
| **Claude – messages** | `POST {Endpoint}/anthropic/v1/messages` (header `anthropic-version: 2023-06-01`) | `x-api-key` header | Generate/extract Markdown tables |
| **Canvas – messaging** | `POST {url}/atr-gateway/genai/messages` | short-token (`apiToken`) | Deliver the result into the chat |

- **Databases / message queues:** None found — "Unable to determine from repository
  analysis." Data is passed via artifact files and function return values.
- **Storage:** Artifact storage is accessed through `core.get_artifact` /
  `core.write_artifact` (paths under `artifacts/` and `uploaded_artifacts/`).

---

## Security and Authentication

Based only on what is visible in the code:

- **Authentication methods:**
  - Quasar & Canvas: username/password exchanged for a short-lived **token**, sent as an
    `apiToken` header.
  - Workday: **HTTP Basic auth** (`requests` `auth=(username, password)`).
  - Claude: **API key** in the `x-api-key` header.
- **Secrets management:** All credentials are fetched at runtime via
  `core.get_secret(<name>)`. **No credentials are hard-coded** in the source — this is good
  practice.
- **Authorization mechanisms:** "Unable to determine from repository analysis" — no
  role/permission checks are performed inside these tools; access is governed by the
  credentials in each secret and by the host platform.
- **Secure configuration practices observed:**
  - Timeouts are set on network calls (e.g. `timeout=30/60`, Claude `(10, 300)`).
  - Filenames are sanitized with regex before writing artifacts (`re.sub(r'[^A-Za-z0-9_\-]', '_', ...)`).
  - Raw responses are truncated in error output (e.g. `response.text[:300]`).
- **Potential concern (fact from code):** `wd_bp_quasar_to_workday_xml` prints Quasar
  credentials and tokens to logs via `print(...)` `[DEBUG]` statements (e.g. token length,
  base URLs). This is a logging-hygiene risk in production. *(See Assumptions, Risks and Gaps.)*

---

## Monitoring and Logging

- **Logging mechanism:** Plain `print(...)` statements. `wd_bp_quasar_to_workday_xml` and
  `wd_bp_quasar_xml_to_csv_pll` are heavily instrumented with `[DEBUG]`, `[WARNING]`, and
  `[ERROR]` prefixed messages tracing each step (token retrieval, per-BP status, chunk
  processing, previews of responses).
- **Structured summaries:** Each generation tool returns a `summary` / `processing_summary`
  object describing per-BP or per-chunk status (`Success`, `Partial Success`,
  `All chunks failed`, error reasons, counts). Operators can inspect this to see what
  succeeded or failed.
- **Monitoring tools / health checks / metrics collection:** "Unable to determine from
  repository analysis" — no dedicated monitoring, health-check endpoint, or metrics library
  is present. Observability relies on the host platform capturing stdout and the returned
  summaries.

---

## Testing

- **Available tests:** None found in this folder — "Unable to determine from repository
  analysis." There are no test files, no test framework configuration, and no coverage
  reports.
- **How to run tests:** Not applicable (no test suite present).
- **Coverage indicators:** None available.

*(Assumption: testing is likely performed manually via the Canvas chat and by inspecting the
generated CSVs, but this is not confirmed by any file in the repo.)*

---

## Troubleshooting

| Issue | Symptoms | Likely cause | Resolution |
|-------|----------|--------------|------------|
| **Missing input** | Returns `{"error": "BusinessProcessName is required"}` or `"payload is required"` | Required parameter not supplied | Provide `BusinessProcessName` and `payload` (and `bp_input_json` for the XML tool) |
| **Invalid payload** | `{"error": "Invalid JSON payload"}` | `payload` is not valid JSON | Pass a valid JSON string |
| **Quasar auth fails** | `"Quasar token generation failed: ..."` / `"Failed to generate Quasar token"` | Wrong/missing `wd-tool-secrets`, network issue, or Quasar down | Verify the secret fields and network access to the Quasar base URL |
| **No scenarios found** | Summary shows `"No matching scenarios after filter"` | The BP name doesn't match anything in the scenario library | Check spelling / try the exact Workday BP name |
| **Reference ID not found** | Summary shows `"Reference Not Found"` | Quasar didn't return `"<bp> (default definition)"` | Confirm the BP exists in `BP_Definition_HCM.xlsx`; check the name |
| **Workday call fails** | Summary shows `"Workday Failed"` | Bad `wd_dnt_10` credentials, report path changed, or Workday down | Verify the secret, the report URL, and Workday availability |
| **Claude credentials error** | `"Failed to retrieve Claude credentials: ..."` | `SCA_WA_Claude_Opus_4_6` missing/misconfigured | Register the secret with `Endpoint`, `api_key`, `deployment_name` |
| **Empty / no table from Claude** | Summary shows `"Claude returned empty response"` or `"No valid Markdown table"` | Model returned nothing or non-table text | Retry; check the model name and input size |
| **Step 2 can't find input file** | `wd_bp_quasar_xml_to_csv_pll` error: *Could not fetch file for conversationid…* | Step 1 not run, or a **different** `conversationid` used | Run `wd_bp_quasar_to_workday_xml` first with the **same** `conversationid` |
| **Unknown Canvas environment** | `"Unknown environment!"` | `environment` not one of the supported values | Use `prod`, `demo`, `stagetest`, `dev`, or leave blank for default |
| **Truncated output** | Table looks cut off | Model hit `max_tokens` | Handled automatically via the "Continue…" loop; if it persists, reduce input size |

---

## Assumptions, Risks and Gaps

**Facts (from the code):**
- The tool files have no `.py` extension but are Python modules.
- No dependency manifest, Dockerfile, Kubernetes manifest, CI/CD config, or test suite
  exists in this folder.
- The `core` SDK is provided by an external host platform.
- Credentials print statements exist in `wd_bp_quasar_to_workday_xml` (`[DEBUG]` logs of
  token length and base URLs; `wdsearchvector`-style tools elsewhere print credentials).
- The Workday report path and the Quasar index/metadata names are hard-coded.
- `wdquasar_reposcenarios` contains an unreachable line after `return` in the shared EIB
  template family; within GTS the generation tools return cleanly.

**Assumptions (clearly labelled):**
- *Assumption:* the platform is an Accenture GenAI environment (inferred from `atr-gateway`
  and `acnopenai` URL paths). Not confirmed by repo files.
- *Assumption:* tools are registered and invoked by that platform's agent runtime (there is
  no `main`/server here).

**Risks / gaps:**
- **Sensitive logging:** printing tokens / base URLs / credential-adjacent info to stdout is
  a security risk if logs are retained or shared. Recommend removing or masking.
- **Hard-coded Workday report owner path** (`.../madhavi.vangara/...`) couples the tool to a
  specific Workday tenant/owner; changes there will break the BP-definition path.
- **No automated tests** → regressions in the fragile Markdown-table parsing may go
  unnoticed.
- **No pinned dependencies** → behaviour may drift with `pandas` / `requests` upgrades
  (e.g. `DataFrame.applymap` is deprecated in newer pandas).
- **Two-step coupling:** step 2 depends on a file named `XML_Details_<conversationid>.txt`;
  if step 1 wrote a different base filename, step 2 won't find it. *(Fact: step 1's
  `filename` parameter controls the name, while step 2 looks specifically for
  `XML_Details_<id>.txt` / `xml_details_<id>.txt`.)*

---

## Business Value

- **Why it exists:** Writing Workday test cases by hand is slow and inconsistent. This agent
  turns a business process name into a complete, downloadable set of test cases in minutes.
- **Business benefits:**
  - **Speed:** parallel AI processing turns work that took hours into minutes.
  - **Consistency:** every process is documented the same way, in plain business English.
  - **Coverage:** the BP-definition path systematically covers condition paths, min/max
    paths, completion, routing, and approvals — reducing the chance a scenario is missed.
- **Productivity improvements:** testers spend less time writing boilerplate and more time
  reviewing and executing tests.
- **User impact:** faster test preparation, higher-quality Workday go-lives, and results
  delivered right inside the chat the user already works in.

---

## Executive Summary

The **GTS agent** automatically generates **Workday HCM test cases**. A user names a
business process (like *Hire* or *Change Job*) in a chat, and the agent produces a
ready-to-use spreadsheet of test cases.

It works in one of two ways: it either **expands a library of pre-written scenarios**, or it
**reads the live process definition from Workday** and builds an exhaustive set of test
cases from it. In both cases it uses the **Claude AI model** to write the test cases in
clear business language, processes large jobs **in parallel** for speed, saves the result as
a **downloadable CSV**, and delivers the link back into the chat.

**Who uses it:** Workday QA testers, test analysts, and business consultants.
**Why it is valuable:** it replaces hours of slow, repetitive manual test writing with a
fast, consistent, high-coverage automated process — helping Workday projects test more
thoroughly and go live with more confidence.

> Some operational details (host platform specifics, dependency versions, tests, deployment)
> are **not present in the repository** and are marked as *"Unable to determine from
> repository analysis"* throughout this document rather than guessed.
