# Manual Tester Agent

A GitHub Copilot + Playwright MCP powered manual QA tester agent that accepts
Jira tickets as input and executes test cases step by step with human-in-the-loop
approval at every step.

---

## What it does

- Parses raw Jira tickets automatically — extracts ticket ID, scenario, URL, and test steps
- Executes each test step using Playwright MCP `browser_*` tools in Chromium
- Takes screenshots as evidence after every single action
- Waits for your approval before moving to the next step
- Self-heals broken selectors, timeouts, URL changes, and text mismatches
- Handles 8 blocker types — popups, CAPTCHA, iframes, session expiry, and more
- Detects silent faults — console errors, toasts, network failures, A/B variants
- Full override control — fail, pass, note, pause, and resume at any point
- Generates a structured final report with heal log, override log, and recommendations

---

## Requirements

| Tool | Version | Check command |
|------|---------|---------------|
| Windows | 10 or 11 | — |
| Node.js | 18 or higher | `node --version` |
| npm | 8 or higher | `npm --version` |
| VS Code | 1.99 or higher | Help → About |
| Git | Any recent | `git --version` |
| GitHub Copilot Chat | Latest | VS Code Extensions panel |

---

## Installation

### 1. Install Node.js

Download the LTS version from [nodejs.org](https://nodejs.org).
Run the installer and keep all defaults.
Make sure **"Add to PATH"** is checked during installation.

Verify in a new terminal window:

```cmd
node --version
npm --version
```

Both should print version numbers. If they do not, restart your PC and try again.

---

### 2. Install VS Code

Download from [code.visualstudio.com](https://code.visualstudio.com) and install.
Must be version **1.99 or higher** — MCP support is not available in older versions.

Verify your version: `Help → About`

---

### 3. Install GitHub Copilot Chat extension

1. Open VS Code
2. Press `Ctrl + Shift + X` to open Extensions
3. Search for **GitHub Copilot Chat**
4. Click Install
5. Sign in with your GitHub account when prompted
6. Confirm Copilot Chat is active — you should see the chat icon in the left sidebar

---

### 4. Clone this repository

Open a terminal (`Win + R` → type `cmd` → Enter) and run:

```cmd
git clone https://github.com/your-username/manual-tester-agent.git
cd manual-tester-agent
```

Then open the project in VS Code:

```cmd
code .
```

---

### 5. Install Playwright MCP

Inside the VS Code terminal (`Ctrl + ~`):

```cmd
npm install @playwright/mcp@latest
```

---

### 6. Install Chromium browser

```cmd
npx playwright install chromium
```

This downloads the Chromium browser that Playwright will control.
It takes about a minute — let it finish completely.

Verify it worked:

```cmd
npx @playwright/mcp --help
```

You should see a list of available flags printed in the terminal.

---

### 7. Allow the MCP server in VS Code

When you open the project, VS Code detects `.vscode/mcp.json` automatically.

1. A notification appears: **"MCP server found: playwright"**
2. Click **Allow**
3. To verify it started: press `Ctrl + Shift + P` and search **MCP: List Servers**
4. You should see `playwright` with a green dot (running)

If it shows red or missing — click it and select **Start**.

---

### 8. Switch Copilot to Agent mode

This is the most commonly missed step.
MCP tools only work in **Agent mode** — not in Ask or Edit mode.

1. Press `Ctrl + Alt + I` to open Copilot Chat
2. Find the mode dropdown at the bottom of the chat panel
3. Change it from **Ask** to **Agent**
4. Click the tools icon (plug icon) next to the input box
5. Confirm you see `playwright` and its `browser_*` tools listed

---

### 9. Verify everything works

Paste this into Copilot Agent chat:

```
Take a screenshot of https://example.com and tell me what you see
```

If Copilot opens a browser, navigates, and returns a screenshot — the entire
pipeline is working correctly and you are ready to run tests.

---

## Project structure

```
manual-tester-agent/
├── .github/
│   └── copilot-instructions.md   ← full tester agent brain (parser + HITL + healing)
├── .vscode/
│   └── mcp.json                  ← connects Playwright MCP to Copilot
├── screenshots/                  ← test evidence saved here (gitignored)
├── .gitignore
├── package.json
└── README.md
```

---

## Running a test

### Step 1 — Format your Jira ticket

Your ticket needs these fields:

```
[TICKET-ID]

Summary: [feature being tested]

Description:
[what the feature does and the URL to test]

Acceptance Criteria:
1. [action] — [expected result]
2. [action] — [expected result]
...
```

### Step 2 — Paste into Copilot Agent chat

Copy the entire ticket from Jira and paste it directly.
No reformatting needed.

### Step 3 — Confirm before starting

Copilot will parse the ticket and show a confirmation block:

```
📋 TICKET PARSED — QA REVIEW REQUIRED

Ticket:   QA-204 — [Summary]
URL:      [extracted URL]
Steps:    [N] total

[list of all steps]

Type: start / edit [N] / skip [N] / url [new url] / abort
```

Review the steps and type `start` to begin.

### Step 4 — Approve each step

After every step Copilot stops and shows:

```
✅ STEP 1 of 6 — PASS

Action:   [what was done]
Expected: [from ticket]
Actual:   [what was observed]
Evidence: [screenshot]

Type: next / fail [reason] / pass [reason] / note [text] / pause / abort
```

---

## Commands during a test run

| Command | What it does |
|---|---|
| `start` | Begin the test run from step 1 |
| `next` | Accept current result and move to next step |
| `fail [reason]` | Override PASS → FAIL with your reason |
| `pass [reason]` | Override FAIL → PASS with your reason |
| `note [text]` | Attach an observation to the step without changing result |
| `pause` | Pause the run at the current step |
| `resume [N]` | Resume from step N (e.g. `resume 4`) |
| `rerun [N]` | Re-execute step N from scratch |
| `skip [N]` | Skip step N and continue from N+1 |
| `abort` | Stop the run and generate a partial report now |

---

## Self-healing behaviour

When something goes wrong the agent attempts to heal before failing.
It always stops and asks you before continuing after a heal.

| Heal type | When it triggers |
|---|---|
| Element not found | Selector broke — tries fallback strategies |
| Page load timeout | Spinner or skeleton visible — waits and retries |
| URL mismatch | URL changed but page content looks correct |
| Text mismatch | Element found but text differs from expected |

---

## Blocker handling

| Blocker | Behaviour |
|---|---|
| Element disabled or covered | Reports — asks how to proceed |
| Popup or cookie banner | Identifies dismiss button — asks before closing |
| Scroll needed | Auto-scrolls silently — no approval needed |
| Session expired | Stops immediately — asks to re-login or abort |
| Multiple matching elements | Lists all matches — asks which one to use |
| Console JS errors | Logs as hidden fault — does not stop the run |
| iFrame content | Switches context automatically — logs it |
| CAPTCHA detected | Stops — asks you to solve it then resume |

---

## What the final report looks like

```
Ticket:    QA-204 — [Summary]
Tested by: GitHub Copilot + Playwright MCP
URL:       [URL]
Date:      [date]
Mode:      Step-by-step with human approval

Step results
| # | Action | Expected | Actual | Auto result | Override | Tester note |

Totals
  Passed (auto):            X
  Passed (tester override): X
  Failed (auto):            Y
  Failed (tester override): Y
  Blocked:                  Z
  Self-healed:              W
  Hidden faults:            V

Override log
Heal and blocker log
Hidden faults found
Root cause for failures
Recommendations
```

---

## Compatible test sites

| Site | URL | Good for |
|---|---|---|
| SauceDemo | https://www.saucedemo.com | Login, cart, checkout |
| DemoQA | https://demoqa.com | Forms, widgets, UI patterns |
| Automation Exercise | https://automationexercise.com | E-commerce full flow |
| The Internet | https://the-internet.herokuapp.com | Every UI pattern |
| Practice Test Automation | https://practicetestautomation.com/practice | Login page |

---

## Updating the instructions

To improve or extend the tester behaviour, edit:

```
.github/copilot-instructions.md
```

Then commit and push:

```cmd
git add .
git commit -m "update tester instructions"
git push
```

---

## Troubleshooting

**MCP server not appearing in Copilot tools**
- Check that `.vscode/mcp.json` has no syntax errors (validate at jsonlint.com)
- Press `Ctrl + Shift + P` → MCP: List Servers → click Start
- Make sure VS Code is 1.99 or higher

**Copilot not calling any browser tools**
- Confirm the mode dropdown shows **Agent** not Ask or Edit
- Click the tools icon and verify playwright tools are listed

**`npx @playwright/mcp --help` not found**
- Run `npm install @playwright/mcp@latest` again
- Restart the VS Code terminal after install

**Git push authentication error**
- Go to GitHub → Settings → Developer settings → Personal access tokens
- Generate a new token with `repo` scope
- Use the token as your password when Git prompts for credentials

---

## License

MIT