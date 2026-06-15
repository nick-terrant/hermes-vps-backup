---
name: github-repo-management
description: "GitHub operations: auth, repos, issues, PRs, code review, CI, releases."
version: 2.0.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [GitHub, Repositories, Git, Releases, Secrets, Configuration, Pull-Requests, Issues, Code-Review, CI/CD, Authentication]
    related_skills: []
---

# GitHub Operations — Complete Reference

All-in-one reference for GitHub operations: authentication, repository management, issues, pull requests, code review, CI, and releases. Each section shows `gh` first, then the `git` + `curl` fallback.

## Table of Contents

1. **Authentication** — Setup (HTTPS tokens, SSH keys, gh CLI)
2. **Repository Management** — Clone, create, fork, configure, releases
3. **Issues** — Create, triage, label, assign, search, bulk ops
4. **Pull Request Workflow** — Branch, commit, push, create PR, CI, merge
5. **Code Review** — Pre-commit gates, local review, PR review, inline comments
6. **Support Files** — Templates, scripts, and detailed references

---

## 1. Authentication

When a user asks you to work with GitHub, run this check first:

```bash
# Check what's available
git --version
gh --version 2>/dev/null || echo "gh not installed"

# Check if already authenticated
gh auth status 2>/dev/null || echo "gh not authenticated"
git config --global credential.helper 2>/dev/null || echo "no git credential helper"
```

**Decision tree:**
1. If `gh auth status` shows authenticated → use `gh` for everything
2. If `gh` is installed but not authenticated → use "gh auth" method below
3. If `gh` is not installed → use "git-only" method below

### HTTPS with Personal Access Token (git-only, no sudo)

Go to **https://github.com/settings/tokens** → Generate new token (classic) with scopes: `repo`, `workflow`, `read:org`.

```bash
# Cache credentials
git config --global credential.helper store

# Test — will prompt for username/token once
git ls-remote https://github.com/<username>/<any-repo>.git
```

### SSH Key Authentication

```bash
ssh-keygen -t ed25519 -C "email@example.com" -f ~/.ssh/id_ed25519 -N ""
cat ~/.ssh/id_ed25519.pub  # Add to https://github.com/settings/keys
ssh -T git@github.com       # Verify
git config --global url."git@github.com:".insteadOf "https://github.com/"
```

### gh CLI Authentication

```bash
# Interactive (desktop)
gh auth login  # Select GitHub.com → HTTPS → browser

# Headless / SSH server
echo "<TOKEN>" | gh auth login --with-token
gh auth setup-git
gh auth status
```

**Auth helper script:** `scripts/gh-env.sh` provides `$AUTH`, `$GH_USER`, `$GITHUB_TOKEN`, `$OWNER`, `$REPO` environment variables for all commands below.

---

## 2. Repository Management

> For environment setup (auth detection, token extraction), source `scripts/gh-env.sh` or run the inline auth detection block from Section 1.

### Cloning Repositories

Cloning is pure `git` — works identically either way:

```bash
# Clone via HTTPS (works with credential helper or token-embedded URL)
git clone https://github.com/owner/repo-name.git

# Clone into a specific directory
git clone https://github.com/owner/repo-name.git ./my-local-dir

# Shallow clone (faster for large repos)
git clone --depth 1 https://github.com/owner/repo-name.git

# Clone a specific branch
git clone --branch develop https://github.com/owner/repo-name.git

# Clone via SSH (if SSH is configured)
git clone git@github.com:owner/repo-name.git
```

**With gh (shorthand):**

```bash
gh repo clone owner/repo-name
gh repo clone owner/repo-name -- --depth 1
```

### Creating Repositories

**With gh:**

```bash
# Create a public repo and clone it
gh repo create my-new-project --public --clone

# Private, with description and license
gh repo create my-new-project --private --description "A useful tool" --license MIT --clone

# Under an organization
gh repo create my-org/my-new-project --public --clone

# From existing local directory
cd /path/to/existing/project
gh repo create my-project --source . --public --push
```

**With git + curl:**

```bash
# Create the remote repo via API
curl -s -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/user/repos \
  -d '{
    "name": "my-new-project",
    "description": "A useful tool",
    "private": false,
    "auto_init": true,
    "license_template": "mit"
  }'

# Clone it
git clone https://github.com/$GH_USER/my-new-project.git
cd my-new-project

# -- OR -- push an existing local directory to the new repo
cd /path/to/existing/project
git init
git add .
git commit -m "Initial commit"
git remote add origin https://github.com/$GH_USER/my-new-project.git
git push -u origin main
```

To create under an organization:

```bash
curl -s -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/orgs/my-org/repos \
  -d '{"name": "my-new-project", "private": false}'
```

### From a Template

**With gh:**

```bash
gh repo create my-new-app --template owner/template-repo --public --clone
```

**With curl:**

```bash
curl -s -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/owner/template-repo/generate \
  -d '{"owner": "'"$GH_USER"'", "name": "my-new-app", "private": false}'
```

### Forking Repositories

**With gh:**

```bash
gh repo fork owner/repo-name --clone
```

**With git + curl:**

```bash
# Create the fork via API
curl -s -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/owner/repo-name/forks

# Wait a moment for GitHub to create it, then clone
sleep 3
git clone https://github.com/$GH_USER/repo-name.git
cd repo-name

# Add the original repo as "upstream" remote
git remote add upstream https://github.com/owner/repo-name.git
```

### Keeping a Fork in Sync

```bash
# Pure git — works everywhere
git fetch upstream
git checkout main
git merge upstream/main
git push origin main
```

**With gh (shortcut):**

```bash
gh repo sync $GH_USER/repo-name
```

### Repository Information

**With gh:**

```bash
gh repo view owner/repo-name
gh repo list --limit 20
gh search repos "machine learning" --language python --sort stars
```

**With curl:**

```bash
# View repo details
curl -s \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO \
  | python3 -c "
import sys, json
r = json.load(sys.stdin)
print(f\"Name: {r['full_name']}\")
print(f\"Description: {r['description']}\")
print(f\"Stars: {r['stargazers_count']}  Forks: {r['forks_count']}\")
print(f\"Default branch: {r['default_branch']}\")
print(f\"Language: {r['language']}\")"

# List your repos
curl -s \
  -H "Authorization: token $GITHUB_TOKEN" \
  "https://api.github.com/user/repos?per_page=20&sort=updated" \
  | python3 -c "
import sys, json
for r in json.load(sys.stdin):
    vis = 'private' if r['private'] else 'public'
    print(f\"  {r['full_name']:40}  {vis:8}  {r.get('language', ''):10}  ★{r['stargazers_count']}\")"

# Search repos
curl -s \
  "https://api.github.com/search/repositories?q=machine+learning+language:python&sort=stars&per_page=10" \
  | python3 -c "
import sys, json
for r in json.load(sys.stdin)['items']:
    print(f\"  {r['full_name']:40}  ★{r['stargazers_count']:6}  {r['description'][:60] if r['description'] else ''}\")"
```

### Repository Settings

**With gh:**

```bash
gh repo edit --description "Updated description" --visibility public
gh repo edit --enable-wiki=false --enable-issues=true
gh repo edit --default-branch main
gh repo edit --add-topic "machine-learning,python"
gh repo edit --enable-auto-merge
```

**With curl:**

```bash
curl -s -X PATCH \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO \
  -d '{
    "description": "Updated description",
    "has_wiki": false,
    "has_issues": true,
    "allow_auto_merge": true
  }'

# Update topics
curl -s -X PUT \
  -H "Authorization: token $GITHUB_TOKEN" \
  -H "Accept: application/vnd.github.mercy-preview+json" \
  https://api.github.com/repos/$OWNER/$REPO/topics \
  -d '{"names": ["machine-learning", "python", "automation"]}'
```

### Branch Protection

```bash
# View current protection
curl -s \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO/branches/main/protection

# Set up branch protection
curl -s -X PUT \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO/branches/main/protection \
  -d '{
    "required_status_checks": {
      "strict": true,
      "contexts": ["ci/test", "ci/lint"]
    },
    "enforce_admins": false,
    "required_pull_request_reviews": {
      "required_approving_review_count": 1
    },
    "restrictions": null
  }'
```

### Secrets Management (GitHub Actions)

**With gh:**

```bash
gh secret set API_KEY --body "your-secret-value"
gh secret set SSH_KEY < ~/.ssh/id_rsa
gh secret list
gh secret delete API_KEY
```

**With curl:**

Secrets require encryption with the repo's public key — more involved via API:

```bash
# Get the repo's public key for encrypting secrets
curl -s \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO/actions/secrets/public-key

# Encrypt and set (requires Python with PyNaCl)
python3 -c "
from base64 import b64encode
from nacl import encoding, public
import json, sys

# Get the public key
key_id = '<key_id_from_above>'
public_key = '<base64_key_from_above>'

# Encrypt
sealed = public.SealedBox(
    public.PublicKey(public_key.encode('utf-8'), encoding.Base64Encoder)
).encrypt('your-secret-value'.encode('utf-8'))
print(json.dumps({
    'encrypted_value': b64encode(sealed).decode('utf-8'),
    'key_id': key_id
}))"

# Then PUT the encrypted secret
curl -s -X PUT \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO/actions/secrets/API_KEY \
  -d '<output from python script above>'

# List secrets (names only, values hidden)
curl -s \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO/actions/secrets \
  | python3 -c "
import sys, json
for s in json.load(sys.stdin)['secrets']:
    print(f\"  {s['name']:30}  updated: {s['updated_at']}\")"
```

Note: For secrets, `gh secret set` is dramatically simpler. If setting secrets is needed and `gh` isn't available, recommend installing it for just that operation.

### Releases

**With gh:**

```bash
gh release create v1.0.0 --title "v1.0.0" --generate-notes
gh release create v2.0.0-rc1 --draft --prerelease --generate-notes
gh release create v1.0.0 ./dist/binary --title "v1.0.0" --notes "Release notes"
gh release list
gh release download v1.0.0 --dir ./downloads
```

**With curl:**

```bash
# Create a release
curl -s -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO/releases \
  -d '{
    "tag_name": "v1.0.0",
    "name": "v1.0.0",
    "body": "## Changelog\n- Feature A\n- Bug fix B",
    "draft": false,
    "prerelease": false,
    "generate_release_notes": true
  }'

# List releases
curl -s \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO/releases \
  | python3 -c "
import sys, json
for r in json.load(sys.stdin):
    tag = r.get('tag_name', 'no tag')
    print(f\"  {tag:15}  {r['name']:30}  {'draft' if r['draft'] else 'published'}\")"

# Upload a release asset (binary file)
RELEASE_ID=<id_from_create_response>
curl -s -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  -H "Content-Type: application/octet-stream" \
  "https://uploads.github.com/repos/$OWNER/$REPO/releases/$RELEASE_ID/assets?name=binary-amd64" \
  --data-binary @./dist/binary-amd64
```

### GitHub Actions Workflows

**With gh:**

```bash
gh workflow list
gh run list --limit 10
gh run view <RUN_ID>
gh run view <RUN_ID> --log-failed
gh run rerun <RUN_ID>
gh run rerun <RUN_ID> --failed
gh workflow run ci.yml --ref main
gh workflow run deploy.yml -f environment=staging
```

**With curl:**

```bash
# List workflows
curl -s \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO/actions/workflows \
  | python3 -c "
import sys, json
for w in json.load(sys.stdin)['workflows']:
    print(f\"  {w['id']:10}  {w['name']:30}  {w['state']}\")"

# List recent runs
curl -s \
  -H "Authorization: token $GITHUB_TOKEN" \
  "https://api.github.com/repos/$OWNER/$REPO/actions/runs?per_page=10" \
  | python3 -c "
import sys, json
for r in json.load(sys.stdin)['workflow_runs']:
    print(f\"  Run {r['id']}  {r['name']:30}  {r['conclusion'] or r['status']}\")"

# Download failed run logs
RUN_ID=<run_id>
curl -s -L \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO/actions/runs/$RUN_ID/logs \
  -o /tmp/ci-logs.zip
cd /tmp && unzip -o ci-logs.zip -d ci-logs

# Re-run a failed workflow
curl -s -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO/actions/runs/$RUN_ID/rerun

# Re-run only failed jobs
curl -s -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO/actions/runs/$RUN_ID/rerun-failed-jobs

# Trigger a workflow manually (workflow_dispatch)
WORKFLOW_ID=<workflow_id_or_filename>
curl -s -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO/actions/workflows/$WORKFLOW_ID/dispatches \
  -d '{"ref": "main", "inputs": {"environment": "staging"}}'
```

### Gists

**With gh:**

```bash
gh gist create script.py --public --desc "Useful script"
gh gist list
```

**With curl:**

```bash
# Create a gist
curl -s -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/gists \
  -d '{
    "description": "Useful script",
    "public": true,
    "files": {
      "script.py": {"content": "print(\"hello\")"}
    }
  }'

# List your gists
curl -s \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/gists \
  | python3 -c "
import sys, json
for g in json.load(sys.stdin):
    files = ', '.join(g['files'].keys())
    print(f\"  {g['id']}  {g['description'] or '(no desc)':40}  {files}\")"
```

### Codebase Inspection (pygount)

Analyze repository composition — lines of code, language breakdown, file counts, code-vs-comment ratios.

```bash
pip install pygount
cd /path/to/repo
pygount --format=summary \
  --folders-to-skip=".git,node_modules,venv,.venv,__pycache__,.cache,dist,build,.next,.tox" \
  .
```

**Always exclude dependency/build dirs** — without `--folders-to-skip`, pygount crawls everything and may hang.

Filter by language:
```bash
pygount --suffix=py --format=summary .
pygount --suffix=py,yaml,yml --format=summary .
```

Pitfalls:
- Markdown shows 0 code lines (pygount classifies all content as comments) — expected
- JSON files show low counts — use `wc -l` directly for accurate JSON line counts
- Large monorepos: use `--suffix` to target specific languages rather than scanning everything

---

### Quick Reference Table

| Action | gh | git + curl |
|--------|-----|-----------|
| Clone | `gh repo clone o/r` | `git clone https://github.com/o/r.git` |
| Create repo | `gh repo create name --public` | `curl POST /user/repos` |
| Fork | `gh repo fork o/r --clone` | `curl POST /repos/o/r/forks` + `git clone` |
| Repo info | `gh repo view o/r` | `curl GET /repos/o/r` |
| Edit settings | `gh repo edit --...` | `curl PATCH /repos/o/r` |
| Create release | `gh release create v1.0` | `curl POST /repos/o/r/releases` |
| List workflows | `gh workflow list` | `curl GET /repos/o/r/actions/workflows` |
| Rerun CI | `gh run rerun ID` | `curl POST /repos/o/r/actions/runs/ID/rerun` |
| Set secret | `gh secret set KEY` | `curl PUT /repos/o/r/actions/secrets/KEY` (+ encryption) |

---

## 3. Issues Management

Create, search, triage, and manage GitHub issues. Source `scripts/gh-env.sh` for auth.

### Viewing Issues

```bash
# List open issues
gh issue list
gh issue list --state open --label "bug"
gh issue list --assignee @me
gh issue view 42
```

### Creating Issues

```bash
gh issue create \
  --title "Login redirect ignores ?next= parameter" \
  --body "## Description\n..." \
  --label "bug,backend" \
  --assignee "username"
```

Templates available in `templates/bug-report.md` and `templates/feature-request.md`.

### Managing Issues (Labels, Assignment, Comments)

```bash
gh issue edit 42 --add-label "priority:high,bug"
gh issue edit 42 --add-assignee username
gh issue comment 42 --body "Investigated — root cause is in auth middleware."
gh issue close 42
gh issue reopen 42
```

### Linking Issues to PRs

PR bodies with `Closes #42`, `Fixes #42`, or `Resolves #42` auto-close the issue on merge.

```bash
# Create branch from issue
gh issue develop 42 --checkout
```

### Bulk Operations

```bash
# Close all issues with a specific label
gh issue list --label "wontfix" --json number --jq '.[].number' | \
  xargs -I {} gh issue close {} --reason "not planned"
```

---

## 4. Pull Request Workflow

Full PR lifecycle: branch → commit → push → create PR → monitor CI → merge.

### Branch Creation

```bash
git fetch origin
git checkout main && git pull origin main
git checkout -b feat/add-user-authentication
```

Naming: `feat/`, `fix/`, `refactor/`, `docs/`, `ci/`. Use **Conventional Commits** — see `references/conventional-commits.md`.

### Creating a PR

```bash
git push -u origin HEAD
gh pr create \
  --title "feat: add JWT-based user authentication" \
  --body "$(cat templates/pr-body-feature.md)" \
  --label "enhancement" \
  --reviewer user1,user2
```

Options: `--draft`, `--base develop`. Templates in `templates/pr-body-bugfix.md` and `templates/pr-body-feature.md`.

### Monitoring CI

```bash
gh pr checks          # One-shot
gh pr checks --watch  # Poll until done
```

### Auto-Fixing CI Failures

1. Get failure details: `gh run list --branch $(git branch --show-current) --limit 5 && gh run view <RUN_ID> --log-failed`
2. Read failure logs → understand error
3. Fix with `patch`/`write_file`
4. `git add . && git commit -m "fix: ..." && git push`
5. Repeat up to 3 times, then ask user. See `references/pr-ci-troubleshooting.md` for common CI failures.

### Merging

```bash
# Squash merge + delete branch (recommended)
gh pr merge --squash --delete-branch

# Auto-merge (merges when checks pass)
gh pr merge --auto --squash --delete-branch
```

---

## 5. Code Review

Review local changes before pushing, or review open PRs on GitHub.

### Pre-Commit Quality Gates

```bash
# Security scan
git diff --staged | grep -in "password\|secret\|api_key\|token.*=\|private_key"

# Quality checks
ruff check .       # lint
mypy src/         # type check
pytest -x -q       # test
```

**Auto-fix flow:** Run linter with `--fix` → re-check gates → review auto-fix diff → fix remaining issues manually.

### Reviewing Local Changes (Pre-Push)

1. `git diff main...HEAD --stat` — see scope
2. `git diff main...HEAD` — read full diff
3. Use `read_file` on changed files for full context
4. Check for: secrets, debug statements, merge conflict markers, oversized files
5. Present findings in structured format: **Critical / Warnings / Suggestions / Looks Good**

### Reviewing a PR on GitHub

```bash
# View PR details and diff
gh pr view 123
gh pr diff 123 --name-only

# Check out locally for full review
gh pr checkout 123

# Leave inline review comments
HEAD_SHA=$(gh pr view 123 --json headRefOid --jq '.headRefOid')
gh api repos/$OWNER/$REPO/pulls/123/comments \
  --method POST \
  -f body="Use parameterized queries." \
  -f path="src/auth.py" \
  -f commit_id="$HEAD_SHA" \
  -f line=45 -f side="RIGHT"

# Submit formal review
gh pr review 123 --approve --body "LGTM!"
gh pr review 123 --request-changes --body "See inline comments."
```

### Review Checklist

Systematically check: **Correctness** (edge cases, error paths), **Security** (no secrets, no injection, auth checks), **Code Quality** (naming, DRY, SRP), **Testing** (new paths covered, error cases), **Performance** (no N+1, no blocking in async), **Documentation** (public APIs documented, non-obvious "why" comments).

Review output template: `references/review-output-template.md`.

---

## 6. Support Files

| File | Purpose |
|------|---------|
| `scripts/gh-env.sh` | Auth detection helper — sets `$AUTH`, `$GH_USER`, `$GITHUB_TOKEN`, `$OWNER`, `$REPO` |
| `references/github-api-cheatsheet.md` | GitHub REST API quick reference |
| `references/conventional-commits.md` | Conventional Commits format guide |
| `references/pr-ci-troubleshooting.md` | Common CI failure patterns and fixes |
| `references/review-output-template.md` | Structured code review output format |
| `templates/bug-report.md` | Issue template for bug reports |
| `templates/feature-request.md` | Issue template for feature requests |
| `templates/pr-body-bugfix.md` | PR body template for bugfix PRs |
| `templates/pr-body-feature.md` | PR body template for feature PRs |

