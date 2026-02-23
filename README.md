# Agentic Repo

A collection of reusable GitHub Actions workflows that use **GitHub Copilot CLI** to automate common repository maintenance tasks — issue triage, labeling, code review, release management, documentation sync, and more.

## Workflows

| Workflow | Trigger | Description |
|---|---|---|
| **Issue Quality Enhancer** | Issue opened | Normalizes issue titles and bodies: translates to English, adds structure, inserts relevant code references, and assigns labels |
| **Smart Labeler** | Issue / PR opened or edited | Analyzes content and applies the most appropriate existing labels |
| **Copilot Suggester** | Manual | Scans the codebase and opens GitHub Discussion ideas for security, performance, UX, and other improvements |
| **Label Beautifier** | Manual | Renames labels to follow a consistent emoji + color scheme; supports dry-run mode |
| **Continuous Documentation** | PR opened | Detects documentation drift between code changes and README / API docs; pushes fixes directly to the PR branch |
| **PR Quality Enhancer** | PR opened | Enhances PR titles and descriptions with structured context, code references, and summary of changes |
| **PR Code Reviewer** | PR opened / updated | AI-powered code review with inline comments on changed files and an overall summary |
| **Stale Manager** | Scheduled (weekly) / Manual | Warns and closes inactive issues and PRs after configurable inactivity periods; supports dry-run mode |
| **Release Notes Generator** | Release created / Manual | Generates structured release notes from commits and merged PRs between tags |
| **Discussion to Issue Converter** | Discussion labeled / Manual | Converts accepted Discussions into actionable issues with context preservation |
| **Duplicate Issue Detector** | Issue opened | Analyzes new issues against existing open issues and flags potential duplicates |
| **Dependency Update Advisor** | Scheduled (weekly) / Manual | Audits project dependencies for outdated packages and security vulnerabilities |
| **README Health Check** | Scheduled (monthly) / Manual | Checks README accuracy and completeness against the actual codebase; supports auto-fix mode |
| **Workflow Installer** | Manual | Deploys the caller workflows from this repo to one or more target repositories |
| **Lint Workflows** | Push / PR | Validates YAML syntax of all workflow files |

## How it works

Each workflow is implemented as a **reusable workflow** (defined here) and a corresponding **caller** (a lightweight file in `caller-workflows/`). Target repositories only need the caller; all logic stays in one place and updates are picked up automatically.

The caller files contain a `__OWNER__` placeholder in the `uses:` reference. When you install them via the **Workflow Installer**, the placeholder is automatically replaced with your GitHub username or organization. If you install callers manually, replace `__OWNER__` with the owner of your fork of this repository.

The workflows use **GitHub Copilot CLI** with the GitHub MCP Server to read repository context, interact with the GitHub API, and produce meaningful, project-aware output.

## Prerequisites

- A GitHub account with **GitHub Copilot** access
- A **Personal Access Token (PAT)** with Copilot permissions, stored as a repository secret named `COPILOT_PAT`

## Installation

### Option A — Caller workflows (recommended)

Copy the files from `caller-workflows/` into the target repository's `.github/workflows/` directory:

```bash
git clone https://github.com/<your-org>/agentic-repo.git
# Replace __OWNER__ with your GitHub org/user before copying
sed -i 's/__OWNER__/<your-org>/g' agentic-repo/caller-workflows/*.yml
cp agentic-repo/caller-workflows/*.yml <target-repo>/.github/workflows/
```

Then add the `COPILOT_PAT` secret to the target repository.

### Option B — Workflow Installer

1. Go to **Actions** → **Workflow Installer**
2. Enter the target repositories (comma-separated): `owner/repo1, owner/repo2`
3. Select which workflows to install
4. Choose whether to open a PR or push directly to the default branch
5. Run the workflow

The installer automatically replaces `__OWNER__` with the correct value.

### Option C — Full copy

Copy the workflows from `.github/workflows/` directly if you prefer to own the full implementation. In this mode each workflow runs standalone — no caller is needed.

## Configuration

### Author filter

By default, the **Issue Quality Enhancer**, **Smart Labeler**, **Continuous Documentation**, **PR Quality Enhancer**, **PR Code Reviewer**, and **Duplicate Issue Detector** workflows only run when the issue/PR author is the repository owner. You can change this behavior using the `author_filter` input:

| Value | Behavior |
|---|---|
| `owner_only` | (default) Only the repository owner triggers the workflow |
| `collaborators` | Any user with write/maintain/admin access can trigger it |
| `all` | Any author triggers the workflow — use with caution |

Example caller with a custom filter:

```yaml
jobs:
  enhance:
    uses: your-org/agentic-repo/.github/workflows/issue-quality-enhancer.yml@main
    with:
      issue_number: ${{ github.event.issue.number }}
      issue_title: ${{ github.event.issue.title }}
      issue_body: ${{ github.event.issue.body }}
      issue_author: ${{ github.event.issue.user.login }}
      author_filter: collaborators
    secrets: inherit
```

### UI analysis (Copilot Suggester)

The Suggester can build, run, and visually analyze your application via Playwright. This is enabled by default but can be disabled with `analyze_ui: false` if your project has no web UI or you want a faster code-only scan.

## Usage

### Issue Quality Enhancer

Triggers automatically when an issue is opened. Enhances title and body, adds labels, and leaves a summary comment.

### Smart Labeler

Triggers on issue and PR open/edit events. Compares existing labels against the repository label list and adjusts them to match the content. When both this workflow and the Issue Quality Enhancer run on the same issue, the Smart Labeler detects that the Enhancer already processed it and skips to avoid conflicts.

### Copilot Suggester

Run manually from the Actions tab. Select the suggestion category (`all`, `security`, `performance`, etc.) and whether to analyze the UI. Copilot scans the codebase, checks for existing issues and discussions, and opens Discussion entries for meaningful findings — skipping suggestions that are already tracked.

### Label Beautifier

Run manually. Set `dry_run: true` to preview proposed changes before applying them. Renames, recolors, and adds descriptions to all labels without breaking existing issue/PR associations.

### Continuous Documentation

Triggers automatically on pull requests that modify code files. Also available as a standalone workflow (no caller needed). Compares the diff against README sections and API docs, edits files directly in the PR branch to fix drift, and posts a summary comment.

### PR Quality Enhancer

Triggers automatically when a pull request is opened. Analyzes the code diff and branch context to enhance the PR title and body with a structured description, summary of changes, code references, and suggested reviewers.

### PR Code Reviewer

Triggers when a pull request is opened or updated. Reviews all changed files and posts inline comments on specific lines highlighting bugs, security concerns, performance issues, and style improvements. Also posts an overall review summary comment.

### Stale Manager

Runs weekly (Monday 9:00 UTC) on a schedule or manually. Scans all open issues and PRs, warns authors of items inactive for a configurable number of days (default 30), and closes items that remain inactive after the warning period (default 14 additional days). Dry-run mode is enabled by default for safe previewing.

### Release Notes Generator

Triggers when a new release is created or can be run manually with a specific tag. Analyzes all commits and merged PRs between the specified tag and the previous tag (auto-detected or manually specified), then updates the release body with categorized, structured notes.

### Discussion to Issue Converter

Triggers when a Discussion is labeled (default trigger label: `✅ accepted`) or can be run manually with a discussion number. Creates a new issue preserving the discussion context, links the issue back to the discussion, and adds a comment to the discussion pointing to the new issue.

### Duplicate Issue Detector

Triggers when a new issue is opened. Compares the issue title and body against all open issues using semantic similarity. If potential duplicates are found, posts a comment listing them with similarity scores and links. Does not automatically close issues — leaves the decision to maintainers.

### Dependency Update Advisor

Runs weekly (Monday 10:00 UTC) on a schedule or manually. Scans the repository for dependency manifests (package.json, requirements.txt, Gemfile, go.mod, etc.), checks for outdated packages and known security vulnerabilities, and opens a summary issue with prioritized update recommendations.

### README Health Check

Runs monthly (1st of each month at 10:00 UTC) on a schedule or manually. Compares README content against the actual codebase to detect outdated installation instructions, missing features, broken references, and inaccurate project structure sections. In `auto-fix` mode, it pushes corrections directly; in `report-only` mode (default), it opens an issue with findings.

### Workflow Installer

Run manually. Installs or converts workflows in one or more target repositories. If a full standalone workflow already exists, it is converted to a caller automatically. Accepts full GitHub URLs, `owner/repo`, or just `repo` (defaults to the current owner).

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
│       ├── pr-quality-enhancer.yml
│       ├── pr-code-reviewer.yml
│       ├── stale-manager.yml
│       ├── release-notes-generator.yml
│       ├── discussion-to-issue.yml
│       ├── duplicate-issue-detector.yml
│       ├── dependency-update-advisor.yml
│       ├── readme-health-check.yml
│       ├── workflow-installer.yml
│       └── lint.yml
├── caller-workflows/
│   ├── issue-quality-enhancer.yml
│   ├── smart-labeler.yml
│   ├── the-suggester-discussion-mode.yml
│   ├── label-beautifier.yml
│   ├── continuous-docs.yml
│   ├── pr-quality-enhancer.yml
│   ├── pr-code-reviewer.yml
│   ├── stale-manager.yml
│   ├── release-notes-generator.yml
│   ├── discussion-to-issue.yml
│   ├── duplicate-issue-detector.yml
│   ├── dependency-update-advisor.yml
│   └── readme-health-check.yml
├── README.md
└── README.es.md
```

## Required secret

| Secret | Scope | Description |
|---|---|---|
| `COPILOT_PAT` | Repository | GitHub PAT with Copilot access. Required in every repository that calls these workflows. |

## Security considerations

All Copilot CLI invocations use the `--allow-all-tools` flag, which grants the AI agent unrestricted access to every tool available in its runtime (GitHub MCP Server, Playwright, file system, etc.). Combined with the `COPILOT_PAT` secret, the agent can read and write repository content, issues, labels, discussions, and PR branches.

**Recommendations:**

- Use a PAT with the **minimum required scopes** (typically `repo`, `copilot`).
- Start with `author_filter: owner_only` (the default) and only expand after evaluating the risk.
- Review Copilot CLI output in the Actions logs after each run.
- For sensitive repositories, run the Suggester and Label Beautifier in `dry_run` mode first.

## Technologies

- [GitHub Actions](https://docs.github.com/en/actions) — workflow orchestration
- [GitHub Copilot CLI](https://docs.github.com/en/copilot/using-github-copilot/using-github-copilot-in-the-command-line) — AI-powered task execution
- [GitHub MCP Server](https://github.com/github/github-mcp-server) — structured GitHub API access for Copilot
- [Playwright MCP Server](https://github.com/microsoft/playwright-mcp) — browser automation for UI analysis (Copilot Suggester)

---

**[Leer en español](README.es.md)**
