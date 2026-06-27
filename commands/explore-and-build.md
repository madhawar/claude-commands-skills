# Collaborative Explore & Build

Interactively explore a web application with the user, then generate a test instruction file and a Playwright MCP script based on what was discovered.

## User Input

$ARGUMENTS

## How to Parse Input

The user provides a free-text description of what they want to explore. Extract:

| Parameter | Required | Description | Example |
|---|---|---|---|
| `JIRA_ID` | **Yes** | Jira ticket ID for file naming | `ABC-12345` |
| `AREA` | **Yes** | What to explore — portal name, feature, or section | `Platform Portal terminals` |

If either is missing, ask the user before starting.

Resolve credentials, URLs, and environment details from `CLAUDE.md` and `.env` files in the project. Do NOT ask the user for these — they are already configured.

## Phase 1 — Setup & Login

1. Read `CLAUDE.md` to identify the correct portal URL and credentials for the area the user wants to explore.

2. Open a visible browser at 1920×1080 using the Playwright MCP:
   ```
   mcp: browser_navigate → portal URL
   mcp: browser_resize → 1920 × 1080
   ```
   The browser window will open on the user's screen.

3. Log in using the resolved credentials via `browser_fill` and `browser_click`.

4. Navigate to the specific section/area the user wants to explore.

5. Take an initial snapshot and screenshot of the landing page:
   ```
   mcp: browser_snapshot
   mcp: browser_screenshot → tests/features/{JIRA_ID}/screenshots/explore/00-landing.png
   ```

6. Tell the user: **"Browser is open at [section]. Tell me what to click, navigate to, or inspect. I'll snapshot and take notes as we go. Say 'done' when finished and I'll generate the test artifacts."**

## Phase 2 — Guided Exploration (Interactive Loop)

All browser interactions use the Playwright MCP exclusively — no terminal commands. Follow the user's directions. For each instruction:

1. **Execute** the action using the appropriate MCP tool:
   - Navigate: `browser_navigate`
   - Click: `browser_click` (use element ref from the most recent snapshot)
   - Fill input: `browser_fill`
   - Select dropdown: `browser_select_option`
   - Hover: `browser_hover`
   - Press key: `browser_press_key`
   - Go back: `browser_navigate` to the previous URL, or `browser_press_key → Alt+Left`
   - Reload: `browser_navigate` to the current URL again

2. **Snapshot** after every action — element refs change between interactions:
   ```
   mcp: browser_snapshot
   ```

3. **Screenshot** when entering a new section/view, or when the user asks:
   ```
   mcp: browser_screenshot → tests/features/{JIRA_ID}/screenshots/explore/{N}-{label}.png
   ```

4. **Note** the following in a running mental log:
   - Page/section name and URL
   - Key UI elements: buttons, fields, tables, modals, dropdowns
   - Element refs (e-numbers from snapshots) and their labels
   - Column names in any tables
   - Available actions (create, edit, delete, kebab menus)
   - Form fields with their types (text, dropdown, checkbox, required/optional)
   - Navigation paths (how to reach this section)
   - Any bugs, oddities, or differences the user points out
   - Norwegian labels and their English translations

Keep notes organized by **section/view**. Each time you enter a new area, start a new section in your notes.

### Responding During Exploration

After each action, briefly report:
- What the page now shows (from the snapshot)
- Key elements visible and their refs
- Anything notable (empty states, unexpected UI, errors)

Then wait for the user's next instruction.

### Special User Commands During Exploration

| User says | Action |
|---|---|
| `snapshot` / `snap` | `browser_snapshot` — report page structure and element refs |
| `screenshot` / `ss` | `browser_screenshot` → `tests/features/{JIRA_ID}/screenshots/explore/` |
| `note: <text>` | Add a custom note to the exploration log |
| `done` / `finish` | End exploration, move to Phase 3 |
| `back` | `browser_press_key → Alt+Left` or navigate to prior URL |
| `refresh` | `browser_navigate` to current URL |

## Phase 3 — Artifact Generation

When the user says "done", generate two artifacts:

### Artifact A: Instruction File

Save to `tests/features/{JIRA_ID}/{JIRA_ID}.instructions.md`:

```markdown
# {JIRA_ID} — {AREA} Functional Test Suite

## Objective
{What this test suite covers — derived from exploration}

## Environment
- Portal: {which portal, resolved from CLAUDE.md}
- URL: {base URL used}

## Sections Covered
{List of sections/views explored, with navigation paths}

## Test Cases

### TC-001: {Test case title}
**Section:** {Section name}
**Navigation:** {How to reach this section}
**Steps:**
1. {Step}
2. {Step}
**Expected Result:** {What should happen}

{Repeat for each test case derived from exploration}

## UI Element Reference
{Table of key elements, their labels, types, and notes discovered}

## Notes
{Any bugs, oddities, or observations from the exploration session}
```

Derive test cases from what was explored — CRUD operations, navigation, data validation, field behavior, etc. Each distinct action or verification becomes a test case.

### Artifact B: Playwright Script

Save to `tests/features/{JIRA_ID}/{area-kebab-case}.spec.ts`:

```typescript
import { test, expect } from '@playwright/test';

test.describe('{AREA} — Functional Tests', () => {

  test.beforeEach(async ({ page }) => {
    // Login flow using env vars from .env
    // Navigate to starting URL
  });

  test('TC-001: {test case title}', async ({ page }) => {
    // Steps discovered during exploration
    // Assertions based on expected behavior
  });

  // ... more test cases
});
```

Use the actual selectors, labels, and structure discovered during exploration — not guessed values. Reference credentials via `process.env` variables, never hardcode them.

### Presenting Artifacts

1. Show the user a summary of what was discovered (sections, test case count, key findings)
2. Present both artifacts for review
3. Ask if they want to adjust, add, or remove anything before saving
4. Save files to `tests/features/{JIRA_ID}/`

## Tips

- Always call `browser_snapshot` before any `browser_click` or `browser_fill` — refs are only valid from the most recent snapshot
- When exploring tables, note both the column headers AND sample data
- For modals/forms, note which fields have `*` (required) markers
- Note the exact Norwegian text on buttons and labels — tests should match these
- If the user mentions a comparison to another portal, note the differences
- Save screenshots to `tests/features/{JIRA_ID}/screenshots/explore/`
- The browser is visible on the user's screen — narrate what you're doing so they can follow along
