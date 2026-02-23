# Agentic Repo

A collection of reusable GitHub Actions workflows that use **GitHub Copilot CLI** to automate common repository maintenance tasks — issue triage, labeling, documentation sync, and more.

## Workflows

| Workflow | Trigger | Description |
|---|---|---|
| **Issue Quality Enhancer** | Issue opened | Normalizes issue titles and bodies: translates to English, adds structure, inserts relevant code references, and assigns labels |
| **Smart Labeler** | Issue / PR opened or edited | Analyzes content and applies the most appropriate existing labels |
| **Copilot Suggester** | Manual | Scans the codebase and opens GitHub Discussion ideas for security, performance, UX, and other improvements |
| **Label Beautifier** | Manual | Renames labels to follow a consistent emoji + color scheme; supports dry-run mode |
| **Continuous Documentation** | PR opened | Detects documentation drift between code changes and README / API docs; pushes fixes directly to the PR branch |
| **Workflow Installer** | Manual | Deploys the caller workflows from this repo to one or more target repositories |

## How it works

Each workflow is implemented as a **reusable workflow** (defined here) and a corresponding **caller** (a ~15-line file in `caller-workflows/`). Target repositories only need the lightweight caller; all logic stays in one place and updates are picked up automatically.

The workflows use **GitHub Copilot CLI** with the GitHub MCP Server to read repository context, interact with the GitHub API, and produce meaningful, project-aware output.

## Prerequisites

- A GitHub account with **GitHub Copilot** access
- A **Personal Access Token (PAT)** with Copilot permissions, stored as a repository secret named `COPILOT_PAT`

## Installation

### Option A — Caller workflows (recommended)

Copy the files from `caller-workflows/` into the target repository's `.github/workflows/` directory:

```bash
git clone https://github.com/<your-org>/agentic-repo.git
cp agentic-repo/caller-workflows/*.yml <target-repo>/.github/workflows/
```

Then add the `COPILOT_PAT` secret to the target repository.

### Option B — Workflow Installer

1. Go to **Actions** → **Workflow Installer**
2. Enter the target repositories (comma-separated): `owner/repo1, owner/repo2`
3. Select which workflows to install
4. Choose whether to open a PR or push directly to the default branch
5. Run the workflow

### Option C — Full copy

Copy the workflows from `.github/workflows/` directly if you prefer to own the full implementation.

## Usage

### Issue Quality Enhancer

Triggers automatically when an issue is opened. Only runs for issues authored by the repository owner. Enhances title and body, adds labels, and leaves a summary comment.

### Smart Labeler

Triggers on issue and PR open/edit events. Compares existing labels against the repository label list and adjusts them to match the content.

### Copilot Suggester

Run manually from the Actions tab. Select the suggestion category (`all`, `security`, `performance`, etc.). Copilot scans the codebase, checks for existing issues, and opens Discussion entries for meaningful findings — skipping suggestions that are already tracked.

### Label Beautifier

Run manually. Set `dry_run: true` to preview proposed changes before applying them. Renames, recolors, and adds descriptions to all labels without breaking existing issue/PR associations.

### Continuous Documentation

Triggers automatically on pull requests that modify code files. Compares the diff against README sections and API docs, edits files directly in the PR branch to fix drift, and posts a summary comment.

### Workflow Installer

Run manually. Installs or converts workflows in one or more target repositories. If a full standalone workflow already exists, it is converted to a caller automatically.

## Project structure

```
agentic-repo/
├── .github/
│   └── workflows/
│       ├── issue-quality-enhancer.yml
│       ├── smart-labeler.yml
│       ├── the-suggester-discussion-mode.yml
│       ├── label-beautifier.yml
│       ├── continuous-docs.yml
│       └── workflow-installer.yml
├── caller-workflows/
│   ├── issue-quality-enhancer.yml
│   ├── smart-labeler.yml
│   ├── the-suggester-discussion-mode.yml
│   ├── label-beautifier.yml
│   └── continuous-docs.yml
├── README.md
└── README.es.md
```

## Required secret

| Secret | Scope | Description |
|---|---|---|
| `COPILOT_PAT` | Repository | GitHub PAT with Copilot access. Required in every repository that calls these workflows. |

## Technologies

- [GitHub Actions](https://docs.github.com/en/actions) — workflow orchestration
- [GitHub Copilot CLI](https://docs.github.com/en/copilot/using-github-copilot/using-github-copilot-in-the-command-line) — AI-powered task execution
- [GitHub MCP Server](https://github.com/github/github-mcp-server) — structured GitHub API access for Copilot
- [Playwright MCP Server](https://github.com/microsoft/playwright-mcp) — browser automation for UI analysis (Copilot Suggester)

---

**[Leer en español](README.es.md)**
