# GitHub Repository Setup

Run these commands once on your local machine to push this folder to GitHub.

---

## Option A — GitHub CLI (Recommended)

If you have the [GitHub CLI](https://cli.github.com) installed and authenticated:

```bash
# 1. Navigate to the repo folder
cd path/to/n8n-llm-pipeline

# 2. Initialize git
git init -b main

# 3. Stage everything
git add .

# 4. Initial commit
git commit -m "Initial commit: 4-LLM autonomous business pipeline"

# 5. Create the GitHub repo and push in one command
gh repo create n8n-llm-pipeline \
  --public \
  --description "4-LLM autonomous business pipeline: GPT-4o Mini, Gemini 1.5 Flash, Claude, Perplexity Sonar" \
  --push \
  --source .
```

---

## Option B — Manual (GitHub UI + git)

```bash
# 1. Go to github.com/new and create a repo named: n8n-llm-pipeline
#    - Do NOT initialize with README (you already have one)
#    - Click "Create repository"

# 2. In your terminal:
cd path/to/n8n-llm-pipeline

git init -b main
git add .
git commit -m "Initial commit: 4-LLM autonomous business pipeline"
git remote add origin https://github.com/YOUR_USERNAME/n8n-llm-pipeline.git
git push -u origin main
```

---

## Recommended GitHub Repo Settings

### Topics (Labels)
Add these topics to your repo for discoverability:
`n8n`, `llm`, `automation`, `openai`, `anthropic`, `gemini`, `perplexity`, `google-sheets`, `workflow`

### Branch Protection (Settings → Branches)
- Protect `main`
- Require pull request before merging
- Require status checks (the JSON validator CI must pass)

### Secrets (Not Needed)
API keys are stored in n8n, not in this repo. No GitHub Secrets needed for the CI workflow.

---

## After Push — GitHub Actions CI

The workflow in `.github/workflows/validate-json.yml` automatically runs on every push or PR to any `.json` file in `workflows/`. It:

1. Validates all workflow files are valid JSON
2. Checks required n8n fields (`name`, `nodes`, `connections`) are present
3. Warns about unfilled `REPLACE_WITH_` placeholders
4. Scans for accidentally committed API keys
5. Reports node/connection counts per workflow

You can see CI results under the **Actions** tab in your GitHub repo.
