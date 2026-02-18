/n
# === Edit only if needed ===
UPSTREAM_OWNER="Mutigelink"
UPSTREAM_REPO="copilot-sdk"
REPO_NAME="${UPSTREAM_REPO}"
FORK_OWNER="stomde"
BRANCH="stoma-woken-up"
# ===========================

# If you haven't forked upstream to your account, fork + clone:
# gh repo fork "${UPSTREAM_OWNER}/${UPSTREAM_REPO}" --clone --remote=true

# If you've already forked (stomde/${REPO_NAME}), clone the fork instead:
gh repo clone "${FORK_OWNER}/${REPO_NAME}"

cd "${REPO_NAME}" || { echo "cd failed; check REPO_NAME"; exit 1; }

# Create and switch to branch
git switch -c "${BRANCH}"

# Create directories
mkdir -p .github/workflows .github/scripts

# Add CI workflow
cat > .github/workflows/ci.yml <<'YML'
name: CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  ci:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.x'

      - name: Set up Node
        uses: actions/setup-node@v4
        with:
          node-version: '18'

      - name: Set up Go
        uses: actions/setup-go@v4
        with:
          go-version: '1.20'

      - name: Make scripts executable
        run: chmod +x .github/scripts/test.sh || true

      - name: Run repository checks
        run: .github/scripts/test.sh
YML

# Add test script (auto-detects languages and runs checks)
cat > .github/scripts/test.sh <<'BASH'
#!/usr/bin/env bash
set -euo pipefail

echo "Detecting project type and running checks..."

# Go checks
if [[ -f go.mod ]]; then
  echo "==> Detected Go project (go.mod found). Running go checks..."
  gofmt -l .
  go vet ./...
  go test ./... -v -race -coverprofile=coverage.out
  echo "Go checks completed."
fi

# Python checks
if [[ -f requirements.txt ]] || [[ -f pyproject.toml ]] || [[ -f setup.py ]]; then
  echo "==> Detected Python project. Running Python checks..."
  python -m pip install --upgrade pip setuptools wheel || true
  if [[ -f requirements.txt ]]; then
    python -m pip install -r requirements.txt || true
  fi
  python -m pip install pytest black flake8 mypy || true
  pytest -q || { echo "pytest failed"; exit 1; }
  black --check . || { echo "black formatting issues"; exit 1; }
  flake8 . || { echo "flake8 issues"; exit 1; }
  mypy . || true
  echo "Python checks completed."
fi

# Node / TypeScript checks
if [[ -f package.json ]]; then
  echo "==> Detected Node project. Running Node checks..."
  if [[ -f package-lock.json ]]; then
    npm ci || npm install
  else
    npm install || true
  fi
  if npm run | grep -q "test"; then
    npm test || { echo "npm test failed"; exit 1; }
  fi
  if npm run | grep -q "lint"; then
    npm run lint || { echo "npm lint failed"; exit 1; }
  fi
  if [[ -f tsconfig.json ]]; then
    npx tsc --noEmit || true
  fi
  echo "Node checks completed."
fi

# Rust checks
if [[ -f Cargo.toml ]]; then
  echo "==> Detected Rust project. Running Rust checks..."
  cargo fmt -- --check || true
  cargo clippy --all-targets --all-features -- -D warnings || true
  cargo test --all || { echo "cargo test failed"; exit 1; }
  echo "Rust checks completed."
fi

echo "All detected checks completed."
BASH

# Make script executable
chmod +x .github/scripts/test.sh

# Add Makefile
cat > Makefile <<'MAKE'
.PHONY: all test lint ci

all: ci

test:
	sh .github/scripts/test.sh

lint:
	sh .github/scripts/test.sh

ci:
	sh .github/scripts/test.sh
MAKE

# Append README section (creates README.md if missing)
cat >> README.md <<'MD'

## Testing & CI

This repository includes a GitHub Actions workflow (.github/workflows/ci.yml) which auto-detects common project types (Go, Python, Node, Rust) and runs tests and linters.

Locally:
- Install your language dependencies (e.g. go mod download, pip install -r requirements.txt, npm ci).
- Run tests and linters with:
  make test
or
  sh .github/scripts/test.sh

Notes:
- If this repo is single-language (e.g., Go-only), you can ask me for a slimmer, optimized CI workflow (with caching and coverage upload).
MD

# Commit & push
git add .github/workflows/ci.yml .github/scripts/test.sh Makefile README.md
git commit -m "ci: add GitHub Actions workflow and test/lint scripts"
git push --set-upstream origin "${BRANCH}"

# Open PR to upstream main
gh pr create --base main --head "${FORK_OWNER}:${BRANCH}" --title "Add CI and test/lint workflow" --body "Adds a GitHub Actions workflow that auto-detects common languages (Go, Python, Node, Rust) and runs tests and common linters. Includes .github/scripts/test.sh and Makefile for local runs."/n# GitHub Copilot CLI (Public Preview)

The power of GitHub Copilot, now in your terminal.

GitHub Copilot CLI brings AI-powered coding assistance directly to your command line, enabling you to build, debug, and understand code through natural language conversations. Powered by the same agentic harness as GitHub's Copilot coding agent, it provides intelligent assistance while staying deeply integrated with your GitHub workflow.

See [our official documentation](https://docs.github.com/copilot/concepts/agents/about-copilot-cli) for more information.

![Image of the splash screen for the Copilot CLI](https://github.com/user-attachments/assets/f40aa23d-09dd-499e-9457-1d57d3368887)


## 🚀 Introduction and Overview

We're bringing the power of GitHub Copilot coding agent directly to your terminal. With GitHub Copilot CLI, you can work locally and synchronously with an AI agent that understands your code and GitHub context.

- **Terminal-native development:** Work with Copilot coding agent directly in your command line — no context switching required.
- **GitHub integration out of the box:** Access your repositories, issues, and pull requests using natural language, all authenticated with your existing GitHub account.
- **Agentic capabilities:** Build, edit, debug, and refactor code with an AI collaborator that can plan and execute complex tasks.
- **MCP-powered extensibility:** Take advantage of the fact that the coding agent ships with GitHub's MCP server by default and supports custom MCP servers to extend capabilities.
- **Full control:** Preview every action before execution — nothing happens without your explicit approval.

We're still early in our journey, but with your feedback, we're rapidly iterating to make the GitHub Copilot CLI the best possible companion in your terminal.

## 📦 Getting Started

### Supported Platforms

- **Linux**
- **macOS**
- **Windows**

### Prerequisites

- (On Windows) **PowerShell** v6 or higher
- An **active Copilot subscription**. See [Copilot plans](https://github.com/features/copilot/plans?ref_cta=Copilot+plans+signup&ref_loc=install-copilot-cli&ref_page=docs).

If you have access to GitHub Copilot via your organization or enterprise, you cannot use GitHub Copilot CLI if your organization owner or enterprise administrator has disabled it in the organization or enterprise settings. See [Managing policies and features for GitHub Copilot in your organization](http://docs.github.com/copilot/managing-copilot/managing-github-copilot-in-your-organization/managing-github-copilot-features-in-your-organization/managing-policies-for-copilot-in-your-organization) for more information.

### Installation

Install with [WinGet](https://github.com/microsoft/winget-cli) (Windows):

```bash
winget install GitHub.Copilot
```

```bash
winget install GitHub.Copilot.Prerelease
```

Install with [Homebrew](https://formulae.brew.sh/cask/copilot-cli) (macOS and Linux):

```bash
brew install copilot-cli
```

```bash
brew install copilot-cli@prerelease
```

Install with [npm](https://www.npmjs.com/package/@github/copilot) (macOS, Linux, and Windows):

```bash
npm install -g @github/copilot
```

```bash
npm install -g @github/copilot@prerelease
```

Install with the install script (macOS and Linux):

```bash
curl -fsSL https://gh.io/copilot-install | bash
```

Or

```bash
wget -qO- https://gh.io/copilot-install | bash
```

Use `| sudo bash` to run as root and install to `/usr/local/bin`.

Set `PREFIX` to install to `$PREFIX/bin/` directory. Defaults to `/usr/local`
when run as root or `$HOME/.local` when run as a non-root user.

Set `VERSION` to install a specific version. Defaults to the latest version.

For example, to install version `v0.0.369` to a custom directory:

```bash
curl -fsSL https://gh.io/copilot-install | VERSION="v0.0.369" PREFIX="$HOME/custom" bash
```

### Launching the CLI

```bash
copilot
```

On first launch, you'll be greeted with our adorable animated banner! If you'd like to see this banner again, launch `copilot` with the `--banner` flag.

If you're not currently logged in to GitHub, you'll be prompted to use the `/login` slash command. Enter this command and follow the on-screen instructions to authenticate.

#### Authenticate with a Personal Access Token (PAT)

You can also authenticate using a fine-grained PAT with the "Copilot Requests" permission enabled.

1. Visit https://github.com/settings/personal-access-tokens/new
2. Under "Permissions," click "add permissions" and select "Copilot Requests"
3. Generate your token
4. Add the token to your environment via the environment variable `GH_TOKEN` or `GITHUB_TOKEN` (in order of precedence)

### Using the CLI

Launch `copilot` in a folder that contains code you want to work with.

By default, `copilot` utilizes Claude Sonnet 4.5. Run the `/model` slash command to choose from other available models, including Claude Sonnet 4 and GPT-5.

### Experimental Mode

Experimental mode enables access to new features that are still in development. You can activate experimental mode by:

- Launching with the `--experimental` flag: `copilot --experimental`
- Using the `/experimental` slash command from within the CLI

Once activated, the setting is persisted in your config, so the `--experimental` flag is no longer needed on subsequent launches.

#### Experimental Features

- **Autopilot mode:** Autopilot is a new mode (press `Shift+Tab` to cycle through modes), which encourages the agent to continue working until a task is completed.

Each time you submit a prompt to GitHub Copilot CLI, your monthly quota of premium requests is reduced by one. For information about premium requests, see [About premium requests](https://docs.github.com/copilot/managing-copilot/monitoring-usage-and-entitlements/about-premium-requests).

For more information about how to use the GitHub Copilot CLI, see [our official documentation](https://docs.github.com/copilot/concepts/agents/about-copilot-cli).

## 🔧 Configuring LSP Servers

GitHub Copilot CLI supports Language Server Protocol (LSP) for enhanced code intelligence. This feature provides intelligent code features like go-to-definition, hover information, and diagnostics.

### Installing Language Servers

Copilot CLI does not bundle LSP servers. You need to install them separately. For example, to set up TypeScript support:

```bash
npm install -g typescript-language-server
```

For other languages, install the corresponding LSP server and configure it following the same pattern shown below.

### Configuring LSP Servers

LSP servers are configured through a dedicated LSP configuration file. You can configure LSP servers at the user level or repository level:

**User-level configuration** (applies to all projects):
Edit `~/.copilot/lsp-config.json`

**Repository-level configuration** (applies to specific project):
Create `.github/lsp.json` in your repository root

Example configuration:

```json
{
  "lspServers": {
    "typescript": {
      "command": "typescript-language-server",
      "args": ["--stdio"],
      "fileExtensions": {
        ".ts": "typescript",
        ".tsx": "typescript"
      }
    }
  }
}
```

### Viewing LSP Server Status

Check configured LSP servers using the `/lsp` command in an interactive session, or view your configuration files directly.

For more information, see the [changelog](./changelog.md).

## 📢 Feedback and Participation

We're excited to have you join us early in the Copilot CLI journey.

This is an early-stage preview, and we're building quickly. Expect frequent updates--please keep your client up to date for the latest features and fixes!

Your insights are invaluable! Open issue in this repo, join Discussions, and run `/feedback` from the CLI to submit a confidential feedback survey!
