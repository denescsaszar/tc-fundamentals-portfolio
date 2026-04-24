# 📄 git-fundamentals.md

## Git: Version Control From First Commit to Pull Requests

> **Goal:** Use Git confidently for daily development — committing, branching, collaborating, and shipping.

---

## 1. What is Git?

Git is a **distributed version control system**. It tracks changes to files over time, letting you revert mistakes, collaborate without overwriting each other's work, and maintain a full history of every decision your codebase has ever made.

> **Why this matters:** Git is non-negotiable in professional development. Every team uses it. Every job posting lists it. A GitHub portfolio is your public proof that you know how to work in a team.

> ⚠️ **Common mistake:** Treating Git like a backup tool — committing once a day with a vague message like `update`. Commits are a changelog. Write them for the developer who reads your history 6 months from now.

---

## 2. Core Concepts (Mental Model First)

Working Directory → Staging Area (Index) → Local Repo → Remote (GitHub)
(your files) (git add) (git commit) (git push)

text

- **Working Directory:** Files on your machine that you're editing
- **Staging Area:** A preview of what your next commit will contain
- **Local Repository:** Your complete history, stored in `.git/`
- **Remote:** GitHub — the shared copy your team works with

---

## 3. Essential Commands

### Setup (one-time)

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git config --global core.editor "code --wait"
```

### Starting a project

```bash
# Initialize a new repo
git init

# Clone an existing repo
git clone https://github.com/username/repo-name.git
```

### The daily workflow

```bash
# Check what's changed
git status

# See the actual diff
git diff

# Stage specific files
git add README.md

# Stage everything
git add .

# Stage interactively (choose chunks)
git add -p

# Commit
git commit -m "add: JSON parsing example for config files"

# Commit all tracked changes without separate add
git commit -am "fix: handle null values in parser"
```

### Syncing with GitHub

```bash
# Download changes (doesn't modify working files)
git fetch origin

# Download AND merge
git pull origin main

# Upload your commits
git push origin main

# Push a new branch and track it
git push -u origin feature/json-parser
```

> **Why this matters:** `fetch` vs `pull` — use `fetch` to see what changed remotely before deciding to merge. Use `pull` when you're ready to integrate.

> ⚠️ **Common mistake:** Pushing directly to `main` on a team project. Always work on a branch. `main` should only receive changes via pull requests.

---

## 4. Branching Strategy

### Create and switch branches

```bash
# Create and switch in one command
git checkout -b feature/add-xml-validator

# Modern equivalent (Git 2.23+)
git switch -c feature/add-xml-validator

# List all branches (* = current)
git branch

# List including remote
git branch -a
```

### Naming convention

main ← stable, deployable code
feature/... ← new functionality
fix/... ← bug fixes
chore/... ← maintenance
docs/... ← documentation updates

text

> **Why this matters:** Branch naming tells your team what's in progress without reading the code.

> ⚠️ **Common mistake:** Long-lived branches. A feature branch that drifts from `main` for weeks becomes a merge nightmare. Merge or rebase frequently.

---

## 5. Merge vs. Rebase

### Merge — Preserves history, adds a merge commit

```bash
git checkout main
git merge feature/add-xml-validator
```

History:
Merge branch 'feature/add-xml-validator'

|\

| \* add XSD validation for books.xml

| \* add books.xml test fixture

| fix: update README

|/

initial commit

text

### Rebase — Rewrites history, linear and clean

```bash
git checkout feature/add-xml-validator
git rebase main
```

History:
add XSD validation for books.xml

add books.xml test fixture

fix: update README

initial commit

text

|          | Merge                   | Rebase                       |
| -------- | ----------------------- | ---------------------------- |
| History  | Preserves branching     | Linear, cleaner              |
| Safety   | Safe on shared branches | Never rebase shared branches |
| Use when | Integrating into `main` | Cleaning up before a PR      |

> **Why this matters:** Recruiters and reviewers read Git history. A clean, linear history signals a developer who thinks before committing.

> ⚠️ **Common mistake:** Rebasing `main` or any branch other people are using. Rebase rewrites commit hashes — it breaks everyone else's local copies.

---

## 6. Pull Requests

```bash
# 1. Create a branch
git checkout -b feature/json-config-parser

# 2. Do your work, commit
git add config-parser.js
git commit -m "feat: add JSON config file parser with error handling"

# 3. Push to GitHub
git push -u origin feature/json-config-parser

# 4. Open a PR on GitHub UI or CLI
gh pr create --title "feat: JSON config parser" --body "Adds parser with schema validation"
```

### Good PR description template

```markdown
## What this PR does

Adds a JSON config file parser that validates against a schema.

## Why

Config parsing previously crashed on malformed JSON.

## How to test

1. Run `node parse.js` with `config.json`
2. Try with `broken-config.json` to see error handling

## Checklist

- [x] Tested manually
- [x] README updated
- [x] No console.log left in
```

> **Why this matters:** A well-written PR tells the reviewer _why_ you made the change, not just _what_ changed. That's what separates junior from senior contributors.

> ⚠️ **Common mistake:** Opening a PR with hundreds of lines changed and a one-word description. Small, focused PRs get reviewed and merged faster.

---

## 7. .gitignore

```gitignore
# Dependencies
node_modules/

# OS files
.DS_Store
Thumbs.db

# Logs
*.log

# Environment variables — never commit secrets
.env
.env.local

# Build output
dist/
build/

# Editor files
.cursor/
.vscode/settings.json
*.swp

# Test coverage
coverage/
```

> **Why this matters:** Committing `node_modules/` (200MB+) or `.env` files (API keys) pollutes your repo and exposes secrets.

> ⚠️ **Common mistake:** Creating `.gitignore` after already committing the files. Remove them first: `git rm -r --cached node_modules/`

---

## 8. Tagging

```bash
# Create an annotated tag (preferred)
git tag -a v1.0.0 -m "First stable release"

# List tags
git tag

# Push tags to GitHub
git push origin --tags
```

> **Why this matters:** Tags mark what's deployed. GitHub turns annotated tags into releases automatically.

---

## 9. GitHub Actions — Intro

Create `.github/workflows/validate.yml`:

```yaml
name: Validate JSON

on:
  push:
    branches: [main, "feature/**"]
  pull_request:

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repo
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "20"

      - name: Validate JSON files
        run: |
          for file in $(find . -name "*.json" -not -path "*/node_modules/*"); do
            echo "Validating $file"
            node -e "JSON.parse(require('fs').readFileSync('$file','utf8'))" \
              && echo "✓ Valid" || echo "✗ Invalid"
          done
```

> **Why this matters:** Automated checks catch mistakes before they reach `main`. This is the foundation of CI/CD.

> ⚠️ **Common mistake:** Hardcoding secrets in YAML. Use GitHub repository Secrets and reference them as `${{ secrets.MY_API_KEY }}`.

---

## Commit Message Convention

type: short description (50 chars max)

text

Types: `feat`, `fix`, `docs`, `chore`, `refactor`, `test`, `style`

```bash
# Good
git commit -m "feat: add XPath query support to XML parser"
git commit -m "fix: handle empty arrays in JSON stringify"
git commit -m "docs: add setup instructions to README"

# Bad
git commit -m "update"
git commit -m "stuff"
git commit -m "WIP"
```

---

## Self-Check

- [ ] Initialize a repo and make your first commit
- [ ] Create a branch, make changes, and merge back to `main`
- [ ] Explain the difference between merge and rebase
- [ ] Write a `.gitignore` for a Node.js project
- [ ] Open a pull request on GitHub with a clear description
- [ ] Explain what a GitHub Action does and when to use one
