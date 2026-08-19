# GTS_TestCaseGeneration_BusinessProcess_Finance (Agent)

**Family:** GTS — *Generate Test Scenarios / Test Cases*
**Purpose:** The **Finance** counterpart of the [GTS](../GTS/) agent. It turns Workday
**Finance Business Processes** into detailed **test cases** (CSV) using the same three-tool
pipeline, but pointed at the Finance knowledge base.

The tool logic is identical to GTS; only the Quasar targets (index / repository) and the
tool names (`*_fin` suffix) differ.

---

## Tools

### `wdquasar_reposcenarios_fin` — *library path*
Pulls curated Finance scenarios from Quasar and expands each into a test case via Claude,
in parallel.
- Params: `payload`, `BusinessProcessName`, `conversationid`, `fileinitialname`.
- Quasar auth via secret `wd-tool-secrets`; Claude via `SCA_WA_Claude_Opus_4_6`.
- **Finance-specific config (module constants):**
  - `QUASAR_INDEX = "wd_TSEndtoEndTesting_Finance_idx"`
  - `QUASAR_METADATA = "Vectorization_Repository___30th_Oct_2025.xlsx"`
  - `QUASAR_SEARCH_LIMIT = "200"`
- Chunks scenarios (CHUNK_SIZE = 40) and fans out across a `ThreadPoolExecutor`
  (MAX_WORKERS = 8); concatenates results into
  `artifacts/<file>_<timestamp>_<conversationid>.csv` and returns a download URL.

### `wd_bp_quasar_to_workday_xml_fin` — *BP-definition path, stage 1*
Resolves each BP to a reference ID via Quasar, pulls the BP definition **XML** from Workday
(secret `wd_dnt_10`, `JN_Business_Process_Definition_Detail` custom report), and uses Claude
to extract every step into a 10-column Markdown table.
- Params: `bp_input_json`, `filename`, `conversationid`.
- Writes the combined step details to an `artifacts/…txt` file for stage 2.

### `wd_bp_quasar_xml_to_csv_pll_fin` — *BP-definition path, stage 2*
Reads the extracted step file, splits it into per-BP chunks, and fans out Claude calls in
parallel to build the full scenario-matrix test cases (infer BP name → full ordered step
trace → TRUE/FALSE, MIN/MAX, completion, alternate-routing and approval rows → plain
English). Writes `artifacts/TestCaseGenerated_<conversationid>.csv` and returns a URL.
- Param: `conversationid`.

### `workday_send_msg_to_canvas_fin`
Posts the result back to the Canvas chat (short-token auth →
`POST /atr-gateway/genai/messages`; env-based secret selection, default `workday`).

---

## Typical flows

**Library path**
`wdquasar_reposcenarios_fin(...)` → CSV → `workday_send_msg_to_canvas_fin`.

**BP-definition path**
`wd_bp_quasar_to_workday_xml_fin(...)` → step-details txt
→ `wd_bp_quasar_xml_to_csv_pll_fin(conversationid)` → `TestCaseGenerated_<id>.csv`
→ `workday_send_msg_to_canvas_fin`.

> For full mechanics of each stage, see the [GTS README](../GTS/README.md) — this agent is
> the same pipeline with Finance-specific Quasar configuration.
