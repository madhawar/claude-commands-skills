# Screenshot → Jira Bug Report

Turn one or more **annotated screenshots** (taken & marked up in Greenshot — arrows, text, labels, comment bubbles, highlights, shapes) into a structured Jira bug report ready to copy-paste, and save a markdown copy for the QA audit trail.

The annotations *are* the bug narrative — read them carefully. An arrow points at the offending element; a comment bubble or text label usually states what's wrong or what was expected; highlights/boxes mark the region of interest; numbered labels imply a step sequence.

## User Input

$ARGUMENTS

## How Screenshots Are Provided

The user attaches/pastes one or more images directly into the chat, **or** gives file paths in `$ARGUMENTS`. Free text in `$ARGUMENTS` may also include: a `JIRA_ID` / epic key, a one-line bug summary, environment hints (Parkly / TimePark, which portal), or an explicit priority/severity. None are required — infer what you can from the images.

If a file path is given, view it with the Read tool. If images are attached inline, read them directly.

## Step 1 — Read & Interpret Each Screenshot

For every screenshot, extract:

1. **The annotations** — transcribe every piece of text, label, comment bubble, and callout verbatim. These are the tester's own words and are the highest-signal source.
2. **What the arrows/highlights point at** — the specific UI element, value, column, error, or region flagged as wrong.
3. **Context clues** — URL bar, portal chrome, page title, breadcrumbs, Norwegian UI labels (see CLAUDE.md glossary), which portal (Admin / Owner / Operator / Platform / Consumer), and product (Parkly vs TimePark — check the domain).
4. **Visible state** — error messages, toasts, validation text, empty cells, mismatched data, broken layout, console/network panels if shown.
5. **DB/API proof** — if a screenshot shows a SQL result, API response, or terminal output, treat it as ground-truth evidence for Actual Result.

## Step 2 — Group Into Bugs (auto-detect)

Decide how many distinct bugs the screenshots represent:

- Screenshots that document **the same defect** (e.g. a setup shot + the failure + a DB proof, or the same issue across Parkly *and* TimePark) → **one** bug report. Note the cross-product occurrence inside that single report.
- Screenshots showing **unrelated defects** → **separate** bug reports.

**Always state your grouping decision up front** (e.g. *"3 screenshots → 1 bug"* or *"3 screenshots → 2 bugs: #1 = shots A+B, #2 = shot C"*) and invite the user to correct it before they paste into Jira.

## Step 3 — Produce the Bug Report

For **each** bug, output this exact format. Keep it tight and concrete — a dev should grasp the bug from this text alone, with the screenshot as confirmation.

```
**Summary:** <MODULE_NAME> | <what happens> when <action/trigger> — follow the pattern `<MODULE> | Y happens when we do Z to X`, e.g. "Operator Portal | Blank 'Sist oppdatert' shown for all rows when viewing Terminaler list">

**1. Pre-requisites** (optional)
<Account/role, portal, feature flags, or data state needed before reproducing. Omit the line entirely if none.>

**2. Steps to Reproduce**
1. <step>
2. <step>
3. <step>
<Derive from numbered annotations, the navigation path, and visible context. Be specific: exact portal URL, menu path, button labels.>

**3. Expected Result**
<What should happen — taken from the annotation's stated expectation, the Admin/source behaviour, or product logic.>

**4. Actual Result**
<What actually happens — what the arrows/comments flag. Quote the on-screen error/value. Cite DB/API evidence if shown.>

**5. Test Data** (optional)
<License plates, parking lot, account emails, IDs, SQL used, timestamps. Omit the line entirely if none.>

**6. Priority** (optional)
<P1 / P2 / P3 — only if confident. Otherwise omit.>

**7. Severity:** <Critical | High | Medium | Low> — <one short clause justifying it>

**Environment:** <Parkly | TimePark> • <Portal> • Staging
**Attached screenshots:** <filenames / brief descriptions, in order>
```

### Severity guide (this Jira's scale)

| Severity | Use when |
|---|---|
| **Critical** | Data loss/corruption, destructive action with no safeguard, payment/billing wrong, security exposure, or core flow fully blocked with no workaround. |
| **High** | Major feature broken or wrong data shown; workaround exists but is painful. Most migration-parity regressions (missing data/columns, broken CRUD) land here. |
| **Medium** | Partial malfunction, confusing/incorrect behaviour with an easy workaround, validation gaps. |
| **Low** | Cosmetic, copy/label, alignment, minor UI polish; no functional impact. |

Priority (optional) reflects business urgency (P1 = fix now → P3 = backlog) and is independent of severity — only fill it when the screenshots/context make it clear.

## Step 4 — Save the Markdown Copy (audit trail)

After presenting the report(s) in chat, save a markdown file per bug:

- **Folder:** `tests/features/{JIRA_ID}/bugs/` when a Jira ID / epic key is supplied; otherwise `bug-reports/`.
- **Filename:** `bug-{short-slug}-{YYYY-MM-DD-HHMM}.md` (slug from the summary; use today's date from context, ask for time if unknown rather than guessing).
- **Contents:** the full formatted report above, plus a transcription of each screenshot's annotations under a `## Screenshot Annotations` heading so the audit record stands alone even if images are lost.
- If a file with a near-identical summary already exists in the target folder, update it instead of creating a duplicate.

## Step 5 — Raise in Jira (when Atlassian MCP is available)

> **Currently:** copy-paste mode only — present the report and let the user paste it into Jira manually.

Once Atlassian MCP access is granted, this command should additionally offer to create the bug directly:

1. Confirm the **target epic / project key** with the user (from `$ARGUMENTS` or ask).
2. Map fields: Summary → issue summary; sections 1–5 → description (keep the numbered headings); Priority → priority field; Severity → the Severity custom field; Environment → environment/labels.
3. Attach the screenshot files to the created issue.
4. **Confirm before creating** — show the user the final payload and the epic it will be filed under, then create on approval (creating a Jira issue is an outward-facing action; never auto-create without an explicit go-ahead).
5. Report back the created issue key + URL.

## Tips

- **Title format:** Always use `<MODULE_NAME> | Y happens when we do Z to X`. The module is the area of the system (e.g. `Operator Portal`, `Platform Portal`, `Owner Portal`, `Consumer Portal`, `Admin Portal`, `Camera API`, `Payment Flow`). The rest describes the symptom and the trigger — not the cause.
- Lead with the tester's own annotation wording — they already diagnosed it; your job is to formalise, not reinterpret.
- If an annotation is ambiguous or a step is missing to reproduce, say so explicitly in the report (`<unclear from screenshot — confirm: ...>`) rather than inventing steps.
- Translate Norwegian UI labels to English in parentheses on first use, but keep the Norwegian for clickable labels in repro steps so the dev can follow the UI.
- When the same bug appears on both Parkly and TimePark, write one report and note both under Environment.
- Don't pad. If a section has nothing real to say, omit the optional ones and keep the required ones crisp.
