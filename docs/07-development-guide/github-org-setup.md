## GitHub Organization & Repository Setup — Free Tier Edition

> **Plan Version**: 1.0 for GitHub Free Tier  
> **Date**: May 29, 2026  
> **Billing**: FREE (no cost) — Upgrade to Team ($21/user/mo) later with zero re-setup  

**TL;DR**: Set up GitHub Free organization with 5 service repositories, branch protection rules, team permissions, CI/CD pipelines (2,000 free minutes/mo), and secrets. Everything works on Free tier — no missing features for startup development.

**What's Available on Free:**
- ✅ Unlimited private repos
- ✅ Branch protection rules (all features)
- ✅ Teams & permissions (org roles)
- ✅ CODEOWNERS file (manual review requests)
- ✅ GitHub Actions (2,000 free minutes/month)
- ✅ PR/Issue templates
- ✅ Organization secrets
- ✅ Discussions

**What Requires Paid Tier Later:**
- ❌ Advanced security scanning (GitHub Advanced Security)
- ❌ Code owners auto-enforcement (Team+ plan)
- ❌ Unlimited Actions minutes (scale beyond 2,000/mo)

---

## **Phase 1: GitHub Organization Setup (1 day)**

### 1. Create GitHub Organization — FREE
**In GitHub UI:**
- Go to github.com → top-right menu → **"New organization"**
- Choose **"Free"** plan (not Team or Enterprise)
- Organization name: `petzonic`
- Organization account owner: your email
- This organization belongs to: Personal (select this)
- Click **"Create organization"**
- Invite co-founders/CTO:
  - Settings → Members → Invite member
  - Assign role: **"Owner"** (they can manage everything)
  - Send invite

### 2. Create 5 Service Repositories — FREE
**Repos to create:**
- `petzonic-api` (Backend, NestJS)
- `petzonic-web` (Web + Admin, Next.js)
- `petzonic-customer-app` (Mobile, Flutter)
- `petzonic-seller-app` (Mobile, Flutter)
- `petzonic-infra` (Infrastructure, Docker, Terraform)

**For each repo in GitHub UI:**
1. Organization → New → Repository
2. Repository name: (e.g., `petzonic-api`)
3. Description: Brief description (link to docs: https://github.com/petzonic/petzonic-docs)
4. Private: ✅ (select this — private repo)
5. Initialize: ☐ DO NOT check "Add README"
6. Click **"Create repository"**
7. After creation:
   - Go to Settings → General
   - **Default branch**: Change from `main` to `develop`
     - Click branch selector → `develop` → Update
   - **Features** → Enable **Discussions** ✅
   - Save

---

## **Phase 2: Branch Protection & Workflows (2-4 hours per repo)**

### 3. Set Branch Protection Rules — FREE (All Features Work)
**For each repo, set rules for `main` branch:**

In GitHub UI:
1. Settings → Branches → Add rule
2. Branch name pattern: `main`
3. ✅ **Require a pull request before merging**
   - Require approvals: 1
   - Dismiss stale pull request approvals when new commits are pushed: ✅
4. ✅ **Require status checks to pass before merging**
   - Status checks: `lint`, `test`, `build` (add these once workflows created)
5. ✅ **Require branches to be up to date before merging**
6. ✅ **Restrict who can push to matching branches** → Admins only
7. ✅ **Allow auto-merge** → Squash merging
8. ✅ **Require signed commits** (optional)
9. Click **"Create"**

**For each repo, set rules for `develop` branch:**
1. Add another rule, pattern: `develop`
2. ✅ Require PR reviews (min 1)
3. ✅ Require status checks (same as main)
4. ✅ Dismiss stale approvals
5. ✅ Require branches up to date
6. Do NOT restrict push (devs need to push to develop)
7. Click **"Create"**

### 4. Create CODEOWNERS File — FREE
**In each repo:**
1. Code → Create new file → `.github/CODEOWNERS`
2. Paste:
```
* @tech-lead @cto
/src/auth/ @backend-team
/src/payments/ @backend-team
/pages/ @frontend-team
/lib/ @flutter-team
```
3. Commit: "Add CODEOWNERS for automatic review requests"

**Note (Free Tier)**: CODEOWNERS will NOT automatically block merges on Free tier. You must manually request reviews. On Team+ plan, they auto-request and can auto-block. Workaround: tech-lead must manually request reviews when PRs are created.

### 5. Create PR & Issue Templates — FREE
**In each repo:**

**Create `.github/pull_request_template.md`:**
1. Code → Create new file → `.github/pull_request_template.md`
2. Paste:
```markdown
## Description
What does this PR do?

## Testing
- [ ] Unit tests passed
- [ ] Integration tests passed
- [ ] Tested locally

## Checklist
- [ ] Code follows style guide
- [ ] No console.logs or debug code
- [ ] Documentation updated
- [ ] Database migrations (if any)
```
3. Commit

**Create `.github/ISSUE_TEMPLATE/bug_report.md`:**
1. Code → Create new file → `.github/ISSUE_TEMPLATE/bug_report.md`
2. Paste:
```markdown
**Describe the bug:**

**Steps to reproduce:**
1. 
2. 

**Expected vs actual:**

**Environment:**
- OS:
- Browser/App version:
```
3. Commit

**Create `.github/ISSUE_TEMPLATE/feature_request.md`:**
1. Code → Create new file → `.github/ISSUE_TEMPLATE/feature_request.md`
2. Paste:
```markdown
**Describe the feature:**

**User story:**
As a [role], I want [goal] so that [benefit]

**Acceptance criteria:**
- [ ] 
- [ ] 
```
3. Commit

---

## **Phase 3: Team & Permissions (1-2 hours)**

### 6. Create GitHub Teams — FREE
**In GitHub UI:**

Organization → **Teams** → **New team**

Create these teams (repeat for each):
1. **backend-team** → Description: "Backend developers (NestJS)" → Create
2. **frontend-team** → Description: "Frontend developers (Next.js)" → Create
3. **flutter-team** → Description: "Mobile developers (Flutter)" → Create
4. **devops-team** → Description: "DevOps & Infrastructure" → Create
5. **leads** → Description: "Tech lead & CTO" → Create
6. **qa** → Description: "QA engineers" → Create

**Add members to each team:**
1. Organization → Teams → Select team (e.g., backend-team)
2. Members → Add member
3. Search & add team members
4. Repeat for all teams

### 7. Set Repository Access — FREE
**For each repo, set team permissions:**

Repository → Settings → Collaborators and teams

| Team | Access Level | Repos | Instructions |
|------|---|---|---|
| `leads` | **Admin** | All 5 repos | Add to each repo |
| `backend-team` | **Maintain** | petzonic-api, petzonic-infra | Can push/merge PRs |
| `frontend-team` | **Maintain** | petzonic-web | Can push/merge PRs |
| `flutter-team` | **Maintain** | petzonic-customer-app, petzonic-seller-app | Can push/merge PRs |
| `devops-team` | **Admin** | petzonic-infra | Full control |
| `qa` | **Triage** | All 5 repos | Can open issues, label (no merge) |

**For each repo:**
1. Settings → Collaborators and teams → Add teams
2. Select team (e.g., backend-team)
3. Choose role: **Maintain** (or Admin/Triage as per table)
4. Confirm

---

## **Phase 4: Secrets & CI/CD (2-3 hours)**

### 8. Add Organization Secrets — FREE
**In GitHub UI:**

Organization → Settings → **Secrets and variables** → **Actions**

Add each secret (used by all repos' CI/CD workflows):

**For each secret:**
1. Click **New organization secret**
2. Name: (e.g., `DATABASE_URL`)
3. Value: (e.g., PostgreSQL staging connection string)
4. Repository access: **All repositories**
5. Click **Add secret**

**Secrets to add:**
```
DATABASE_URL          (PostgreSQL staging)
DATABASE_URL_PROD     (PostgreSQL production)
REDIS_URL             (Redis endpoint)
MEILISEARCH_KEY       (Search engine key)
AWS_ACCESS_KEY_ID     (AWS credentials)
AWS_SECRET_ACCESS_KEY (AWS credentials)
AWS_REGION            (ap-south-1)
ECR_REGISTRY          (AWS ECR URL)
RAZORPAY_KEY_ID       (Payment gateway)
RAZORPAY_KEY_SECRET   (Payment gateway)
FIREBASE_PROJECT_ID   (Push notifications)
SLACK_WEBHOOK         (Deployment alerts)
```

**Note (Free Tier)**: 2,000 free GitHub Actions minutes/month per account. This is enough for ~100 CI runs. Once you exceed, upgrade to Team plan ($21/user/mo) for unlimited minutes.

### 9. Create GitHub Actions Workflows — FREE (2,000 min/mo)
**In each repo, create `.github/workflows/` files:**

Example for `petzonic-api` (backend):
1. Code → Create new file → `.github/workflows/lint.yml`
2. Paste: (see example workflow below)
3. Commit
4. Repeat for `test.yml`, `build.yml`, `deploy-staging.yml`, `deploy-prod.yml`

**Example: `lint.yml` for Node.js repo**
```yaml
name: Lint

on:
  pull_request:
    branches: [develop, main]
  push:
    branches: [develop, main]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
      - uses: actions/setup-node@v4
        with:
          node-version: 22.x
          cache: 'pnpm'
      - run: pnpm install
      - run: pnpm lint
```

**Example: `test.yml`**
```yaml
name: Test

on:
  pull_request:
    branches: [develop, main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
      - uses: actions/setup-node@v4
        with:
          node-version: 22.x
          cache: 'pnpm'
      - run: pnpm install
      - run: pnpm test
```

**Note (Free Tier)**: Each workflow run counts against 2,000 min/month. A typical lint + test takes 2-3 minutes per PR. 100 PRs/month = ~300 minutes (well within free tier).

---

## **Phase 5: Local Cloning (2-4 hours)**

### 10. Set Up SSH Keys (One-time)
```bash
# Generate SSH key (if you don't have one)
ssh-keygen -t ed25519 -C "your.email@company.com"
# Press Enter for default location & no passphrase (for CI/CD convenience)

# Copy public key
cat ~/.ssh/id_ed25519.pub
```

**Add to GitHub:**
1. github.com → top-right profile → Settings → SSH and GPG keys
2. Click **New SSH key**
3. Paste the public key
4. Title: "Dev Machine"
5. Click **Add SSH key**

### 11. Clone All Repos
```bash
mkdir petzonic-workspace && cd petzonic-workspace

git clone git@github.com:petzonic/petzonic-api.git
git clone git@github.com:petzonic/petzonic-web.git
git clone git@github.com:petzonic/petzonic-customer-app.git
git clone git@github.com:petzonic/petzonic-seller-app.git
git clone git@github.com:petzonic/petzonic-infra.git
```

### 12. Global Git Config
```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@company.com"
git config --global pull.rebase true  # Rebase by default
```

### 13. Set Up Each Repo
```bash
cd petzonic-api
cp .env.example .env
# Edit .env with staging credentials (DB, Redis, API keys)
pnpm install
pnpm test
pnpm dev  # Verify it starts on localhost
```

Repeat for each repo.

### 14. Start Local Infrastructure
```bash
cd petzonic-infra
docker-compose -f docker-compose.dev.yml up -d
# Brings up: PostgreSQL, Redis, Meilisearch
```

---

## **Phase 6: Team Handoff & Testing (1-2 days before teams start)**

### 15. Create Onboarding Documentation
**In each repo, create `CONTRIBUTING.md`:**
1. Code → Create new file → `CONTRIBUTING.md`
2. Paste:
```markdown
# Contributing to PetZonic

## Branching
- Always branch from `develop`
- Branch naming: `feature/description`, `fix/description`
- Never commit directly to `main` or `develop`

## PR Workflow
1. Create branch: `git checkout -b feature/my-feature`
2. Make changes, commit, push
3. Open PR: `develop` ← your branch
4. Request review from tech lead or domain expert
5. Address feedback, push new commits (don't force-push)
6. After approval, maintainer merges & deletes branch

## Code Standards
- Run `pnpm lint` before pushing
- All tests must pass: `pnpm test`
- No console.logs in production code
- Follow [coding standards](../coding-standards.md)

## Questions?
See `.github/DEVELOPMENT.md` for local setup help.
```

**In each repo, create `.github/DEVELOPMENT.md`:**
1. Code → Create new file → `.github/DEVELOPMENT.md`
2. Paste setup instructions (prereqs, env vars, how to start dev server, troubleshooting)

### 16. Test End-to-End Workflow
**For backend repo (petzonic-api):**

```bash
git checkout develop
git pull origin develop
git checkout -b feature/github-setup-test
echo "# Test PR for GitHub setup" >> README.md
git add README.md
git commit -m "feat: test PR for github setup"
git push origin feature/github-setup-test
```

Then in GitHub UI:
1. Create PR: `feature/github-setup-test` → `develop`
2. Verify:
   - [ ] CI workflows run (lint, test)
   - [ ] Branch protection shows required reviews
   - [ ] Tech lead can request review (even on Free tier)
   - [ ] Cannot merge without approval
3. Approve PR
4. Merge (squash)
5. Verify:
   - [ ] Branch auto-deleted
   - [ ] If configured: staging deploy triggered

### 17. Schedule Team Onboarding Session
**Duration**: 2-3 hours

**Agenda**:
1. Show GitHub org & 5 repos
2. Explain branching model (feature → develop → main)
3. Demo on screen:
   - Clone repo
   - Run setup (install, tests)
   - Create feature branch
   - Create PR
   - Request review
   - Merge
4. Explain CI/CD: where to see logs, how long runs take
5. Explain deployment: staging (auto) vs production (manual)
6. Q&A

---

## **Free Tier Limitations & Solutions**

| Limitation | Workaround |
|---|---|
| Code owners not auto-enforced | Tech lead manually requests reviews in each PR |
| 2,000 Actions minutes/month | ~100 PRs with lint+test. Upgrade to Team plan if exceeded |
| No code scanning | Use open-source tools (npm audit, trivy) in CI/CD |
| Limited secrets (no org secrets on very old Free) | Use repo secrets instead (slight duplication) |

---

## **Prevent Conflicts & Ensure Smooth Development**

✅ **Tech Lead reviews ALL PRs** — catches design issues early  
✅ **Branch protection enforces PR workflow** — no direct pushes  
✅ **CODEOWNERS file documents domain experts** — team knows who to ask  
✅ **Weekly sync** on blockers, pending PRs, conflicts  
✅ **Staging mirrors production** — catch integration issues before deploy  
✅ **Deployment runbook (in docs)** — defines who deploys when & how  
✅ **Shared dev environment** (docker-compose) — all devs have same local setup
