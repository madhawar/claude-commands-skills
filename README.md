# Testing Skills & Commands

This directory contains Claude Code `.skill` files and slash command `.md` files — reusable, self-contained testing capabilities and workflows that can be installed into Claude Code. Each `.skill` archive bundles a workflow definition (`SKILL.md`) and reference documents that Claude consults during execution. Commands in `commands/` are plain markdown files that define step-by-step workflows invocable as custom slash commands.

## Commands at a Glance

| File | Command | Purpose |
|---|---|---|
| `bug-report.md` | Screenshot → Jira Bug Report | Turn annotated screenshots into structured Jira bug reports with markdown audit copies |
| `explore-and-build.md` | Collaborative Explore & Build | Interactively explore a web app with the user, then generate test instruction files and Playwright scripts |

---

## bug-report.md

**Purpose:** Turn one or more annotated screenshots (Greenshot — arrows, text, comment bubbles, highlights) into a structured Jira bug report ready to copy-paste, and save a markdown copy for the QA audit trail.

**Trigger when:** the user pastes or attaches screenshots of a bug, asks to "write up a bug report", "create a Jira ticket from this screenshot", or wants to document a defect found during testing.

**Key behaviors:**

| Situation | Behavior |
|---|---|
| Multiple screenshots, same defect | Groups into one bug report; notes cross-product occurrence |
| Multiple screenshots, different defects | Produces separate reports; states grouping decision upfront |
| Jira ID / epic key provided | Saves markdown to `tests/features/{JIRA_ID}/bugs/` |
| No Jira ID | Saves to `bug-reports/` |
| Atlassian MCP available | Offers to create the Jira issue directly (with user approval) |

**Workflow at a glance:**
1. Read & interpret each screenshot — transcribe annotations, identify flagged UI elements, infer portal/product from context
2. Group into bugs (auto-detect related vs. unrelated screenshots)
3. Produce structured report: Summary, Pre-requisites, Steps to Reproduce, Expected Result, Actual Result, Test Data, Priority, Severity, Environment
4. Save markdown copy as audit trail
5. Optionally raise in Jira via Atlassian MCP (confirm before creating)

**Severity scale:** Critical → data loss / security / payment; High → major feature broken; Medium → partial malfunction with workaround; Low → cosmetic/copy.

---

## explore-and-build.md

**Purpose:** Collaboratively explore a live web application with the user via a visible Playwright browser, then generate a test instruction file and a Playwright spec script based on what was discovered.

**Trigger when:** the user wants to walk through a feature together before writing tests, says "let's explore X and build test cases", "open the browser and navigate with me", or provides a Jira ID and area to investigate interactively.

**Required inputs:** `JIRA_ID` (for file naming) and `AREA` (portal name, feature, or section to explore). Credentials and URLs are resolved from `CLAUDE.md` and `.env` — the user is not asked for these.

**Phases:**

| Phase | What happens |
|---|---|
| Phase 1 — Setup & Login | Opens a 1920×1080 browser, logs in, navigates to the target area, takes an initial screenshot |
| Phase 2 — Guided Exploration | Interactive loop: executes user directions (click, fill, navigate), snapshots after every action, screenshots on new views, builds a running log of UI elements, labels, and observations |
| Phase 3 — Artifact Generation | Produces an instruction file (`{JIRA_ID}.instructions.md`) with test cases derived from exploration, and a Playwright spec (`{area}.spec.ts`) using real selectors discovered during the session |

**Special exploration commands:** `snapshot`/`snap`, `screenshot`/`ss`, `note: <text>`, `done`/`finish`, `back`, `refresh`.

**Output artifacts:**
- `tests/features/{JIRA_ID}/{JIRA_ID}.instructions.md` — structured test suite with TC-IDs, navigation paths, steps, and expected results
- `tests/features/{JIRA_ID}/{area-kebab-case}.spec.ts` — Playwright TypeScript spec using actual selectors, credentials via `process.env`

---

## Skills at a Glance

| File | Skill | Purpose |
|---|---|---|
| `black-box-testing.skill` | Black-Box Testing | Functional test design and execution against live web apps, no source code required |
| `exploratory-testing.skill` | Exploratory Testing | Charter-guided unscripted discovery, including accessibility and UX lenses |
| `smoke-sanity-testing.skill` | Smoke & Sanity Testing | Fast go/no-go build gate — wide shallow smoke or narrow deep sanity |
| `test-cases-from-code.skill` | Test Cases From Code | Reverse-engineer test cases from source when no spec or author is available |
| `white-box-testing.skill` | White-Box Testing | Structural testing driven by coverage analysis with full source access |

---

## black-box-testing.skill

**Purpose:** Design and execute functional tests against a running web application without access to its source code. Produces maximum defect-finding coverage with the fewest redundant cases.

**Trigger when:** the user says "test this form/page/flow", "write test cases for", "find bugs in", or mentions black-box testing, test case design, equivalence partitioning, boundary value analysis, decision tables, state transition testing, pairwise/combinatorial testing, or negative testing.

**Key techniques (in `references/techniques.md`):**

| Situation | Technique |
|---|---|
| Inputs with ranges or limits | Boundary Value Analysis (BVA) |
| Inputs with distinct valid/invalid categories | Equivalence Partitioning (EP) |
| Multiple conditions combining into outcomes | Decision Table Testing |
| Workflows with states and transitions | State Transition Testing |
| Many independent configuration combinations | Pairwise (all-pairs) |
| End-to-end user journeys | Scenario / Use-Case Testing |
| Any feature | Error Guessing + Negative Testing |

**Workflow at a glance:**
1. Understand the target (URL, constraints, roles, test data)
2. Map the test surface (inputs, states, business rules, boundaries)
3. Select techniques matched to the feature shape
4. Design concrete test cases (ID, technique, precondition, steps, input data, expected result)
5. Present cases to the user for review, then execute via Playwright
6. Report: result overview → defects found → coverage notes

**Reference files:** `references/techniques.md`, `references/playwright-patterns.md`

---

## exploratory-testing.skill

**Purpose:** Simultaneous learning, test design, and execution — guided by a charter rather than a script. Finds defects that scripted suites never discover, including accessibility (WCAG) and usability/UX issues. Drives exploration through Playwright and feeds findings back into the scripted suite.

**Trigger when:** the user mentions exploratory testing, session-based testing, test charters, testing tours, heuristic evaluation, "find bugs without a script", "poke around this app", accessibility, a11y, WCAG, keyboard navigation, screen reader, color contrast, usability, UX testing, or "is this intuitive".

**Three lenses:**

| Charter focus | Lens |
|---|---|
| General behavior, edge cases, "what breaks" | Functional (charters-and-tours.md) |
| Keyboard/screen reader/contrast | Accessibility (accessibility.md) |
| Clarity, consistency, design compliance | Usability (usability.md) |

**Workflow at a glance:**
1. Set a charter: *Explore (target) using (resources/tours) to discover (information)*
2. Locate oracles: design spec / WCAG / consistency heuristics (HICCUPPS)
3. Choose one lens per pass
4. Explore — run tours, apply heuristics (SFDPOT, Goldilocks, CRUD), follow surprises
5. Capture during the session: charter, observations, bugs, questions, new charters
6. Debrief: coverage → bugs → observations/questions → new test-case candidates → new charters

**Reference files:** `references/usability.md`, `references/accessibility.md`, `references/charters-and-tours.md`

---

## smoke-sanity-testing.skill

**Purpose:** Fast confidence checks that produce a single go/no-go decision. Not an exhaustive bug hunt — a gate. Two modes: **smoke** (wide and shallow across critical paths after every new build) and **sanity** (narrow and deep on a changed area after a specific fix).

**Trigger when:** the user mentions smoke testing, sanity testing, build verification (BVT), "is this build stable", "is it safe to deploy", "quick check before we test", "did my fix work", "go/no-go", post-deploy check, or wants to gate a build before fuller testing.

**Mode selection:**

| Context | Mode |
|---|---|
| New build / deployment / "is it stable" | Smoke — wide & shallow, critical paths only |
| Specific fix or small change just landed | Sanity — narrow & deep on the changed area |

**Workflow at a glance:**
1. Determine mode (smoke or sanity)
2. Scope: smoke → identify critical paths; sanity → identify what changed and its blast radius
3. Execute fast via Playwright — one shallow happy-path check per function (smoke) or depth-first on the changed area (sanity); stop early on a hard blocker
4. Issue an explicit **GO** or **NO-GO** verdict
5. Report: verdict → mode & scope → checks table → blockers (if NO-GO) → recommendation

**Reference files:** `references/smoke.md`, `references/sanity.md`

---

## test-cases-from-code.skill

**Purpose:** Recover a usable test suite for a feature when the codebase is the only reliable source of truth — no current spec, no reachable author, no domain expert, and no existing automated tests. Reconciles pre-existing test cases (e.g. in Excel) against current code, hunts internal inconsistencies, and produces a risk register for anything that cannot be resolved from code alone.

**Trigger when:** the user says "create test cases from this branch", "our test cases are outdated", "fill the gaps in our test scenarios", "test this feature without domain knowledge", "no one knows how this works", or any situation where the codebase must serve as both the spec and the oracle.

**Core principle:** The codebase proves *behavior-as-written*, never *correctness*. Every output carries a provenance label so readers always know what was verified versus assumed.

**Provenance labels:**

| Label | Meaning |
|---|---|
| `CODE-VERIFIED` | Derived from code **and** confirmed by running the app |
| `CODE-DERIVED` | Derived from code, not yet executed |
| `EXISTING-OK` | Pre-existing test case; matches current code |
| `EXISTING-UPDATED` | Pre-existing case was stale; rewritten to match code |
| `DISCREPANCY` | Existing expectation conflicts with code; contested — do not pass/fail until adjudicated |

**Workflow at a glance (6 phases):**
1. **Triage** — identify scenario type (greenfield / gap-fill / update) and hunt for any intent authority (PR description, commit messages, linked tickets)
2. **Extract behavioral spec** — flat numbered list of "when X, system does Y" with `file:line` citations
3. **Reconcile** against existing Excel — matches, contradictions (→ risk register), gaps
4. **Consistency hunt** — scan for internal code contradictions (dead branches, overlapping validations, boundary mismatches) using the smell catalog
5. **Generate/update test cases** — fill coverage gaps; weight toward boundaries and error paths
6. **Execute & risk-register handoff** — run against the app, promote `CODE-DERIVED → CODE-VERIFIED`, route unresolved items to the release owner

**Reference files:** `references/analysis-passes.md`, `references/consistency-smells.md`

---

## white-box-testing.skill

**Purpose:** Test code from the inside, with full source access. Uses coverage analysis to find what existing tests don't exercise, then applies structural techniques to derive inputs that reach those paths. Goal: cover every reachable behavior, not just every line.

**Trigger when:** the user mentions white-box testing, structural testing, glass-box testing, code coverage, statement/branch/condition coverage, MC/DC, basis path testing, cyclomatic complexity, data flow testing, def-use chains, loop testing, mutation testing, or says "improve test coverage", "write unit tests for", "find untested branches", or "assess my test suite".

**Technique ladder:**

| Need | Technique |
|---|---|
| Every line runs at least once | Statement coverage |
| Every decision tested true and false | Branch / decision coverage *(default for most code)* |
| Each boolean sub-condition independently exercised | Condition coverage / MC/DC |
| Independent paths through complex logic | Basis path testing (cyclomatic complexity) |
| Variables correct between definition and use | Data flow testing (def-use chains) |
| Loops correct at 0 / 1 / many / boundary iterations | Loop testing |
| Assess whether the tests themselves are adequate | Mutation testing |

**Workflow at a glance:**
1. Understand scope, stack, existing coverage baseline, and criticality
2. Confirm runner and coverage tooling; get a baseline number (`references/coverage-tooling.md`)
3. Select technique(s) scaled to complexity and criticality
4. Design tests: name after what they cover, arrange minimal state, act, assert on outcomes
5. Execute with coverage; confirm new tests raised coverage and all pass
6. Report: coverage before → after → tests added → findings (dead code, impossible conditions, suspected defects) → remaining gaps

**Reference files:** `references/coverage-tooling.md`, `references/techniques.md`
