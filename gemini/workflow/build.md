# build

🚨 **CMD START: /build (v7)** - STATEFUL, HIGH-QUALITY WORKFLOW.

## PROTOCOL
**FAIL-SAFE:** Non-zero exit codes stop cursor. Use `|| exit 0`.
**PATHS:** Use `/` for all paths.

### 1. INIT
1. **Check Active:** Read `./.build_state/active_run.json`. If missing -> NEW.
2. **NEW:**
   a. **Dirs:** Write `./.build_state/.gitkeep`, `./docs/builds/.gitkeep`.
   b. **Vars:** Parse `[proj]`. Set `[TS]` via `date +%Y%m%d-%H%M%S` or Powershell.
   c. **Run ID:** Set `[ID]=[TS]-[proj]`. Set `[DOCS]="./docs/builds/[ID]"`. Write `[DOCS]/.gitkeep`.
   d. **State:** Set `[STATE]="[DOCS]/state.json"`. Write:
      ```json
      {"project":"[proj]","run_id":"[ID]","docs":"[DOCS]","phase":"PHASE_1_REQUIREMENTS","status":"pending","stack":"unknown","history":[{"phase":"START","ts":"[TS]"}]}
      ```
   e. **Pointer:** Write `./.build_state/active_run.json`: `{"active_run_path":"[DOCS]"}`.
   f. **OUT:**
      🔒 **AUTO BUILD (v7)**
      Init: "[proj]"
      Docs: **[DOCS]**
      To: **PHASE 1**
   g. **Load:** Read `[STATE]`. (Error? Re-init).
3. **EXISTING:**
   a. **Pointer:** Read `./.build_state/active_run.json` -> `[DOCS]`.
   b. **State:** Read `[DOCS]/state.json` -> `STATE`.
   c. **OUT:**
      🔒 **AUTO BUILD (v7)**
      Resuming: "[STATE.project]"
      Docs: **[DOCS]**
      Phase: **[STATE.phase]**
      Proceeding...

## PROTOCOLS (GLOBAL)
**INTERVIEW:** If ambiguous, ask ONE specific question with options (A, B, C). Wait.
**EFFICIENCY:** Be concise. No explanations unless asked. Silent Persona/Critic steps.
**CRITIC (QA):**
   1. **Review:** Deliverable vs Inputs.
   2. **Reject:** If flawed, OUT: "CRITIC: REJECTED. [Reason]...". Fix -> Retry (Max 3).
   3. **Pass:** Silent proceed.

## CONDUCTOR
Execute `STATE.phase`. Paths relative to `[DOCS]`. Update `[STATE]` on exit.

### IF PHASE_1_REQUIREMENTS
**Goal:** `requirements.md`. **Persona:** Lead BA (Silent).
1. Analyze prompt. Detect Ambiguity (Interview).
2. **Write** `requirements.md` (Goal, Stories, Criteria, reqs).
3. **CRITIC:** Check vs Prompt.
4. **Update State:** phase="PHASE_2_RESEARCH", status="pending_approval", deliverables.reqs="requirements.md".
5. **OUT:** "Drafted requirements.md. **Checkpoint 1/6**. Review/Reply."
6. **STOP.**

### IF PHASE_2_RESEARCH
**Goal:** `research_findings.md`. **Persona:** Researcher (Silent).
1. Read `requirements.md`. OUT: "Researching..."
2. **Web Search:** If avail, research stack/competitors.
3. **STACK:** Detect best stack (e.g., node, python). Set `[STACK]`.
4. **Write** `research_findings.md` (Summary, Stack: `[STACK]`, Patterns).
5. **CRITIC:** Check vs Reqs.
6. **Update State:** phase="PHASE_3_UX_PROTOTYPING", stack="[STACK]", deliverables.research="research_findings.md".
7. **OUT:** "Research done. Stack: **[STACK]**. **Checkpoint 2/6**. Review/Reply."
8. **STOP.**

### IF PHASE_3_UX_PROTOTYPING
**Goal:** `wireframes.md` (if UI). **Persona:** UX Designer (Silent).
1. **Check:** UI needed?
   - **NO:** OUT "No UI." Update State (PHASE_4). **Auto-Proceed.**
2. **YES:** OUT "Prototyping..."
   - Interview for ambiguity.
   - **Write** `wireframes.md` (Mermaid, text outlines).
   - **CRITIC:** Check vs Stories.
   - **Update State:** phase="PHASE_4_TECHNICAL_DESIGN", deliverables.proto="wireframes.md".
   - **OUT:** "UI Prototype: wireframes.md. Run dev server to verify. **Checkpoint 3/6**. Review/Reply."
   - **STOP.**

### IF PHASE_4_TECHNICAL_DESIGN
**Goal:** `technical_design.md`. **Persona:** Architect (Silent).
1. Read inputs. Interview ambiguity.
2. **Write** `technical_design.md` (Stack: `[STACK]`, Schema, File Struct, API, Flow Diagram).
3. **CRITIC:** Check vs Reqs/Stack.
4. **Update State:** phase="PHASE_5_TRACEABILITY_CHECK", deliverables.design="technical_design.md".
5. **OUT:** "Design saved. Checking traceability..."
6. **Auto-Proceed.**

### IF PHASE_5_TRACEABILITY_CHECK
**Goal:** Verify Matrix. **Persona:** QA Automation (Silent).
1. OUT: "Traceability check..."
2. **Write** `traceability_matrix.md` (Req vs File/Comp).
3. **CRITIC:** Check for gaps. (Reject = Fix Design + Matrix).
4. **Pass:** OUT "Traceability checked."
5. **Update State:** phase="PHASE_6_IMPLEMENTATION_PLAN", deliverables.trace="traceability_matrix.md".
6. **OUT:** "Traceability Report: traceability_matrix.md. **Checkpoint 4/6**. Review Design/Reply."
7. **STOP.**

### IF PHASE_6_IMPLEMENTATION_PLAN
**Goal:** `implementation_plan.md`. **Persona:** Tech Lead (Silent).
1. **Write** `implementation_plan.md` (Task list).
2. **CRITIC:** Check vs Design (Order/Missing files).
3. **Update State:** phase="PHASE_7_CODE_GENERATION", deliverables.plan="implementation_plan.md".
4. **OUT:** "Plan: implementation_plan.md. **Checkpoint 5/6**. Review/Reply."
5. **STOP.**

### IF PHASE_7_CODE_GENERATION
**Goal:** Code (TDD). **Persona:** AI Coder (Silent).
1. OUT "Generating code (TDD)..." Read Plan.
2. **TDD Loop:**
   - **Red:** Write failing test.
   - **Green:** Write code.
   - **Refactor.**
3. **Write full code.** No placeholders.
4. **CRITIC:** Check vs Plan.
5. **Update State:** phase="PHASE_8_TEST_AND_VALIDATION", deliverables.code="src".
6. **OUT:** "Code generated. Validating..."
7. **Auto-Proceed.**

### IF PHASE_8_TEST_AND_VALIDATION
**Goal:** Tests. **Persona:** Test Lead (Silent).
1. OUT "Validating [STACK]..."
2. **Cmds:**
   - Node: `npm install`, `npm test`
   - Py: `pip install -r requirements.txt`, `pytest`
   - Rust: `cargo build`, `cargo test`
   - Go: `go mod tidy`, `go test ./...`
3. **Run:** `[INSTALL] > install.log`, `[TEST] > test_output.log` (|| exit 0).
4. **Write** `test_report.md`.
5. **CRITIC:** Check vs Reqs. (Reject = Fix -> Re-run).
6. **Update State:** phase="PHASE_8_5_EXTERNAL_LINTING", deliverables.test_rpt="test_report.md".
7. **OUT:** "Tests passed. Report: test_report.md. Checking Lint..."
8. **Auto-Proceed.**

### IF PHASE_8_5_EXTERNAL_LINTING
**Goal:** Coderabbit. **Persona:** DevOps (Silent).
1. Check `coderabbit`.
   - **Missing:** OUT "Skipping Lint." Update State (PHASE_9). **Auto-Proceed.**
2. **Found:**
   - OUT "Running Coderabbit..."
   - Run: `coderabbit review --plain > coderabbit_report.txt`.
   - **Update State:** phase="PHASE_8_6_LINT_REVIEW", deliverables.lint="coderabbit_report.txt".
   - **OUT:** "Lint Report: coderabbit_report.txt. **Checkpoint 6/6**. Reply **'AUTOFIX'** or **'MANUAL'**."
   - **STOP.**

### IF PHASE_8_6_LINT_REVIEW
**Goal:** Fix/Proceed. **Persona:** AI Coder.
- **MANUAL:** Wait for "PROCEED".
- **AUTOFIX:** Loop Fix -> Verify.
- **Done:** Update State (PHASE_9). **Auto-Proceed.**

### IF PHASE_9_FINAL_REPORT
**Goal:** Finalize. **Persona:** PM (Silent).
1. OUT "Finalizing..."
2. **Write** `FINAL_REPORT.md` (Summary, Links).
3. **Write** `DEPLOYMENT.md` (Install/Run for [STACK]).
4. **Archive:** Delete `./.build_state/active_run.json`.
5. **CRITIC:** specific check for broken links/guides.
6. **Update State:** phase="COMPLETE", status="finished", deliverables.final="FINAL_REPORT.md".
7. **OUT:**
   Project: [STATE.project]
   Status: COMPLETE
   Docs: **[DOCS]**
   - `FINAL_REPORT.md`: Audit
   - `DEPLOYMENT.md`: Run Guide
   Workflow finished. Use /build to restart.
8. **STOP.**
