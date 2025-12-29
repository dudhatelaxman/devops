# Git and GitHub for DevOps Engineers: A Comprehensive Guide

**Last Updated:** December 29, 2025

## Table of Contents
1. [Git Fundamentals](#1-git-fundamentals)
2. [Advanced Git Workflows](#2-advanced-git-workflows)
3. [GitHub Features and Capabilities](#3-github-features-and-capabilities)
4. [Infrastructure as Code (IaC) Version Control](#4-infrastructure-as-code-iac-version-control)
5. [CI/CD Pipeline Integration](#5-cicd-pipeline-integration)
6. [Containerization and Registry Management](#6-containerization-and-registry-management)
7. [Security and Compliance](#7-security-and-compliance)
8. [Monitoring and Logging](#8-monitoring-and-logging)
9. [Collaboration and Team Management](#9-collaboration-and-team-management)
10. [Automation and Scripting](#10-automation-and-scripting)
11. [Disaster Recovery and Backup Strategies](#11-disaster-recovery-and-backup-strategies)
12. [Best Practices and Optimization](#12-best-practices-and-optimization)

---

## 1. Git Fundamentals

### 1.1 Understanding Git Architecture

Git is a distributed version control system that allows DevOps engineers to track, manage, and collaborate on infrastructure code, configuration files, and deployment scripts.

**Core Concepts:**

- **Repository (Repo):** A directory containing all project files and Git history
- **Commit:** A snapshot of changes with a unique SHA-1 hash (e.g., `a1b2c3d4e5f6g7h8i9j0`)
- **Branch:** An independent line of development (default: `main` or `master`)
- **Remote:** A version of the repository hosted on a server (e.g., GitHub, GitLab)
- **Staging Area:** A temporary holding area before committing changes
- **Working Directory:** Local files currently being edited

**Basic Git Workflow:**

```bash
# Initialize a new repository
git init

# Clone an existing repository
git clone https://github.com/dudhatelaxman/devops.git
cd devops

# Check status of working directory
git status

# Stage changes for commit
git add .
git add filename.tf

# Commit staged changes with descriptive message
git commit -m "Add Terraform configuration for VPC"

# View commit history
git log --oneline --graph --all

# Push commits to remote repository
git push origin main

# Pull latest changes from remote
git pull origin main

# Fetch updates without merging
git fetch origin
```

### 1.2 Configuration and Setup

```bash
# Set global user configuration
git config --global user.name "Laxman Dudhat"
git config --global user.email "laxman@devops.local"

# Configure default branch name
git config --global init.defaultBranch main

# Set default editor for commit messages
git config --global core.editor vim

# Configure line ending handling (important for cross-platform teams)
git config --global core.autocrlf true  # Windows
git config --global core.autocrlf input  # macOS/Linux

# Enable credential caching (for automation)
git config --global credential.helper cache
git config --global credential.helper 'cache --timeout=3600'

# View all configuration settings
git config --list
git config --global --list
```

### 1.3 Object Model and Internals

**Git Objects:**

1. **Blob:** Stores file content (binary large object)
2. **Tree:** Directory structure, mapping filenames to blobs
3. **Commit:** Contains tree hash, parent commit, author, and message
4. **Tag:** Permanent reference to a specific commit

```bash
# View object details
git cat-file -t <hash>  # Show object type
git cat-file -p <hash>  # Show object content
git ls-tree <commit>    # Show tree contents
git show <commit>       # Display commit details

# Create annotated tag
git tag -a v1.0.0 -m "Production release"
git push origin v1.0.0

# Create lightweight tag
git tag v1.0.0-rc1
```

---

## 2. Advanced Git Workflows

### 2.1 Branching Strategies

**Git Flow Model:**

This workflow is ideal for managing releases and hotfixes in DevOps environments.

```
main (production) ─────────────────────────────
                    ↑                       ↑
                    └───release/1.0.0───────┘

develop ──────────────────────────────────────
        │    │                    ↑
        └────┼──→ feature/auth────┘
             │
             └──→ feature/monitoring
```

**Implementation:**

```bash
# Initialize git flow
git flow init

# Create and work on feature branch
git flow feature start authentication
git commit -m "Implement OAuth2 authentication"
git flow feature finish authentication

# Create release branch
git flow release start 1.0.0
git commit -m "Bump version to 1.0.0"
git flow release finish 1.0.0

# Create hotfix for production
git flow hotfix start 1.0.1
git commit -m "Fix critical security vulnerability"
git flow hotfix finish 1.0.1
```

**Trunk-Based Development:**

For continuous deployment environments.

```bash
# Create short-lived feature branches (max 3 days)
git checkout -b feature/auto-scaling-policy
# Make small, focused commits
git commit -m "Add CPU-based auto-scaling policy"
# Submit PR for immediate review and merge
# Delete branch after merge
git push origin --delete feature/auto-scaling-policy
```

### 2.2 Merge Strategies

```bash
# Fast-forward merge (preferred for clean history)
git checkout main
git merge --ff feature/logging

# Create merge commit (preserves branch history)
git merge --no-ff feature/database-migration

# Squash merge (combines all commits into one)
git merge --squash develop
git commit -m "Merge all develop changes"

# Rebase and merge (linear history)
git rebase main feature/monitoring
git checkout main
git merge --ff feature/monitoring

# Abort merge if conflicts arise
git merge --abort
```

### 2.3 Advanced Git Commands

```bash
# Cherry-pick specific commit to another branch
git cherry-pick a1b2c3d4

# Interactive rebase to modify commit history
git rebase -i HEAD~5  # Rebase last 5 commits
# Options: pick, reword, squash, fixup, edit, drop

# Find commits by message
git log --grep="security" --oneline

# Find commits that introduced a bug
git bisect start
git bisect bad HEAD
git bisect good v1.0.0
# Test current commit to narrow down

# Show differences between branches
git diff main..develop
git diff --name-only main develop

# Stash changes temporarily
git stash
git stash list
git stash pop  # Apply most recent stash

# Reset to previous state
git reset --soft HEAD~1   # Undo commit, keep changes staged
git reset --mixed HEAD~1  # Undo commit, changes in working directory
git reset --hard HEAD~1   # Completely discard commit

# Revert a commit (creates new commit undoing changes)
git revert a1b2c3d4
```

---

## 3. GitHub Features and Capabilities

### 3.1 Repository Management

**Creating and Configuring Repositories:**

```bash
# Clone with SSH (preferred for DevOps)
git clone git@github.com:dudhatelaxman/devops.git

# Clone with HTTPS
git clone https://github.com/dudhatelaxman/devops.git

# Add multiple remotes for mirroring
git remote add origin git@github.com:dudhatelaxman/devops.git
git remote add backup git@github.com:dudhatelaxman/devops-backup.git
git remote -v  # List all remotes

# Update remote URL
git remote set-url origin git@github.com:dudhatelaxman/devops.git
```

**Repository Settings:**

- **Branch Protection Rules:** Require PR reviews, pass status checks before merge
- **Ruleset Configurations:** Enforce commit signatures, require linear history
- **Default Branch:** Set to `main` or `develop` depending on workflow
- **Archive Policy:** Set up automated archival of old branches

### 3.2 Pull Requests and Code Review

**Creating Effective Pull Requests:**

```bash
# Push feature branch to GitHub
git push -u origin feature/kubernetes-deployment

# Create PR through GitHub UI or CLI
gh pr create --title "Add Kubernetes manifests" \
  --body "Adds deployment, service, and ingress manifests" \
  --base main --head feature/kubernetes-deployment

# List open PRs
gh pr list --state open

# Add reviewers to PR
gh pr edit <PR_NUMBER> --add-reviewer @dudhatelaxman

# View PR details
gh pr view <PR_NUMBER>

# Approve PR
gh pr review <PR_NUMBER> --approve

# Request changes
gh pr review <PR_NUMBER> --request-changes --body "Needs additional testing"

# Merge PR
gh pr merge <PR_NUMBER> --squash
```

**PR Best Practices for DevOps:**

- Keep PRs focused on single responsibility (max 400 lines of change)
- Include clear description of what changed and why
- Link to related issues: `Closes #123`
- Add diagrams or screenshots for infrastructure changes
- Ensure all status checks pass before merging
- Require minimum 2 approvals for production changes

### 3.3 Issues and Project Management

```bash
# Create issue via CLI
gh issue create --title "Implement monitoring dashboard" \
  --body "Add Prometheus metrics and Grafana dashboard" \
  --label enhancement,monitoring

# List issues with filters
gh issue list --state open --label critical
gh issue list --assignee @me

# Close issue
gh issue close <ISSUE_NUMBER>

# Link PR to issue
gh pr create --title "Fix monitoring" --body "Closes #456"
```

---

## 4. Infrastructure as Code (IaC) Version Control

### 4.1 Terraform State and Code Management

**Directory Structure for IaC:**

```
terraform/
├── environments/
│   ├── dev/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── terraform.tfvars
│   ├── staging/
│   └── prod/
├── modules/
│   ├── vpc/
│   ├── eks/
│   ├── rds/
│   └── security_groups/
├── global/
│   ├── main.tf
│   └── variables.tf
└── .gitignore
```

**Git Ignore for Terraform:**

```gitignore
# Local .terraform directories
**/.terraform/*

# .tfstate files
*.tfstate
*.tfstate.*
*.tfvars
*.tfvars.json

# Crash log files
crash.log
crash.*.log

# Exclude all .tfvars files, which are likely to contain sensitive data
*.tfvars
*.tfvars.json

# Ignore override files
override.tf
override.tf.json
*_override.tf
*_override.tf.json

# IDE files
.idea/
*.swp
*.swo
*~

# OS files
.DS_Store
.vscode/
```

**Committing Terraform Code:**

```bash
# Validate configuration before committing
terraform validate
terraform fmt -recursive

# Commit with clear messaging about infrastructure changes
git add terraform/environments/prod/main.tf
git commit -m "Add auto-scaling group for application servers

- Increases min capacity from 3 to 5 instances
- Enables spot instances for cost optimization
- Updates lifecycle policy to protect from accidental termination"

# Create release tags for infrastructure versions
git tag -a terraform-prod-v2.3.0 -m "Production infrastructure update"
git push origin terraform-prod-v2.3.0
```

### 4.2 Ansible and Configuration Management

```yaml
# .gitignore for Ansible
vault-pass.txt
*.vault
~/.ansible/
hosts.local
group_vars/all/secrets.yml
host_vars/*/secrets.yml
```

**Ansible Directory Structure:**

```
ansible/
├── inventories/
│   ├── production/
│   │   ├── hosts.yml
│   │   ├── group_vars/
│   │   └── host_vars/
│   └── staging/
├── roles/
│   ├── docker/
│   ├── prometheus/
│   ├── elasticsearch/
│   └── kubernetes/
├── playbooks/
│   ├── deploy.yml
│   ├── configure.yml
│   └── maintenance.yml
└── ansible.cfg
```

**Committing Ansible Playbooks:**

```bash
# Validate syntax
ansible-playbook --syntax-check playbooks/deploy.yml

# Dry-run to check changes
ansible-playbook -i inventories/production/hosts.yml \
  playbooks/deploy.yml --check

# Commit with descriptive messages
git commit -m "Add Elasticsearch cluster configuration

- Configures 3-node cluster with 16GB heap
- Enables cross-cluster replication for disaster recovery
- Implements security with TLS certificates
- Tested in staging environment"
```

### 4.3 Kubernetes Manifests

**Directory Structure:**

```
kubernetes/
├── base/
│   ├── kustomization.yaml
│   ├── deployment.yaml
│   ├── service.yaml
│   └── configmap.yaml
├── overlays/
│   ├── dev/
│   │   └── kustomization.yaml
│   ├── staging/
│   └── prod/
│       ├── kustomization.yaml
│       ├── hpa.yaml
│       └── pdb.yaml
└── helm-charts/
    ├── myapp/
    │   ├── Chart.yaml
    │   ├── values.yaml
    │   └── templates/
```

**Git Workflow for K8s:**

```bash
# Validate manifests before committing
kubeval -d kubernetes/base/

# Check for security issues
kubesec scan kubernetes/base/deployment.yaml

# Commit with environment context
git commit -m "Update application deployment manifest

- Upgrade image to v2.3.5 (security patch)
- Increase memory limit from 512Mi to 1Gi
- Add readiness probe with 30s timeout
- Environment: staging

Tested with: kubectl apply --dry-run=client -f kubernetes/staging/"
```

---

## 5. CI/CD Pipeline Integration

### 5.1 GitHub Actions for DevOps

**Workflow File Structure:**

```yaml
# .github/workflows/deploy-prod.yml
name: Deploy to Production

on:
  push:
    branches: [main]
    paths:
      - 'terraform/**'
      - 'kubernetes/**'
      - 'ansible/**'
  workflow_dispatch:  # Manual trigger

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}/app

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v2
        with:
          terraform_version: 1.6.0
      
      - name: Validate Terraform
        run: |
          cd terraform/environments/prod
          terraform init -backend=false
          terraform validate
          terraform fmt -check -recursive
      
      - name: Validate Kubernetes Manifests
        run: |
          kubectl apply -f kubernetes/prod/ --dry-run=client -o yaml
      
      - name: Run Security Scan
        run: |
          docker run --rm -v $PWD:/scan aquasec/trivy:latest fs /scan

  deploy:
    needs: validate
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/checkout@v3
      
      - name: Deploy Infrastructure
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        run: |
          cd terraform/environments/prod
          terraform init
          terraform plan -out=tfplan
          terraform apply tfplan
      
      - name: Deploy Application
        run: |
          kubectl apply -f kubernetes/prod/
          kubectl rollout status deployment/app -n production
      
      - name: Notify Deployment
        if: success()
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
          text: 'Production deployment successful'
          webhook_url: ${{ secrets.SLACK_WEBHOOK }}
```

**Common DevOps Workflows:**

```yaml
# .github/workflows/security-scan.yml
name: Security Scanning

on:
  pull_request:
    branches: [main, develop]
  schedule:
    - cron: '0 2 * * *'  # Daily at 2 AM UTC

jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Run Trivy Image Scan
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          scan-ref: '.'
          format: 'sarif'
          output: 'trivy-results.sarif'
      
      - name: Upload SARIF to GitHub Security
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: 'trivy-results.sarif'
      
      - name: Run KICS Kubernetes Scan
        uses: Checkmarx/kics-github-action@master
        with:
          path: 'kubernetes/'
          output_formats: 'sarif,json'
```

### 5.2 Committing CI/CD Configurations

```bash
# Validate workflow syntax
gh workflow list

# Test workflow locally (requires act)
act -j validate

# Commit with clear documentation
git commit -m "Add GitHub Actions CI/CD pipeline

Workflows included:
- Terraform validation and plan
- Container image building and scanning
- Kubernetes manifest validation
- Automated deployment to staging and production
- Security scanning with Trivy and KICS
- Slack notifications on deployment

Triggers:
- Push to main: auto-deploy to production
- Push to develop: auto-deploy to staging
- Pull requests: run all validation checks
- Daily schedule: run security scans"
```

### 5.3 Multi-Environment Deployments

```bash
# Branch naming for environments
feature/new-monitoring  → Deploy to dev
develop                 → Deploy to staging
main                    → Deploy to production

# Tag conventions
git tag -a v1.2.3-staging -m "Staging release"
git tag -a v1.2.3       -m "Production release"

# Commit with environment information
git commit -m "Configure blue-green deployment strategy

Environments:
- Production: rolling updates with 5% traffic
- Staging: canary deployment with health checks
- Development: immediate deployment

Rollback procedure: git revert <commit-hash>"
```

---

## 6. Containerization and Registry Management

### 6.1 Docker and Container Images

**Dockerfile Version Control:**

```dockerfile
# Dockerfile (committed to repository)
FROM alpine:3.18 as builder

RUN apk add --no-cache \
    build-base \
    curl \
    git

WORKDIR /build
COPY . .
RUN make build

FROM alpine:3.18

RUN apk add --no-cache \
    ca-certificates \
    curl \
    dumb-init

COPY --from=builder /build/app /usr/local/bin/app

HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:8080/health || exit 1

ENTRYPOINT ["/usr/bin/dumb-init", "--"]
CMD ["app"]
```

**Docker Compose with Version Control:**

```yaml
# docker-compose.yml
version: '3.9'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    image: myapp:${VERSION:-latest}
    environment:
      - LOG_LEVEL=info
      - DATABASE_URL=postgresql://postgres:5432/myapp
    ports:
      - "8080:8080"
    depends_on:
      postgres:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  postgres:
    image: postgres:15-alpine
    environment:
      - POSTGRES_PASSWORD=secure_password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  postgres_data:
```

**Git Workflow for Container Changes:**

```bash
# Update version in files
echo "VERSION=2.5.0" > .env
sed -i 's/image: myapp:.*/image: myapp:2.5.0/' docker-compose.yml

# Build and test locally
docker-compose build
docker-compose up -d
docker-compose logs -f app

# Commit changes
git commit -m "Release container version 2.5.0

Changes:
- Update base image to Alpine 3.18 (security patches)
- Upgrade Go dependencies to latest stable
- Reduce image size from 85MB to 42MB
- Add SBOM generation for supply chain security

Build: docker build -t myapp:2.5.0 .
Test:  docker-compose up && curl http://localhost:8080/health"

# Tag release
git tag -a v2.5.0 -m "Container release 2.5.0"
git push origin v2.5.0
```

### 6.2 Container Registry Management

**Registry Authentication:**

```bash
# Store registry credentials in .gitignore
echo ".docker/" >> .gitignore
echo "regcred.json" >> .gitignore

# Create Docker config for CI/CD
git commit -m "Add Docker registry configuration

Registry: ghcr.io
Authentication: GitHub Personal Access Token (secrets)
Image naming: ghcr.io/dudhatelaxman/devops:latest"

# Use GitHub Packages in workflow
# Automatically authenticated with GITHUB_TOKEN
docker build -t ghcr.io/dudhatelaxman/devops:latest .
docker push ghcr.io/dudhatelaxman/devops:latest
```

**Artifact Management:**

```bash
# Tag convention for images
git tag v1.2.3        # Stable release
git tag v1.2.3-rc.1   # Release candidate
git tag v1.2.3-beta   # Beta release
git tag v1.2-latest   # Latest minor version

# GitHub Container Registry integration
# Automatically builds on push to main
git push origin main  # Triggers: ghcr.io/user/repo:latest
```

---

## 7. Security and Compliance

### 7.1 SSH Keys and Authentication

**SSH Key Management:**

```bash
# Generate SSH key for DevOps automation
ssh-keygen -t ed25519 -f ~/.ssh/github-devops -C "devops@company.local"

# Add to GitHub account (Settings → SSH and GPG keys)
cat ~/.ssh/github-devops.pub

# Configure git to use specific key
git config --local core.sshCommand "ssh -i ~/.ssh/github-devops"

# Test SSH connection
ssh -T git@github.com

# Add key to GitHub Actions secrets
gh secret set SSH_KEY < ~/.ssh/github-devops
```

### 7.2 GPG Commit Signing

**Setup GPG Signing:**

```bash
# Generate GPG key
gpg --full-generate-key
# Select: RSA and RSA, 4096 bits, no expiration

# List keys
gpg --list-keys
gpg --list-secret-keys

# Export public key to GitHub
gpg --armor --export <KEY_ID> | pbcopy
# Add to GitHub account (Settings → SSH and GPG keys)

# Configure Git to sign commits
git config --global user.signingkey <KEY_ID>
git config --global commit.gpgsign true

# Sign individual commits
git commit -S -m "Add Kubernetes RBAC policies"

# Verify signed commits
git verify-commit <COMMIT_HASH>
git log --show-signature
```

**Enforce Signed Commits in CI:**

```yaml
# .github/workflows/check-signatures.yml
name: Verify Commit Signatures

on: [pull_request]

jobs:
  verify:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0
      
      - name: Verify All Commits Are Signed
        run: |
          git rev-list --all --not --remotes=origin | while read commit; do
            if ! git verify-commit "$commit" 2>/dev/null; then
              echo "Unsigned commit: $commit"
              exit 1
            fi
          done
```

### 7.3 Secrets Management

**Never Commit Secrets:**

```bash
# Create .gitignore for sensitive files
cat > .gitignore << EOF
# Secrets and credentials
.env
.env.local
.env.*.local
secrets/
credentials.json
id_rsa
*.pem
vault-pass.txt
ansible/group_vars/*/vault.yml
terraform/*.auto.tfvars
EOF

# Check for secrets before committing
gh secret scan-git-history

# Use GitHub Secrets in workflows
# Reference with: ${{ secrets.AWS_ACCESS_KEY_ID }}

# Use Terraform variables for sensitive values
git commit -m "Add infrastructure configuration

Note: Run 'terraform apply -var-file=secrets.tfvars' locally
Secrets managed in: TF_VAR_* environment variables"
```

**Secret Rotation in CI/CD:**

```yaml
# .github/workflows/rotate-secrets.yml
name: Rotate Secrets

on:
  schedule:
    - cron: '0 0 1 * *'  # Monthly
  workflow_dispatch:

jobs:
  rotate:
    runs-on: ubuntu-latest
    steps:
      - name: Rotate AWS Access Keys
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        run: |
          aws iam create-access-key --user-name devops-user
          aws iam list-access-keys --user-name devops-user
          # Update secrets in GitHub Actions
```

### 7.4 Compliance and Audit Logging

```bash
# Enable audit logging
git config --global core.logallrefupdates true

# View reflog (local history of branch changes)
git reflog show

# Commit compliance documentation
git commit -m "Add security and compliance documentation

Features implemented:
- GPG signed commits (enforced in main branch)
- Secret scanning in push events
- Audit logging of all code changes
- Access control via GitHub Teams
- Code review requirements (2+ approvals)
- Protected branch rules with required checks

Compliance standards:
- SOC2 compliance for access control
- CIS Kubernetes Benchmark for K8s manifests
- NIST guidelines for cryptographic signing"
```

---

## 8. Monitoring and Logging

### 8.1 Git Activity Monitoring

**Tracking Code Changes:**

```bash
# Monitor repository activity
git log --all --oneline --graph --decorate

# Track contributions per developer
git shortlog -sne

# Find high-impact changes
git log --stat | grep -A 5 "insertion\|deletion"

# Monitor branch activity
git for-each-ref --sort=-committerdate refs/heads/

# Track file modifications over time
git log -p -- terraform/environments/prod/main.tf | head -100
```

**GitHub Insights:**

```bash
# Export commit data for analysis
git log --format="%H %ai %an %s" > commit-history.txt

# Analyze deployment frequency
git log --oneline --since="1 week ago" | grep -i "deploy\|release"

# Track infrastructure changes
git log --oneline --all -- terraform/ kubernetes/ ansible/

# Monitor code review metrics
gh pr list --state closed --limit 50 --format='{{.number}}\t{{.author}}\t{{.reviewDecisions}}'
```

**Commit Log for Audit:**

```bash
# Generate audit report
git log --all --format="%H | %ai | %an | %ae | %s" > audit-log-$(date +%Y%m%d).txt

# Track sensitive data access
git log --all -S "password\|secret\|token" --oneline

# Monitor branch deletions
git reflog --all | grep "delete"

# Create monthly audit trail
git commit -m "Monthly audit report: October 2024

Metrics:
- Total commits: 287
- Contributors: 5
- Releases: 3
- Critical hotfixes: 1
- Code review time: avg 4 hours"
```

### 8.2 GitHub Webhooks and Notifications

**Webhook Configuration:**

```bash
# Create webhook via GitHub API
curl -X POST https://api.github.com/repos/dudhatelaxman/devops/hooks \
  -H "Authorization: token $GITHUB_TOKEN" \
  -d '{
    "name": "web",
    "active": true,
    "events": ["push", "pull_request", "deployment"],
    "config": {
      "url": "https://webhook.example.com/github",
      "content_type": "json"
    }
  }'

# List webhooks
gh api repos/dudhatelaxman/devops/hooks --jq '.[] | {id, events, url}'

# Test webhook
gh api repos/dudhatelaxman/devops/hooks/1/tests/latest
```

**Integration with Monitoring Systems:**

```bash
# Commit webhook configuration
git commit -m "Configure GitHub webhooks for monitoring

Webhooks:
- Push events → Trigger deployment pipeline
- Pull requests → Notify Slack channel
- Releases → Create incident in PagerDuty
- Issues → Sync with Jira

Managed by: GitHub Actions webhook dispatcher"
```

### 8.3 Metrics and KPIs

```bash
# Track deployment frequency
git log --oneline --since="1 month ago" | grep -i "deploy" | wc -l

# Calculate lead time for changes
git log --format="%ai" | head -1  # Latest commit
git log --format="%ai" | tail -1  # Oldest commit

# Monitor change failure rate
gh pr list --state closed --limit 100 | \
  awk '{if ($7 ~ /merged/) print "success"; else print "closed"}' | \
  sort | uniq -c

# Time to resolve incidents
git log --all -S "hotfix\|revert" --oneline --format="%ai %s"

# Track infrastructure configuration changes
git log --oneline -- terraform/ kubernetes/ | wc -l
```

---

## 9. Collaboration and Team Management

### 9.1 Team Structure and Permissions

**GitHub Teams Configuration:**

```bash
# Create organization team for DevOps
gh team create devops-platform \
  --description "Platform engineering and infrastructure team" \
  --privacy closed

# Add members to team
gh team member add devops-platform --member user1 --role member
gh team member add devops-platform --member user2 --role maintainer

# Configure team permissions
gh api repos/dudhatelaxman/devops/teams/devops-platform/permissions -X PUT \
  -f permission=admin

# List team members
gh team member list devops-platform

# Create branch protection rule by team
gh api repos/dudhatelaxman/devops/branches/main/protection \
  -d '{
    "required_pull_request_reviews": {
      "required_approving_review_count": 2,
      "require_code_owner_reviews": true
    },
    "restrict_who_can_push_to_matching_refs": {
      "teams": ["devops-platform"]
    }
  }' -X PUT
```

**CODEOWNERS File:**

```
# .github/CODEOWNERS

# Terraform infrastructure
/terraform/ @dudhatelaxman @team-devops
/terraform/environments/prod/ @dudhatelaxman

# Kubernetes manifests
/kubernetes/ @team-platform
/kubernetes/prod/ @dudhatelaxman

# CI/CD workflows
/.github/workflows/ @dudhatelaxman @team-devops

# Documentation
*.md @team-devops
```

### 9.2 Collaborative Development

**Feature Branch Workflow:**

```bash
# Developer 1: Create feature branch
git checkout -b feature/logging-enhancement
# ... make commits ...
git push -u origin feature/logging-enhancement

# Open PR with detailed description
gh pr create --title "Add structured logging to API" \
  --body "Implements JSON logging with trace IDs for better observability

### What changed
- Added structured logger using logrus
- Implements correlation IDs for distributed tracing
- Reduces log noise by filtering debug messages in production

### Testing
- Tested locally: \`go test ./...\`
- Staging deployed and verified

### Checklist
- [x] Tests pass
- [x] Documentation updated
- [x] No breaking changes
"

# Developer 2: Review PR
gh pr review <PR_NUMBER> \
  --request-changes \
  --body "Please add unit tests for the new logger interface"

# Developer 1: Update PR
git commit -m "Add unit tests for structured logger

- Coverage increased to 89%
- Tests verify JSON output format
- Tests check trace ID propagation"
git push origin feature/logging-enhancement

# Approve and merge
gh pr review <PR_NUMBER> --approve
gh pr merge <PR_NUMBER> --squash --delete-branch
```

**Communication Through Commits:**

```bash
# Clear commit messages for asynchronous communication
git commit -m "Refactor monitoring configuration

BREAKING CHANGE: Prometheus config format changed from YAML to HCL

Migration guide:
1. Update prometheus.yml to prometheus.hcl
2. Run: prometheus-migrate prometheus.yml > prometheus.hcl
3. Test in staging: prometheus --config.file=prometheus.hcl

Affected systems:
- Metrics collection (same endpoints)
- Alert routing (same rules)
- Dashboard queries (compatible)

Timeline:
- Staging: 2025-01-15
- Production: 2025-01-22 (with rollback plan)"
```

### 9.3 Pair Programming and Knowledge Sharing

```bash
# Branch for collaborative work
git checkout -b pair/database-upgrade

# Frequent commits for knowledge capture
git commit -m "Pair programming session: Database upgrade planning

Participants: @alice @bob
Duration: 2 hours

Decisions made:
1. Use pg_upgrade for in-place upgrade to PostgreSQL 15
2. Maintenance window: Sunday 2AM UTC
3. Backup before upgrade and 24-hour retention

Next steps:
- Create runbook for upgrade procedure
- Schedule test run in staging
- Prepare rollback procedure"

# Record important discussions
git commit -m "Document decision: Switch to managed RDS

Context: Performance issues with self-managed PostgreSQL

Decision: Migrate to AWS RDS PostgreSQL with Multi-AZ
Approved by: @infrastructure-lead

Rationale:
- Reduced operational overhead
- Better backup and recovery capabilities
- Automatic failover for HA
- Cost competitive with self-managed

Timeline:
- Staging migration: Jan 10-12
- Dry-run in production: Jan 15-16
- Production migration: Jan 20 (Sunday, 2AM UTC)"
```

---

## 10. Automation and Scripting

### 10.1 Git Hooks for DevOps

**Pre-commit Hooks:**

```bash
#!/bin/bash
# .git/hooks/pre-commit

set -e

echo "Running pre-commit checks..."

# Prevent large files
MAX_SIZE=10485760  # 10MB
for file in $(git diff --cached --name-only); do
    size=$(git cat-file -s :$file 2>/dev/null || echo 0)
    if [ "$size" -gt "$MAX_SIZE" ]; then
        echo "Error: $file is too large ($size bytes)"
        exit 1
    fi
done

# Validate YAML files
git diff --cached --name-only | grep -E '\.(yaml|yml)$' | while read file; do
    if ! command -v yamllint &> /dev/null; then
        pip install yamllint
    fi
    yamllint "$file" || exit 1
done

# Validate Terraform
git diff --cached --name-only | grep -E '\.tf$' | while read file; do
    terraform fmt -check "$file" || exit 1
done

# Check for secrets
if git diff --cached | grep -E 'password|secret|token|apikey' -i; then
    echo "Error: Potential secrets detected in commit"
    exit 1
fi

# Run security scanner
if command -v trivy &> /dev/null; then
    trivy fs . --exit-code 1
fi

echo "Pre-commit checks passed!"
```

**Commit Message Lint:**

```bash
#!/bin/bash
# .git/hooks/commit-msg

# Enforce commit message format

MSG=$(cat $1)
REGEX='^(feat|fix|docs|style|refactor|perf|test|chore|ci|revert)(\(.+\))?: .{20,}'

if ! [[ $MSG =~ $REGEX ]]; then
    echo "Error: Invalid commit message format"
    echo "Expected: <type>(<scope>): <description>"
    echo "Example: feat(monitoring): add Prometheus alerting rules"
    exit 1
fi

# Check for minimum description length
if [[ ${#MSG} -lt 50 ]]; then
    echo "Error: Commit message too short (minimum 50 characters)"
    exit 1
fi

exit 0
```

**Post-merge Hooks:**

```bash
#!/bin/bash
# .git/hooks/post-merge

# Update dependencies after merge
if git diff HEAD@{1} -- package.json > /dev/null; then
    npm install
fi

if git diff HEAD@{1} -- requirements.txt > /dev/null; then
    pip install -r requirements.txt
fi

if git diff HEAD@{1} -- go.mod > /dev/null; then
    go mod download
fi

# Update documentation
if git diff HEAD@{1} -- terraform/ | grep -q 'variable\|output'; then
    terraform-docs markdown terraform/ > INFRASTRUCTURE.md
    git add INFRASTRUCTURE.md || true
fi
```

### 10.2 Automation Scripts

**Automated Release Script:**

```bash
#!/bin/bash
# scripts/release.sh

set -e

VERSION=$1
if [ -z "$VERSION" ]; then
    echo "Usage: ./scripts/release.sh <version>"
    exit 1
fi

echo "Creating release: $VERSION"

# Update version in files
sed -i "s/VERSION=.*/VERSION=$VERSION/" .env
sed -i "s/version: .*/version: $VERSION/" helm-charts/myapp/Chart.yaml
sed -i "s/version: .*/version: $VERSION/" docker-compose.yml

# Commit version bump
git add .env helm-charts/ docker-compose.yml
git commit -m "Release version $VERSION"

# Create annotated tag
git tag -a "$VERSION" -m "Release $VERSION"

# Build and push artifacts
docker build -t myapp:$VERSION .
docker tag myapp:$VERSION ghcr.io/dudhatelaxman/devops:$VERSION
docker push ghcr.io/dudhatelaxman/devops:$VERSION

# Push commits and tags
git push origin main
git push origin $VERSION

echo "Release $VERSION complete!"
```

**Automated Deployment Script:**

```bash
#!/bin/bash
# scripts/deploy.sh

ENVIRONMENT=$1
if [ -z "$ENVIRONMENT" ]; then
    echo "Usage: ./scripts/deploy.sh <dev|staging|prod>"
    exit 1
fi

echo "Deploying to: $ENVIRONMENT"

# Validate configuration
terraform -chdir="terraform/environments/$ENVIRONMENT" validate

# Create Terraform plan
terraform -chdir="terraform/environments/$ENVIRONMENT" plan -out=tfplan

# Apply with approval
read -p "Apply changes? (y/n) " -n 1 -r
if [[ $REPLY =~ ^[Yy]$ ]]; then
    terraform -chdir="terraform/environments/$ENVIRONMENT" apply tfplan
    
    # Deploy to Kubernetes
    kubectl apply -f kubernetes/$ENVIRONMENT/
    kubectl rollout status deployment/app -n $ENVIRONMENT
    
    # Create deployment record
    git log --oneline -1 >> .deployments
    git add .deployments
    git commit -m "Record deployment to $ENVIRONMENT"
    git push origin main
fi
```

### 10.3 Continuous Integration Automation

**Matrix Testing:**

```yaml
# .github/workflows/test.yml
name: Test

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        go-version: ['1.20', '1.21', '1.22']
        postgres-version: ['13', '14', '15']
    
    services:
      postgres:
        image: postgres:${{ matrix.postgres-version }}
        env:
          POSTGRES_PASSWORD: postgres
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432

    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-go@v4
        with:
          go-version: ${{ matrix.go-version }}
      - run: go test ./...
      - run: go build -o app .
```

---

## 11. Disaster Recovery and Backup Strategies

### 11.1 Repository Backup and Mirroring

**Automated Backup Strategy:**

```bash
#!/bin/bash
# scripts/backup-repository.sh

REPO_URL="https://github.com/dudhatelaxman/devops.git"
BACKUP_DIR="/backups/git-repos"
REPO_NAME="devops"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"

# Clone bare repository
git clone --bare "$REPO_URL" "$BACKUP_DIR/$REPO_NAME-$TIMESTAMP.git"

# Create compressed archive
tar -czf "$BACKUP_DIR/$REPO_NAME-$TIMESTAMP.tar.gz" \
  -C "$BACKUP_DIR" "$REPO_NAME-$TIMESTAMP.git"

# Keep only last 30 days
find "$BACKUP_DIR" -name "*.tar.gz" -mtime +30 -delete

# Backup to S3
aws s3 cp "$BACKUP_DIR/$REPO_NAME-$TIMESTAMP.tar.gz" \
  s3://backup-bucket/git-repos/

echo "Repository backup completed: $TIMESTAMP"
```

**Mirror Repository:**

```bash
# Create mirror repository
git clone --mirror https://github.com/dudhatelaxman/devops.git \
  /path/to/devops.git

# Add to cron for automatic updates
0 */6 * * * cd /path/to/devops.git && git fetch --all

# Configuration file for mirror
git commit -m "Configure repository mirror

Backup location: /backups/devops.git
Mirror server: backup.example.com
Sync frequency: Every 6 hours
Retention policy: 30 days"
```

### 11.2 Disaster Recovery Procedures

**Recovery from Accidental Deletion:**

```bash
# View deleted branches
git reflog show origin/<branch>

# Recover deleted branch
git checkout -b <recovered-branch> <reflog-entry>

# If reflog is unavailable, use backup
git clone /backups/devops-backup.git
git checkout <commit-hash>
git push origin <recovered-branch>

# Document recovery
git commit -m "Recovery: Restored branch from backup

Lost branch: feature/old-feature
Recovery date: 2025-12-29
Method: From S3 backup dated 2025-12-28
Verification: All commits recovered, tested locally"
```

**Recovery from Corrupted Repository:**

```bash
# Check repository integrity
git fsck --verbose

# Verify all objects
git verify-pack -v .git/objects/pack/*.idx

# Garbage collection
git gc --aggressive

# Restore from backup
tar -xzf /backups/devops-backup.tar.gz
cd devops.git
git clone . /path/to/recovery/devops

# Document incident
git commit -m "Incident: Repository corruption recovery

Date: 2025-12-29
Issue: Corrupted objects in pack files
Solution: Restored from S3 backup
Method: git fsck && git gc --aggressive
Status: All commits verified
Impact: No data loss"
```

### 11.3 Infrastructure Configuration Backups

**Backup Terraform State:**

```bash
# Backup remote state
terraform state pull > terraform.state.backup

# Version control state (encrypted)
git-crypt init
git-crypt add-gpg-user <KEY_ID>

echo "*.state.backup filter=git-crypt diff=git-crypt" >> .gitattributes

git add terraform.state.backup .gitattributes
git commit -m "Add encrypted Terraform state backup"

# Schedule automated backups
aws s3 sync terraform/ s3://backup-bucket/terraform/ \
  --exclude '*.tfstate*' --exclude '.terraform/*'
```

**Backup Kubernetes Configurations:**

```bash
#!/bin/bash
# scripts/backup-kubernetes.sh

BACKUP_DIR="kubernetes-backup-$(date +%Y%m%d_%H%M%S)"
mkdir -p "$BACKUP_DIR"

# Backup all resources
kubectl get all --all-namespaces -o yaml > "$BACKUP_DIR/all-resources.yaml"
kubectl get secrets --all-namespaces -o yaml > "$BACKUP_DIR/secrets.yaml"
kubectl get pvc --all-namespaces -o yaml > "$BACKUP_DIR/volumes.yaml"

# Commit to repository
git add "$BACKUP_DIR"
git commit -m "Kubernetes cluster backup: $(date +%Y-%m-%d)

Namespaces backed up:
- production
- staging
- monitoring

Resources:
- Deployments: $(kubectl get deployments --all-namespaces --no-headers | wc -l)
- StatefulSets: $(kubectl get statefulsets --all-namespaces --no-headers | wc -l)
- Services: $(kubectl get services --all-namespaces --no-headers | wc -l)
- ConfigMaps: $(kubectl get configmaps --all-namespaces --no-headers | wc -l)

Backup location: $BACKUP_DIR/all-resources.yaml"

git push origin main
```

---

## 12. Best Practices and Optimization

### 12.1 Git Best Practices

**Clean and Efficient Repository:**

```bash
# Maintain lean repository size
# Remove large files from history
git filter-branch --tree-filter 'find . -size +100M -delete' HEAD
git gc --aggressive

# Use shallow clones for CI/CD
git clone --depth 1 --branch main https://github.com/dudhatelaxman/devops.git

# Partial clone (for large monorepos)
git clone --filter=blob:none https://github.com/dudhatelaxman/devops.git

# Regular maintenance
git maintenance run --auto

# Clean unused branches
git fetch --prune origin
git branch -vv | grep gone | awk '{print $1}' | xargs git branch -D
```

**Commit Hygiene:**

```
✅ Good Commits:
- Single responsibility principle
- Atomic changes (complete, logical unit)
- Clear message describing "what" and "why"
- Properly formatted (50 char title, 72 char body)
- Includes references to issues/tickets

❌ Bad Commits:
- Multiple unrelated changes
- Incomplete features
- Vague messages ("fix stuff", "wip")
- Very large change sets (>500 lines)
- No context or rationale
```

**Conventional Commits:**

```
<type>(<scope>): <subject>
<BLANK LINE>
<body>
<BLANK LINE>
<footer>

Types:
- feat: New feature
- fix: Bug fix
- docs: Documentation
- style: Code style
- refactor: Code refactoring
- perf: Performance improvements
- test: Tests
- chore: Build, dependencies
- ci: CI/CD changes
- infra: Infrastructure

Example:
feat(monitoring): add Prometheus alerting rules

Adds alert rules for:
- High CPU usage (>80% for 5min)
- High memory usage (>90%)
- Pod restart loops

Closes #1234
```

### 12.2 Performance Optimization

**Large Repository Management:**

```bash
# Use worktrees for parallel work
git worktree add ../devops-hotfix hotfix-branch
cd ../devops-hotfix
# Make changes in parallel to main checkout

# Remove worktree
git worktree remove ../devops-hotfix

# Sparse checkout for large monorepos
git sparse-checkout init
git sparse-checkout set terraform/environments/prod

# Shallow clone for faster operations
git clone --depth 5 --branch main https://github.com/dudhatelaxman/devops.git
```

**Optimize Git Configuration:**

```bash
# Enable compression for large objects
git config --global core.compression 9

# Parallel transfers
git config --global fetch.parallel 8
git config --global push.parallel 8

# Use Delta Islands (for large repositories)
git config --global feature.experimental true
git config --global feature.island true

# Cache credentials securely
git config --global credential.helper 'cache --timeout=7200'

# Optimize pack files
git config --global gc.autoPackLimit 50
git config --global gc.autodetach true
```

### 12.3 Documentation and Knowledge Management

**README for DevOps Team:**

```markdown
# DevOps Repository Structure

## Quick Start
1. Clone: `git clone git@github.com:dudhatelaxman/devops.git`
2. Install tools: `make install-tools`
3. Set up environment: `cp .env.example .env`
4. Deploy: `make deploy ENV=dev`

## Directory Structure
- `terraform/` - Infrastructure as Code
- `kubernetes/` - Container orchestration
- `ansible/` - Configuration management
- `scripts/` - Automation scripts
- `.github/workflows/` - CI/CD pipelines

## Contributing
1. Create feature branch: `git checkout -b feature/my-feature`
2. Make changes with clear commits
3. Push and create pull request
4. Ensure all checks pass
5. Wait for 2 approvals before merging

## Deployment
- `main` branch: Auto-deploys to production
- `develop` branch: Auto-deploys to staging
- Tags `v*` trigger release builds

## Support
- Team: @devops-team
- Slack: #devops-platform
- Runbooks: /docs/runbooks/
```

**Commit Message Examples for Documentation:**

```bash
# Document architectural decision
git commit -m "ADR-001: Kubernetes for orchestration

Decision: Use Kubernetes instead of Docker Swarm
Status: Accepted
Date: 2025-12-15

Context:
- Need for multi-cloud deployment
- Team expertise with Kubernetes
- Better ecosystem and tooling

Decision:
- Migrate from Docker Swarm to Kubernetes
- Use managed services (EKS, GKE) when possible
- Standardize on Helm for package management

Consequences:
- Increased operational complexity
- Better scalability and resilience
- Improved developer experience

Alternatives considered:
1. Docker Swarm - simpler but limited
2. Nomad - more complex, less community support"

# Document runbook
git commit -m "Add runbook: Database failover procedure

Runbook: database-failover.md
Updated: 2025-12-29
Last tested: 2025-12-15

Procedures documented:
1. Detection of primary failure
2. Promotion of standby to primary
3. Failover to read replica if needed
4. Connection string updates
5. Verification steps
6. Rollback procedure

Expected duration: 2-5 minutes
Risk level: Medium
Requires approval: YES
"
```

### 12.4 Team Workflow Standards

**Definition of Done:**

```
Code change is complete when:
- ✅ Code written and tested locally
- ✅ Unit tests added (min 80% coverage)
- ✅ Integration tests passed
- ✅ Code review approved (2+ reviewers)
- ✅ All CI checks passed
- ✅ Documentation updated
- ✅ Runbooks updated if needed
- ✅ Merged to develop/main
- ✅ Deployed to appropriate environment
- ✅ Monitoring and alerts verified
```

**Change Management Process:**

```
1. Planning (1-2 days)
   - Create issue with detailed requirements
   - Get stakeholder approval
   - Estimate effort and timeline

2. Development (varies)
   - Create feature branch
   - Regular commits with clear messages
   - Self-review before PR

3. Code Review (4-24 hours)
   - Submit pull request
   - Address review comments
   - Get required approvals

4. Testing (varies)
   - Run in staging environment
   - Perform load testing if needed
   - Verify monitoring/alerts

5. Deployment (15 mins - 2 hours)
   - Schedule deployment window
   - Execute deployment
   - Monitor metrics and logs
   - Have rollback ready

6. Post-Deployment (24-48 hours)
   - Monitor for issues
   - Gather feedback
   - Document lessons learned
```

**Git Workflow Checklist:**

```bash
# Before pushing
git status                              # Check what's staged
git diff --cached                       # Review changes
git commit --dry-run                    # Validate message format

# Before creating PR
git log origin/main..HEAD               # View commits to be merged
git diff origin/main                    # Compare with main branch
gh workflow list                        # Verify CI/CD will run

# PR Creation
gh pr create                            # Create with template
gh pr view <number>                     # Verify details
git push origin <branch>                # Push if not auto-pushed

# PR Review
gh pr review <number> --request-changes # Request changes if needed
gh pr review <number> --approve         # Approve if satisfied

# Merge and Cleanup
gh pr merge <number> --squash            # Merge PR
git checkout main                       # Switch to main
git pull origin main                    # Update local main
git branch -D feature-branch            # Delete local branch
git push origin --delete feature-branch # Delete remote branch
```

---

## Summary and Key Takeaways

### Essential DevOps Git Skills:
1. **Master branching strategies** - Choose git-flow or trunk-based based on team needs
2. **Automate everything** - Use GitHub Actions for CI/CD pipelines
3. **Secure your infrastructure** - GPG sign commits, manage secrets separately
4. **Document decisions** - Use commits to record architectural decisions
5. **Maintain clean history** - Squash, rebase, and keep commits atomic
6. **Collaborate effectively** - Use PRs, code reviews, and clear communication
7. **Plan for disasters** - Backup repositories and test recovery procedures
8. **Monitor and audit** - Track all infrastructure changes and deployments
9. **Optimize workflows** - Use hooks, scripts, and automation to save time
10. **Continuous improvement** - Review metrics and improve processes regularly

### Tools and Resources:
- **CLI Tools:** `gh` (GitHub CLI), `git-flow`, `act` (local workflow testing)
- **Monitoring:** GitHub Insights, git-log analysis, webhook integrations
- **Security:** GitGuardian, Trivy, KICS, Secret Scanning
- **Automation:** GitHub Actions, Terraform, Ansible, Helm
- **Documentation:** READMEs, Wikis, Runbooks, ADRs

### Continuous Learning:
- Review commits weekly for best practices
- Audit repository every month
- Update documentation with new processes
- Share knowledge with team through pair programming
- Attend DevOps community events

---

**Document Version:** 1.0  
**Last Updated:** December 29, 2025  
**Author:** DevOps Team  
**License:** Internal Use  
**Next Review:** March 29, 2026
