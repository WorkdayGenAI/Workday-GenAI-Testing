# GTS (Agent)

**Family:** GTS — *Generate Test Scenarios / Test Cases*
**Purpose:** Turn Workday **HCM Business Processes** into detailed, downloadable
**test cases** (CSV). This agent offers **two independent generation paths**:

1. **Library path** — expand pre-authored scenarios from the Quasar test-scenario library.
2. **BP-definition path** — read a Business Process's live step-by-step definition from
   Workday and derive a full scenario matrix from it (a two-stage pipeline).

Output CSV columns (BP-definition path):
`Business Process Name | Test Case ID | Test Scenario | Security Role | Test Data / Inputs | Test Case Description | Test Steps | Expected Result`

---

## Tools

### 1. `wdquasar_reposcenarios` — *library path*
Pulls curated scenarios for each BP from Quasar and has Claude expand each into a formal
test case, in **parallel**.

| Parameter | Description |
|-----------|-------------|
| `payload` | Base JSON payload for the Quasar `searchvectors` API |
| `BusinessProcessName` | JSON array (or comma list) of BP names |
| `conversationid` | Included in the output filename |
| `fileinitialname` | Base name for the output CSV |

**Behaviour**
- Auth to Quasar (secret `wd-tool-secrets`) → per-BP vector search against
  `HCM_Test_Scenario_Library_Consolidated.xlsx`.
- Filters returned scenarios so `Workday BP` / `Scenario Name` actually matches the BP.
- Chunks scenarios (**CHUNK_SIZE = 40**) and fans out Claude calls across a
  `ThreadPoolExecutor` (**MAX_WORKERS = 8**), each thread using its own `requests.Session`.
- Claude (secret `SCA_WA_Claude_Opus_4_6`) emits a Markdown table per chunk; a `max_tokens`
  continuation loop handles long output; chunk order is preserved.
- Concatenates all per-BP DataFrames → `artifacts/<file>_<timestamp>_<conversationid>.csv`
  and returns a download URL plus a per-BP processing summary.

### 2. `wd_bp_quasar_to_workday_xml` — *BP-definition path, stage 1*
Looks up each BP's definition and extracts its steps.

| Parameter | Description |
|-----------|-------------|
| `bp_input_json` | JSON `bp_list` array (or comma list) of BP names |
| `filename` | Base filename for the extracted-steps text file |
| `conversationid` | Included in the output filename |

**Behaviour**
- Quasar auth (secret `wd-tool-secrets`) → search `BP_Definition_HCM.xlsx` in index
  `wd_TSEndtoEndTesting_idx` to resolve each BP to a **reference ID** (matches
  `"<bp> (default definition)"`).
- Workday RaaS (secret `wd_dnt_10`): `GET …/customreport2/accenture_dpt10/…/
  JN_Business_Process_Definition_Detail` with the reference ID → BP definition **XML**.
- Claude extracts every `wd:Steps_group` verbatim into a 10-column Markdown table
  (order, step, type, triggers, To-Do, conditions, groups, alternate groups, completion,
  due-date flags).
- Writes the combined step details to `artifacts/<filename>_<conversationid>.txt` and
  returns a download URL. Heavily instrumented with `[DEBUG]` logging.

> The downstream tool expects the step file at `artifacts/XML_Details_<conversationid>.txt`.

### 3. `wd_bp_quasar_xml_to_csv_pll` — *BP-definition path, stage 2*
Reads the extracted step file and generates the final test-case matrix.

| Parameter | Description |
|-----------|-------------|
| `conversationid` | Locates the input file `artifacts/XML_Details_<id>.txt` |

**Behaviour**
- Splits the file into anonymous per-BP chunks (`=`-delimited header + Markdown step table).
- Fans out Claude calls in **parallel** (`ThreadPoolExecutor`, MAX_WORKERS = 8), each chunk
  processed independently and results reassembled in original order.
- The Claude prompt is a large, self-contained **test-case generator** that:
  - first **infers the BP name** (`INFERRED_BP:` line),
  - lists **every step** in `wd:Order` sequence as EXECUTE or SKIP (full step trace),
  - builds a **scenario matrix** (one TRUE + one FALSE row per business condition, plus
    MIN-PATH, MAX-PATH, completion, alternate-routing and approval rows),
  - rotates a single initiating security group per row,
  - enforces **plain business English** (never raw Workday syntax like `1=0?`).
- Renumbers `Test Case ID` per inferred-BP acronym, concatenates, and writes
  `artifacts/TestCaseGenerated_<conversationid>.csv`; returns URL + chunk summary.

### `workday_send_msg_to_canvas_wd`
Posts the result back to the Canvas chat (short-token auth →
`POST /atr-gateway/genai/messages`; env-based secret selection, default `workday`).

---

## Typical flows

**Library path**
`wdquasar_reposcenarios(...)` → CSV of test cases → `workday_send_msg_to_canvas_wd`.

**BP-definition path**
`wd_bp_quasar_to_workday_xml(...)` → `XML_Details_<id>.txt`
→ `wd_bp_quasar_xml_to_csv_pll(conversationid)` → `TestCaseGenerated_<id>.csv`
→ `workday_send_msg_to_canvas_wd`.

> The **GTS_TestCaseGeneration_BusinessProcess_Finance** agent is the same pipeline pointed
> at the Finance Quasar index/repository.
