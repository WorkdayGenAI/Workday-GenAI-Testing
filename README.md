# Workday GenAI Testing

A collection of **GenAI agents** that automate Workday testing artifacts — test data,
test scenarios and test cases — for Workday HCM and Finance implementations.

Each top-level folder is a **separate agent**. An "agent" here is a bundle of Python
**tool functions** that are registered on a conversational GenAI platform (Accenture
GenWizard / ATR gateway, with a "Canvas" chat UI). A user talks to the agent in chat,
the agent's LLM decides which tool(s) to call, and the tools do the heavy lifting:
searching a knowledge base, calling Workday, invoking Claude, and producing a
downloadable file.

> **Note on file naming:** the tool files in each folder have **no `.py` extension**,
> but they are plain Python modules. Each file defines one primary function whose name
> matches the file name.

---

## Platform building blocks

Every tool is built from the same handful of platform primitives and external systems.

### The `core` module (platform SDK)
| Call | Purpose |
|------|---------|
| `core.get_secret(key)` | Fetch credentials/config from the platform secret store |
| `core.get_artifact(path, text=)` | Read an uploaded or generated file (`uploaded_artifacts/…`, `artifacts/…`) |
| `core.write_artifact(path, bytes)` | Save a generated file to the artifact store |
| `core.generate_presigned_url(name)` | Produce a temporary download link for a generated file |
| `core.get_local_value` / `set_local_value` | Per-conversation scratch state |

### External systems
- **Quasar** – an ACN vector-search / knowledge-base API
  (`/atr-gateway/identity-management/api/v1/auth/short-token` for auth,
  `/api/v2/acnopenai/searchvectors` for retrieval). Holds the pre-authored test
  scenario libraries and Workday BP definition repositories.
- **Workday** – accessed via **RaaS custom reports** (`/ccx/service/customreport2/…`)
  using HTTP Basic auth, returning JSON or XML.
- **Claude (LLM)** – called two ways:
  - An **Anthropic-compatible HTTP endpoint** (secret `SCA_WA_Claude_Opus_4_6`,
    `/anthropic/v1/messages`) — used by almost every generation tool.
  - **Claude Code CLI on AWS Bedrock**, driven inside a pseudo-terminal (secret
    `Bedrock Sonnet 4.5`) — used only by the *SA - Workday TestScenario CWB* agent.
- **Canvas** – the conversational UI. The `*send_msg_to_canvas*` tools post the
  agent's result back into the chat (`/atr-gateway/genai/messages`).

### Common patterns
- **Markdown → DataFrame → CSV/XLSX**: the LLM is prompted to emit a strict Markdown
  table; helpers parse it into a pandas DataFrame and export a file.
- **`max_tokens` continuation loop**: if Claude stops with `stop_reason == "max_tokens"`,
  the tool sends "Continue from exactly where you stopped" until the table is complete.
- **Parallel Claude calls**: the heavier pipelines chunk their input and fan out across
  a `ThreadPoolExecutor` (each thread gets its own `requests.Session`).

---

## The agents at a glance

| # | Folder (agent) | What it produces | Primary input | Key tools |
|---|----------------|------------------|---------------|-----------|
| 1 | **GTD-Assign Roles** | EIB test-data template (Assign Roles / TDW2) | Uploaded EIB Excel + markdown data | `wd_gtd_eib_template`, `WD_TDM_report_GenwizardOrg_Mapping` |
| 2 | **GTD-Candidate Data** | EIB test-data template (Candidate Data / TDW5) | Quasar search + EIB Excel | `wdsearchvector`, `wd_gtd_eib_template` |
| 3 | **GTD-Hire Data** | EIB test-data template (Hire Data / TDW6) | Quasar search + Workday reports + EIB Excel | `wdsearchvector`, `WD_TDM_report_Genwizard_Applicant_ids`, `wd_gtd_eib_template` |
| 4 | **GTD-Job Requisition** | EIB test-data template (Job Requisition / TDW1) | Quasar search + EIB Excel | `wdsearchvector`, `wd_gtd_eib_template` |
| 5 | **GTD-JR** | EIB test-data template (Job Requisition variant) | Quasar search + EIB Excel | `wdsearchvector`, `wd_gtd_eib_template1` |
| 6 | **GTD-Position Data** | EIB test-data template (Position Data / TDW4) | Quasar search + Workday report + EIB Excel | `wdsearchvector`, `WD_TDM_report_GenwizardOrg_Mapping`, `wd_gtd_eib_template` |
| 7 | **GTD-Post Job** | EIB test-data template (Job Posting / TDW3) | Quasar search + EIB Excel | `wdsearchvector`, `wd_gtd_eib_template` |
| 8 | **GTS** | Test **cases** CSV from Business Process definitions (HCM) | Business Process names | `wdquasar_reposcenarios`, `wd_bp_quasar_to_workday_xml`, `wd_bp_quasar_xml_to_csv_pll` |
| 9 | **GTS_TestCaseGeneration_BusinessProcess_Finance** | Same pipeline as GTS, tuned for **Finance** | Business Process names | `*_fin` variants of the GTS tools |
| 10 | **SA - Workday TestScenario CWB** | Test cases CSV via **Claude Code CLI (Bedrock)** | Uploaded BP-steps Excel | `gp_file_download_CM_WD` |
| 11 | **SA-GTS_UserStory_Businessprocess** | Test cases CSV from a repo **or** a raw user story | BP names / user-story text | `wdquasarbp_to_testcases_csv`, `wd_bp_quasar_to_workday_csv_us` |
| 12 | **Workday Test Scenario Generator** | Full scenario-matrix test cases CSV | Uploaded BP-steps XLSX | `wd_test_scenerio_generation_to_csv` |

> There is also a stray empty file `GTS_TestCaseGeneration_BusinessProcess` at the repo
> root — a placeholder with no content.

---

## The three agent families

### 1. GTD — "Generate Test Data" (agents 1–7)
These agents build **EIB (Enterprise Interface Builder) load templates** filled with test
data, one agent per Workday object / TDW pattern:

| Pattern | Output file | Agent |
|---------|-------------|-------|
| TDW1 | `Job_Requisition.xlsx` | GTD-Job Requisition / GTD-JR |
| TDW2 | `Assign_Roles.xlsx` | GTD-Assign Roles |
| TDW3 | `Job_Posting.xlsx` | GTD-Post Job |
| TDW4 | `Position_Data.xlsx` | GTD-Position Data |
| TDW5 | `Candidate_Data.xlsx` | GTD-Candidate Data |
| TDW6 | `Hire_Data.xlsx` | GTD-Hire Data |

**Flow:** (optionally) pull reference data from Quasar (`wdsearchvector`) and/or Workday
reports → the agent LLM produces a Markdown data table → `wd_gtd_eib_template` writes
that data into the uploaded EIB Excel workbook (matching columns by normalized name,
skipping the *Overview* sheet, writing from row 6) → returns a download link →
`workday_gtd_send_msg_to_canvas` posts back to chat.

### 2. GTS — "Generate Test Scenarios/Cases" (agents 8, 9, 11)
These agents turn **Workday Business Processes** into detailed **test cases**. Two source
strategies are used:
- **From a pre-authored scenario library** (`wdquasar_reposcenarios`,
  `wdquasarbp_to_testcases_csv`): retrieve curated scenarios from Quasar and have Claude
  expand each into a formal test case.
- **From the live BP definition** (`wd_bp_quasar_to_workday_xml` +
  `wd_bp_quasar_xml_to_csv_pll`): look up the BP, pull its step-by-step definition XML
  from Workday, extract the steps, then have Claude derive a full scenario matrix
  (TRUE/FALSE conditions, MIN/MAX paths, completion & approval variants) as test cases.

The **Finance** agent (9) is the same pipeline pointed at the Finance Quasar index and
repository. The **User Story** agent (11) adds a path that generates fresh test cases
directly from user-story text with no repository lookup.

### 3. Single-tool scenario generators (agents 10, 12)
These take an **uploaded spreadsheet of BP steps** and produce test cases in one shot:
- **Workday Test Scenario Generator** calls the Claude HTTP endpoint with a large,
  self-contained scenario-generation prompt.
- **SA - Workday TestScenario CWB** instead drives the **Claude Code CLI on Bedrock**
  inside a pseudo-terminal to do the same job.

---

## Output shape (test-case agents)

Most test-case tools emit a CSV with these columns:

```
Business Process Name | Test Case ID | Test Scenario | Security Role |
Test Data / Inputs | Test Case Description | Test Steps | Expected Result
```

Key prompt guarantees enforced across the GTS-family tools:
- **Plain business English only** — no raw Workday condition syntax (e.g. `1=0?`) in any cell.
- **Full ordered step trace** — every BP step appears in `wd:Order` sequence as EXECUTE or SKIP.
- **One security group per row**, rotated across initiating groups.
- **Scenario coverage** — one TRUE + one FALSE row per business condition, plus
  MIN-PATH, MAX-PATH, completion, alternate-routing and approval rows.

---

## Repository layout

```
Workday-GenAI-Testing-main/
├── README.md                                   ← this file
├── GTD-Assign Roles/                           ← agent 1  (see folder README)
├── GTD-Candidate Data/                         ← agent 2
├── GTD-Hire Data/                              ← agent 3
├── GTD-Job Requisition/                        ← agent 4
├── GTD-JR/                                     ← agent 5
├── GTD-Position Data/                          ← agent 6
├── GTD-Post Job/                               ← agent 7
├── GTS/                                        ← agent 8
├── GTS_TestCaseGeneration_BusinessProcess_Finance/ ← agent 9
├── SA - Workday TestScenario CWB/              ← agent 10
├── SA-GTS_UserStory_Businessprocess/           ← agent 11
├── Workday Test Scenario Generator/            ← agent 12
└── GTS_TestCaseGeneration_BusinessProcess      ← empty placeholder file
```

Each agent folder contains its own `README.md` documenting its tools in detail.
