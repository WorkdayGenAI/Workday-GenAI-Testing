# GTS — Workday Generate Test Scenario

## Contents (click to jump)

1. [What Does This Do?](#section-1--what-does-this-do)
2. [The Story of How It Works](#section-2--the-story-of-how-it-works)
3. [What You Get Out of It](#section-3--what-you-get-out-of-it)
4. [What Can It Do? (The Five Helpers)](#section-4--what-can-it-do-the-five-helpers)
5. [The Big Picture](#section-5--the-big-picture)
6. [Before You Start (Your Checklist)](#section-6--before-you-start-your-checklist)
7. [How to Set It Up](#section-7--how-to-set-it-up)
8. [Settings You Need to Fill In](#section-8--settings-you-need-to-fill-in)
9. [What Could Go Wrong](#section-9--what-could-go-wrong)
10. [Why This Matters](#section-10--why-this-matters)
11. [Known Limitations and Watch-Outs](#section-11--known-limitations-and-watch-outs)
12. [Word List (Plain-English Dictionary)](#word-list-plain-english-dictionary)

---

## Section 1 — What Does This Do?

> **Why read this?** In 30 seconds you'll know what this thing is and whether it's worth caring about.

Imagine you had to sit down and hand-write a checklist of every single way to test a piece of
HR software — "what if the manager approves?", "what if a step is skipped?", "what if the new
hire is part-time?" — and write out fifty or a hundred of these, one careful step at a time.
That takes a skilled person **hours per process**, and it's easy to forget a case.

**This tool does that writing for you, in minutes.** You tell it the name of a
**business process** (a named task inside Workday — the big HR and finance software companies
use to hire people, change jobs, and run payroll), and it hands you back a finished
**spreadsheet** (a table you can open in Excel) full of ready-to-use test instructions.

Before this existed, testers wrote everything by hand. Now a robot writes the first full draft
and a human just reviews it. **Who uses it?** Software testers, quality checkers, and business
analysts on Workday projects — mostly at Accenture.

---

## Section 2 — The Story of How It Works

> **Why read this?** So you understand what's happening "behind the curtain" when you press go — no surprises.

Here is the journey, told as a story. There are a few different methods into the same use case, so
we'll walk through each one.

### 🔍 Method 1 — "Look it up in the library"

When a tester types *"Generate test cases for Hire Employee"* into the company chat window, the
tool searches a big **searchable filing cabinet** (an internal Accenture search system called
**Quasar** that finds things by meaning, not just exact spelling) and pulls out every
pre-written scenario that matches "Hire Employee." It then hands those scenarios to **Claude**
(the artificial-intelligence writing assistant, built by Anthropic — this is the "brain" that
actually writes the words). To go faster on big jobs, it splits the work into small piles and
sends **several piles to Claude at once** — like having eight assistants each write a different
chapter of a book at the same time instead of one person writing all of it.

### 📖 Method 2 — "Read the official Workday rulebook"

If no pre-written notes exist, the tool can go and read the **live rulebook straight from
Workday itself** — the official, step-by-step definition of how a process like "Hire Employee"
actually runs. This path is a **two-stage journey**:

1. **Stage 1 — Fetch the rules.** The tool first asks **Quasar** to look up the exact
   reference ID for "Hire Employee", then uses that ID to call **Workday's own report** and
   download the full process definition (in technical XML format). Claude reads this raw
   definition and extracts all the steps into a neat text file, which gets saved for stage 2.

2. **Stage 2 — Write the test cases.** A second tool picks up that text file (matched by the
   same chat ID so it finds the right one) and hands the extracted steps to Claude again.
   Claude now generates the full set of test cases — covering every yes/no branch, shortest
   path, longest path, completion step, and approval outcome — and produces the final
   spreadsheet.

> **Why two stages?** Because the Workday rulebook can be very large and complex. Splitting the
> work into "understand the rules" and then "write the tests" gives Claude the best chance of
> being thorough and accurate.

### ✍️ Method 3 — "Write from a user story"

If you have a **user story** (a short plain-English description of what someone needs, written
like *"As an HR Partner, I want to transfer an employee so the org chart updates"*), you can
paste that directly in. The AI **invents brand-new test cases from scratch** — no filing
cabinet or Workday connection needed.

### 📄 Method 4 — "Upload a spreadsheet"

You can also **upload a spreadsheet** that lists the steps of a process (exported from
Workday). The tool reads the file, hands the step data to Claude, and gets back the finished
test cases. Two versions of this exist — one uses Claude over the web (the standard way), and
one uses a bigger, more industrial engine (see Helper 5 below).

---

Either way, once the AI finishes, Claude writes each situation up as a proper test case: who
does it, what they type in, the exact steps, and what the correct result should look like — all
in plain business English, with no confusing computer codes. If Claude's answer ever gets cut
off for being too long, most helpers politely say *"keep going from where you stopped"* until
the whole thing is finished.

Finally, all the pieces are stitched together into one **spreadsheet file**, saved safely, and a
**temporary download link** (a private web address that stops working after a while, like a
locker that auto-locks) is sent back into the chat for you.

---

## Section 3 — What You Get Out of It

> **Why read this?** So you know exactly what lands in your hands at the end — no vague "output."

At the end you receive **a download link in the chat**. Clicking it gives you a **CSV file**
(a plain spreadsheet that opens in Microsoft Excel or Google Sheets).

Inside that spreadsheet, **each row is one test case**, and the columns are roughly these:

- **Business Process Name** — which Workday task this test is for (e.g. "Hire Employee")
- **Test Case ID / Scenario ID** — a short code so each test can be tracked (e.g. `HE0001`)
- **Test Scenario** — a short title saying what this row is checking
- **Security Role** — which kind of user runs this test (e.g. "HR Partner", "Recruiter")
- **Test Data / Inputs** — the exact values a tester should type in
- **Test Case Description** — a couple of sentences on what makes this case special
- **Test Steps** — the full numbered list of actions, in order, start to finish
- **Expected Result** — what the tester should see after each step if it worked

> **Note:** The exact column names vary slightly between helpers (for example, the user-story
> helper calls one column "Scenario ID" starting with `US0001`, while the upload helper adds a
> "Business Process Name" column). The overall structure is always the same.

The file is named with the date, time, and a chat ID so it never overwrites an old one, for
example: `HireEmployee_20260819_101500_abc123.csv`.

---

## Section 4 — What Can It Do? (The Five Helpers)

> **Why read this?** The family has five members. This tells you which one to reach for.

All five do "turn a Workday process into test cases," but they differ in **where they get their
information** and **how they talk to the AI**. Here they are in friendly terms.

### Helper 1 — "Expand the Ready-Made Library" (HR version)
**Folder:** `GTS Business Process`

You give it a business process name. It searches the **filing cabinet** for pre-written HR
scenarios and asks the AI to turn each one into a full test case — several at a time for speed.
*Example:* ask for `["Hire", "Promote Employee"]` and get one spreadsheet covering both.

> This folder contains **four tools** that work together:
>
> | Tool | What it does |
> |------|-------------|
> | `wdquasar_reposcenarios` | **Library path** — searches Quasar for pre-written scenarios, sends them to Claude in parallel, produces a CSV of test cases. |
> | `wd_bp_quasar_to_workday_xml` | **Workday path, Stage 1** — looks up the process name in Quasar to find its reference ID, calls Workday's report to download the process definition, uses Claude to extract all the step details, and saves them to a text file. |
> | `wd_bp_quasar_xml_to_csv_pll` | **Workday path, Stage 2** — picks up the text file from Stage 1 (matched by the same chat ID), sends the extracted steps to Claude in parallel, and produces the final CSV of test cases. |
> | `workday_send_msg_to_canvas_wd` | **Chat reply** — sends the download link (or any message) back into the company chat window. |

### Helper 2 — "Expand the Ready-Made Library" (Finance version)
**Folder:** `GTS_TestCaseGeneration_BusinessProcess_Finance`

Exactly the same structure as Helper 1, but pointed at the **Finance** filing cabinet instead
of the HR one. Same four tools, same two paths, same result — just a different drawer of the
cabinet. The tool names all end in `_fin` so nobody mixes them up. The Finance library file is
`Vectorization_Repository___30th_Oct_2025.xlsx` (instead of the HR library's
`HCM_Test_Scenario_Library_Consolidated.xlsx`).

### Helper 3 — "Write Fresh Cases from a User Story"
**Folder:** `SA-GTS_UserStory_Businessprocess`

This folder has **three tools** offering **two separate ways in**:

| Tool | What it does |
|------|-------------|
| `wdquasarbp_to_testcases_csv` | **Library path** — same idea as Helper 1's library search: looks up pre-written scenarios in Quasar and asks Claude to generate test cases. This version processes chunks one at a time (no parallel speedup). |
| `wd_bp_quasar_to_workday_csv_us` | **User-story path** — you paste in a plain-English user story, and Claude invents brand-new test cases from scratch. No filing cabinet or Workday connection needed. Scenario codes start at `US0001`. This tool makes a **single Claude call** — it does not retry if the response is cut off. |
| `workday_send_msg_to_canvas` | **Chat reply** — sends the download link back into the chat. |

### Helper 4 — "Turn an Uploaded Spreadsheet into Test Cases"
**Folder:** `Workday Test Scenario Generator`

Here you **upload a spreadsheet** (XLSX format) that lists the steps of a process (exported
from Workday). The tool reads it, converts it to text, hands it to the AI, and gets back the
finished test cases. If Claude's response gets cut off, this tool **automatically asks it to
continue** until the whole answer is received.

This folder has **two tools**:

| Tool | What it does |
|------|-------------|
| `wd_test_scenerio_generation_to_csv` | Reads the uploaded XLSX, sends step data to Claude, produces a CSV of test cases. Supports automatic continuation if the response is cut off. |
| `workday_send_msg_to_canvas` | Sends the download link back into the chat. |

### Helper 5 — "The Heavy-Duty Uploader" (uses a different AI setup)
**Folder:** `SA - Workday TestScenario CWB`

Also takes an **uploaded spreadsheet**, but instead of asking the AI over the web, it actually
**installs and runs the Claude Code program on the server** (a command-line version of the AI
assistant) and drives it automatically to produce the test cases. It uses a different set of AI
login keys (Amazon's **Bedrock** service — a cloud that hosts AI models). Think of it as the
same goal reached with a bigger, more industrial engine under the hood.

This folder has **one tool**:

| Tool | What it does |
|------|-------------|
| `gp_file_download_CM_WD` | Downloads the uploaded XLSX, reads it, builds a detailed prompt, installs Claude Code CLI, runs it in a terminal session via Bedrock, reads the response from Claude Code's transcript file, and converts it to a CSV. |

> ⚠️ **Important:** This helper only works on **Linux servers**. It uses Linux-specific
> features (pseudo-terminals, file control signals) that are not available on Windows or macOS.
> Confirm with your platform team that the server runs Linux before deploying this helper.

---

## Section 5 — The Big Picture

> **Why read this?** One simple map so you can see how all the pieces fit together.

### System Overview — How All the Pieces Connect

<p align="center">
  <img src="docs/GTS Agent Workflow - From User Request to Test Case (1).png" width="700"/>
</p>

### Detailed Path Map — Every Helper Step-by-Step, with Examples

Each box below shows the exact pipeline for one helper. Follow the arrows from left to right
to see what happens at each stage. **Real-world examples** are included so you can see exactly
what you'd type and what comes back.

<p align="center">
  <img src="docs/Workday Test Case Generation Agent - E2E workflow (1).png" width="700"/>
</p>

### Example Walkthrough — The Most Common Flow

> **Scenario:** You want test cases for the *"Hire Employee"* business process and the HR
> scenario library is already loaded.

<p align="center">
  <img src="docs/Hire Employee Test Case Generation Workflow (1).png" width="700"/>
</p>

> **Total time:** About 2–5 minutes from typing to download link.

---

### Quick-reference: "Which Method do I use?"

| I have… | Use this helper | Folder |
|---------|----------------|--------|
| An HR process name and the scenario library is loaded | **Helper 1** — Library path | `GTS Business Process` |
| An HR process name but no pre-written scenarios exist | **Helper 1** — Workday-rulebook path | `GTS Business Process` |
| A Finance process name | **Helper 2** | `GTS_TestCaseGeneration_BusinessProcess_Finance` |
| A written user story | **Helper 3** — User-story path | `SA-GTS_UserStory_Businessprocess` |
| An HR process name (also has library search) | **Helper 3** — Library path | `SA-GTS_UserStory_Businessprocess` |
| A spreadsheet exported from Workday | **Helper 4** | `Workday Test Scenario Generator` |
| A spreadsheet and you need the heavy-duty engine | **Helper 5** (Linux servers only) | `SA - Workday TestScenario CWB` |

---

**What each piece in the diagram does:**

**🧑 You.** You type a request — a process name, a user story, or upload a file.

**🤖 The Platform.** It reads your request and routes it to the right tool. There is no single
"master brain" — each tool is registered independently on the platform, and the platform's
agent framework (or you, the user) decides which one fits.

**🗄️ Quasar.** The searchable filing cabinet. It does two jobs depending on the path:
- *Library path:* finds pre-written scenarios that match your process name, even if the wording
  isn't identical.
- *Workday-rulebook path:* looks up the exact reference ID for your process name, so the next
  step can call Workday's report.

**🏢 Workday.** The source of truth — the real, live definition of how each HR or finance
process actually runs. Only used in the Workday-rulebook path (Helper 1 & 2), and only
*after* Quasar has found the right reference ID.

**📄 Your Upload / Story.** The raw material when you'd rather bring your own step list or
description instead of searching.

**🧠 Claude API.** The writing brain for Helpers 1–4. It reads whatever raw material arrives
and turns it into clean, human-readable test cases. Called over the web using Accenture's AI
platform.

**🧠 Claude Code + Bedrock.** The writing brain for Helper 5 only. Instead of calling Claude
over the web, it installs and runs the Claude Code command-line program on the server, powered
by Amazon's Bedrock AI service. Same brain, different engine.

**📊 The Spreadsheet.** Holds all the finished test cases in one tidy table.

**💬 send_msg_to_canvas.** A small utility tool (every helper folder has one) that sends the
download link back into the company chat. It picks the right login based on which environment
you're pointing at (live, demo, test, dev, and so on).

---

## Section 6 — Before You Start (Your Checklist)

> **Why read this?** So you don't hit a wall halfway through. Tick these off first.

This is **not an app you install on your laptop.** It runs on an Accenture internal AI platform
(the code refers to web addresses containing `atr-gateway` and `acnopenai`).

- [ ] **A way into the company chat window** — the place you type your request. Ask your project
      lead for the address and a login.
- [ ] **The helpers must already be switched on** — an administrator must have published these
      tools on the platform. Confirm with your platform team that they're live.
- [ ] **For the HR library path:** the scenario library must already be loaded into the filing
      cabinet. Ask your platform team to confirm
      `HCM_Test_Scenario_Library_Consolidated.xlsx` is in place.
- [ ] **For the Finance library path:** ask your platform team to confirm
      `Vectorization_Repository___30th_Oct_2025.xlsx` is in place.
- [ ] **For the library or Workday path:** the exact process name. Small spelling differences can
      return nothing — use "Hire Employee", not "New Hire".
- [ ] **For the Workday-rulebook path:** remember it's a two-stage process. Run Stage 1 first,
      then Stage 2 with the **same conversation ID**.
- [ ] **For the user-story path:** a written user story with as much detail as you can give.
- [ ] **For the upload helpers:** your step-list spreadsheet (XLSX format) ready to upload.
- [ ] **For Helper 5 specifically:** confirm the server runs **Linux** and has **Node.js/npm**
      installed.

> **New here?** Ask your lead one question: *"Do I have a name to look up, a story to describe, or
> a file to upload?"* That single answer tells you which helper to use.

---

## Section 7 — How to Set It Up

> **Why read this?** These steps are for the **platform team** deploying the tools. Everyday users
> only need chat access and can skip this.

**Step 1 — Get a copy of the code.**
Download or copy this whole `GTS Repos` folder onto the server where the platform runs. Each
sub-folder is one helper (one independently registered agent with its own set of tools).

**Step 2 — Add the secret logins to the platform's safe.**
The platform has a locked digital safe (called a **secret vault**) for passwords and connection
details. Register the secrets listed in Section 8. If one is missing, the matching helper will
fail when it tries to connect.

**Step 3 — Install the small helper packages the code needs.**
Type this into the server's terminal (the black window where you type commands):

```bash
pip install requests pandas openpyxl
```

This tells the computer to download three helper packages: one for talking to the web
(`requests`), one for handling spreadsheet-style data (`pandas`), and one for reading Excel files
(`openpyxl`). You'll see text scroll by; at the end it should say "Successfully installed." If you
see a **red error**, jump to Section 9.

> **Note:** Helper 5 (`SA - Workday TestScenario CWB`) also needs:
> - **Node.js/npm** — a separate software toolkit — available on the server.
> - **pyte** — a Python terminal emulator package (`pip install pyte`).
> - A **Linux server** — this helper uses Linux-only system features and will not work on Windows.
> - The **Claude Code program** — installed automatically by the tool on first run, but npm must
>   be present for the install to succeed.

**Step 4 — Register each tool file on the platform.**
Every code file inside these folders must be registered as a "tool" under an agent, following
your organisation's deployment guide. Each sub-folder is a separate agent with multiple tools.

**Step 5 — Test with a simple request.**
Open the chat and type *"Generate test cases for Hire Employee."* If everything is wired up, you
should get a download link back within a few minutes. If you get an error, see Section 9.

---

## Section 8 — Settings You Need to Fill In

> **Why read this?** These are the "passwords and addresses" the tools need. Nothing works without them.

The tools do **not** read settings from a normal file. They read named **secrets** out of the
platform's locked safe. Below, each row is one secret you must register. **The actual values are
not in the code (which is correct and safe)** — get them from your platform team.

### Core Secrets (used by the main tools)

| What it is (plain English) | Secret name in the safe | What it holds | Used by |
|---|---|---|---|
| **The filing-cabinet login (Quasar)** | `wd-tool-secrets` | web address + username + password for Quasar | Helpers 1, 2, 3 (library & Workday paths) |
| **The AI's login (Claude over the web)** | `SCA_WA_Claude_Opus_4_6` | AI web address, secret key, model name | Helpers 1, 2, 3, 4 |
| **The Workday login** | `wd_dnt_10` | Workday address + username + password | Helper 1 & 2 (Workday-rulebook path only) |
| **The AI's login (Amazon Bedrock)** | `Bedrock Sonnet 4.5` | region, access key, secret key, model name | Helper 5 only |

### Chat-Reply Secrets (used by the `send_msg_to_canvas` tools)

Different helpers use slightly different secret names for different environments. Below is the
complete list across all helpers:

| Environment | Helper 1 & 2 use | Helpers 3, 4 use |
|------------|-----------------|-----------------|
| **Default (no env specified)** | `workday` | `workday_CUI` |
| **prod** (live) | `workday_prod` | `cs_atr_prod` |
| **demo** | `workday_CUI` | `workday_CUI` |
| **stagetest** | `workday_stgtest` | `cs-atr-stgtest` |
| **dev** | `convoui_cui_dev` | `cs_atr_dev` |
| **sapcui** | *(not available)* | `cui_sap_cred` |
| **platforms-dev** | *(not available)* | `atr_bg_cred` |
| **next-gen** | *(not available)* | `nextgencui` |

> **Values we could not find in the code:** every actual URL, username, and password. That's by
> design. *Ask your platform team or the internal Accenture wiki.*

---

## Section 9 — What Could Go Wrong

> **Why read this?** Real problems, described the way a person would actually say them, with fixes.

**Problem:** *"I typed a process name but it said no scenarios were found."*
**Why:** The filing cabinet found nothing matching your spelling, or that process has no
pre-written notes yet.
**Fix:** Double-check the exact Workday name (spelling and capitals matter). Try a shorter version
("Hire" instead of "Hire Employee — Full-time"). If it's genuinely not in the library, switch to
the user-story path or the Workday-rulebook path instead.
*(Technical detail: the summary shows `"No matching scenarios after filter"`.)*

**Problem:** *"I asked for the Workday-rulebook path but it couldn't find the process."*
**Why:** Quasar didn't return an entry named like *"Hire (default definition)"* when looking up the
reference ID.
**Fix:** Confirm the process exists in the BP definition file (`BP_Definition_HCM.xlsx` for HR)
and check the name spelling.
*(Technical detail: the summary shows `"Reference Not Found"`.)*

**Problem:** *"The second step said it couldn't find the file from the first step."*
**Why:** The Workday-rulebook path is two stages. Stage 2 looks for a text file left behind by
Stage 1, matched by the **same chat ID**. If you used a different ID (or skipped Stage 1), it
can't find it.
**Fix:** Run Stage 1 first, and use the **exact same conversation ID** for Stage 2.

**Problem:** *"The download link says Access Denied or Expired."*
**Why:** The link is temporary and stops working after a while.
**Fix:** Re-run the request for a fresh link and download it right away.

**Problem:** *"It's been thinking for ages and never sent a file."*
**Why:** Big jobs across many processes take a few minutes. Helper 3's library path works through
chunks one at a time (unlike Helpers 1 and 2 which process them in parallel), so it can be
slower.
**Fix:** Give it up to ~10 minutes. If nothing after 15, refresh the chat and try again with fewer
processes at once, or one at a time.

**Problem:** *"The AI login failed."*
**Why:** The AI's secret key is wrong, missing, or misconfigured.
**Fix:** Ask your platform team to check the relevant AI secret (`SCA_WA_Claude_Opus_4_6` for
Helpers 1–4, or `Bedrock Sonnet 4.5` for Helper 5) has the right key, address, and model name.

**Problem:** *"The test cases look cut off at the end."*
**Why:** The AI hit its length limit. Most helpers automatically ask it to continue (Helpers 1, 2,
the library path of Helper 3, and Helper 4 all do this). However, the **user-story path** (Helper
3) and **Helper 5** (CWB) make only a single call and do not retry.
**Fix:** Re-run for just that one process or a more focused input; the next attempt usually
finishes cleanly.

**Problem:** *"Helper 5 crashed with an error about `pty` or `fcntl`."*
**Why:** Helper 5 uses Linux-only features. It cannot run on Windows or macOS.
**Fix:** Deploy Helper 5 on a Linux server only. Confirm with your platform team.

---

## Section 10 — Why This Matters

> **Why read this?** For the manager deciding whether to keep funding this. Here's the value in plain numbers.

**Time saved.** Writing test cases for one Workday process by hand takes an experienced tester
**several hours**. This produces a full, structured draft in **under 10 minutes** — a saving of
roughly **90% or more** on the writing effort. A human still reviews it, but that review is
minutes, not hours.

**Fewer mistakes.** People get tired and skip cases. The robot systematically covers the yes/no
branches, the shortest and longest paths, completion, and approval outcomes — so scenarios are far
less likely to be missed.

**Consistency.** Ten people writing by hand produce ten different styles and depths. The AI applies
the same structure every time, which makes reviewing and comparing much faster.

**Flexibility.** Because the family offers a library lookup, a live-Workday read, a from-scratch
user story, and two upload paths, it fits teams at every stage — mature projects with a scenario
library and brand-new ones starting from nothing.

**What would happen without it?** Test teams would spend **days** writing documentation before they
could even begin testing. On a project with dozens of Workday processes, that's the difference
between a fast testing cycle and a slow one.

---

## Section 11 — Known Limitations and Watch-Outs

> **Why read this?** Being honest about the gaps, so nobody gets a nasty surprise in production.

We separate what we **know** from what we're **guessing**:
- ✅ **Confirmed** — seen directly in the code.
- 🔶 **Assumption** — a reasonable inference we can't fully prove from the files.

**✅ Confirmed — Some login details get printed into the server's log files.**
The `send_msg_to_canvas` tools in the `SA-GTS_UserStory_Businessprocess` and
`Workday Test Scenario Generator` folders print **username and password** into the server log (the
running diary of everything the app does) on line 35 of the `workday_send_msg_to_canvas` file in
each of those folders. Anyone who can read those logs could see them in plain text. The
`GTS Business Process` and Finance folder versions do **not** have this problem.
*Recommended fix for the developers: remove the `creds.get("password")` and
`creds.get("username")` from those print statements.*

**✅ Confirmed — Helper 5 runs the AI with safety prompts turned off.**
The heavy-duty uploader launches the command-line AI with a `--dangerously-skip-permissions` flag,
which lets it act without asking for confirmation. That's convenient on a controlled server, but
it should only run in a locked-down, trusted environment.

**✅ Confirmed — Helper 5 only works on Linux.**
This tool uses Linux-specific features (`pty`, `fcntl`, `termios`, `select`, `pyte`) that are not
available on Windows or macOS. Deploying on a non-Linux server will cause an immediate crash.

**✅ Confirmed — Two tools do not retry if the AI's answer is cut off.**
The **user-story path** in Helper 3 (`wd_bp_quasar_to_workday_csv_us`) and **Helper 5**
(`gp_file_download_CM_WD`) each make a single Claude call. If the response hits the token limit,
it won't be automatically continued. All other helpers (1, 2, the library path of 3, and 4)
**do** support automatic continuation.

**✅ Confirmed — An outdated data function is used in places.**
Some code uses an older spreadsheet function (`applymap`) that has been marked for removal in newer
versions of the `pandas` package. This occurs in `wdquasar_reposcenarios`,
`wdquasar_reposcenarios_fin`, and `wd_bp_quasar_xml_to_csv_pll`. On newer servers this may warn
and could eventually break.
*Recommended fix: switch `applymap` to the newer equivalent (`map`).*

**✅ Confirmed — There are no automated tests in this folder.**
Every change must be checked by hand by running it through the chat. That raises the risk of a
change quietly breaking something.

**✅ Confirmed — Some connections are hard-wired to specific accounts.**
The Workday-rulebook path points at a specific report owned by a specific person
(`madhavi.vangara/JN_Business_Process_Definition_Detail`). If that report moves, is renamed, or
that person's access changes, that path breaks.

**✅ Confirmed — Helper 3's library path processes chunks sequentially, not in parallel.**
Unlike Helpers 1 and 2 (which use `ThreadPoolExecutor` to process multiple chunks at once),
Helper 3's library path (`wdquasarbp_to_testcases_csv`) processes one chunk at a time. This means
it will be noticeably slower for large numbers of scenarios.

**🔶 Assumption — This runs on an Accenture internal AI platform.**
Based on the web addresses in the code (`atr-gateway`, `acnopenai`) it looks like an Accenture
GenAI/GenWizard platform, but the files don't state this outright.

> **Things we simply couldn't find in the files:** the exact download-link expiry time, the maximum
> number of processes allowed in one request, and how new scenarios get added to the filing
> cabinet. *Ask your platform team or the internal wiki.*

---

## Word List (Plain-English Dictionary)

> Every technical term used above, explained simply. Look here anytime you get stuck.

| Term | Plain-English meaning |
|---|---|
| **Business Process** | A named task inside Workday — e.g. "Hire Employee", "Change Job", "Terminate Employee". |
| **Test Case** | A step-by-step instruction sheet telling a tester what to do, what to type, and what the right result looks like. |
| **Scenario** | One specific situation being tested — e.g. "hire a part-time employee with no benefits." |
| **Workday** | Big cloud software companies use to run HR, payroll, and finance. This tool writes tests for it. |
| **HCM** | "Human Capital Management" — Workday's HR side (hiring, job changes, terminations). |
| **Quasar** | Accenture's internal searchable filing cabinet that finds things by meaning, not exact spelling. |
| **Claude** | The artificial-intelligence writing assistant (made by Anthropic) that actually writes the test cases. |
| **Claude Code** | A command-line version of that AI assistant that runs as a program on a server (used by Helper 5). |
| **AI / Artificial Intelligence** | Software that can read and write in human language, here used to draft the test cases. |
| **CSV / Spreadsheet** | A simple table file that opens in Excel or Google Sheets; each row is one record. |
| **XLSX** | An Excel spreadsheet file — the format you upload for Helpers 4 and 5. |
| **User Story** | A short plain-English wish written as *"As a [role], I want [action] so that [benefit]."* |
| **Security Role** | In Workday, what a person is allowed to do (e.g. "HR Partner" can start a hire). |
| **Secret / Secret Vault** | A locked digital safe on the platform holding passwords and keys, kept out of the code. |
| **Download Link** | A private web address for grabbing your file that expires after a set time. |
| **In Parallel** | Doing several things at once (like eight assistants working simultaneously) instead of one at a time. |
| **Log / Log File** | The running diary the software keeps of everything it does — useful for debugging, risky if it holds passwords. |
| **Bedrock** | Amazon's cloud service that hosts AI models; Helper 5 logs into the AI through it. |
| **Terminal** | The black command-line window where you type instructions to a computer. |
| **PTY / Pseudo-Terminal** | A simulated terminal window that lets one program (the tool) drive another (Claude Code) as if a human were typing. Linux only. |
| **Environment** | A version of a system for a purpose — "dev" (for building), "test", and "prod" (the live one real users see). |
| **Reference ID** | Workday's internal identifier for a business process definition — needed to fetch the rulebook. |
| **Two-Stage Pipeline** | The Workday-rulebook path is split into two steps: first fetch and extract the rules, then generate test cases from those rules. |
| **send_msg_to_canvas** | The small utility tool that sends results (like download links) back into the company chat window. |

---

*This README was written by reading the actual code inside every folder of `GTS Repos`. All
✅ Confirmed points were seen directly in the source. All 🔶 Assumptions are clearly labelled.
For deeper, developer-level detail, see the original README inside each individual sub-folder.*
