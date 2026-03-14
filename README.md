# Manual Tester Agent

A GitHub Copilot + Playwright MCP powered manual QA tester agent.
Accepts Jira tickets as input and executes test cases step by step
with human-in-the-loop approval at every step.

## What it does

- Parses raw Jira tickets automatically (ticket ID, scenario, URL, test steps)
- Executes each test step using Playwright MCP browser_* tools
- Takes screenshots as evidence after every action
- Waits for human approval before moving to the next step
- Self-heals broken selectors, timeouts, URL changes, text mismatches
- Handles 8 blocker types — popups, CAPTCHA, iframes, session expiry and more
- Detects silent faults — console errors, toasts, network failures, A/B variants
- Full override control — fail, pass, note, pause, resume at any point
- Generates structured final report with heal log, override log, recommendations

## Setup

### Prerequisites
- Windows
- Node.js 18+
- VS Code 1.99+
- GitHub Copilot Chat extension
- Git

### Installation
```cmd