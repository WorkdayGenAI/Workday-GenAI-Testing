# SA-GTS_UserStory_Businessprocess (Agent)

**Family:** GTS — *Generate Test Scenarios / Test Cases*
**Purpose:** Generate Workday **test cases** (CSV) from **two possible sources**:

1. A **pre-authored scenario library** in Quasar (BP-driven), or
2. A **raw user story** (description + acceptance criteria) — no repository lookup, Claude
   writes fresh test cases.

This lets the agent serve both "expand the curated library for these BPs" and "turn this
user story into test cases" requests.

---

## Tools

### `wdquasarbp_to_testcases_csv` — *library / BP path*
Retrieves curated scenarios per BP from Quasar and expands each into a test case via Claude
(sequential chunk loop).

| Parameter | Description |
|-----------|-------------|
| `payload` | Base JSON payload for the Quasar `searchvectors` API |
| `BusinessProcessName` | JSON array (or comma list) of BP names |
| `conversationid` | Included in the output filename |
| `fileinitialname` | Base name for the output CSV |

**Behaviour**
- Quasar auth (secret `wd-tool-secrets`), one shared `requests.Session`; per-BP vector
  search against `HCM_Test_Scenario_Library_Consolidated.xlsx`, then filters matches by
  `Workday BP` / `Scenario Name`.
- Chunks scenarios (**CHUNK_SIZE = 40**) and calls Claude (secret
  `SCA_WA_Claude_Opus_4_6`) per chunk with a `max_tokens` continuation loop; parses each
  Markdown table to a DataFrame.
- Concatenates per-BP results (prefixed with `Business Process Name`) →
  `artifacts/<file>_<timestamp>_<conversationid>.csv`; returns URL + processing summary.
- *(This is the single-threaded sibling of GTS's parallel `wdquasar_reposcenarios`.)*

### `wd_bp_quasar_to_workday_csv_us` — *user-story path*
Generates **fresh, original** test cases directly from user-story text — no Quasar lookup.

| Parameter | Description |
|-----------|-------------|
| `BusinessProcessName` | The BP name to attribute the cases to |
| `payload` | The user-story content (description + acceptance criteria) |
| `fileinitialname` | Base name for the output CSV |
| `conversationid` | Included in the output filename |

**Behaviour**
- Calls Claude (secret `SCA_WA_Claude_Opus_4_6`) with a prompt that explicitly says *do not
  copy existing scenarios — create fresh cases* from the story, generating "all possible
  scenarios" for the description and acceptance criteria.
- Output Markdown columns:
  `Business Process Name | Scenario ID | Test Scenario | Security Role | Test Data/Inputs | Scenario Description | Test Steps | Expected Result`
  (Scenario IDs like `US0001`, `US0002`).
- Parses to a DataFrame, validates required columns, writes
  `artifacts/<file>_<timestamp>_<conversationid>.csv`, returns URL.

### `workday_send_msg_to_canvas`
Posts the result back to the Canvas chat (short-token auth →
`POST /atr-gateway/genai/messages`; env-based secret selection, default `workday_CUI`;
supports extra environments such as `sapcui`, `platforms-dev`, `next-gen`).

---

## Typical flows

**From the scenario library**
`wdquasarbp_to_testcases_csv(...)` → CSV → `workday_send_msg_to_canvas`.

**From a user story**
`wd_bp_quasar_to_workday_csv_us(BusinessProcessName, payload=<story>, ...)` → CSV
→ `workday_send_msg_to_canvas`.
