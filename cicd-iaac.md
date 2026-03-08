# CI/CD & Infrastructure as Code

## CI/CD Pipeline (GitHub Actions)

### Overview

The CI/CD pipeline automates frontend library builds, testing, validation, and deployment to npm for the single-page-express modernization project. The pipeline ensures quality gates through multi-stage verification with a 2-approver production gate for npm package publishing.

### Stage 1: Build & Lint

**Purpose:** Build frontend library bundles, run linting, and perform static analysis.

**Trigger:** Push to any branch, Pull Request to main/develop

**Jobs:**

#### Job 1.1: Frontend Build
```yaml
name: Frontend Build
runs-on: ubuntu-latest
strategy:
  matrix:
    node-version: [18.x, 20.x, 22.x]
steps:
  - uses: actions/checkout@v4
  - name: Use Node.js ${{ matrix.node-version }}
    uses: actions/setup-node@v4
    with:
      node-version: ${{ matrix.node-version }}
      cache: 'npm'
  - name: Install Dependencies
    run: npm ci
  - name: Build Library Bundles
    run: npm run build
  - name: Verify Bundle Output
    run: |
        ls -lh dist/
        echo "Bundle sizes:"
        du -h dist/
  - name: Upload Build Artifacts
    uses: actions/upload-artifact@v4
    with:
      name: dist-bundles-${{ matrix.node-version }}
      path: dist/
      retention-days: 7
```

#### Job 1.2: Lint & Static Analysis
```yaml
name: Lint & Static Analysis
runs-on: ubuntu-latest
steps:
  - uses: actions/checkout@v4
  - name: Use Node.js 22.x
    uses: actions/setup-node@v4
    with:
      node-version: 22.x
      cache: 'npm'
  - name: Install Dependencies
    run: npm ci
  - name: Run Standard.js Lint
    run: npm run lint
  - name: Run ESLint
    run: npx eslint .
  - name: Check for Security Vulnerabilities
    run: npm audit --audit-level=moderate
  - name: Check for Outdated Dependencies
    run: npx npm-check-updates
```

**Exit Criteria:**
- All bundle formats generated successfully (5 variants)
- Bundle size ≤ 100KB (2x current baseline ~50KB)
- No linting errors (Standard.js, ESLint)
- No HIGH/CRITICAL npm security vulnerabilities

**Artifacts:**
- `dist/` directory with all bundle variants
- Lint reports

---

### Stage 2: Test & Validation

**Purpose:** Run full test suite including Playwright E2E tests, pytest for FastAPI backend, and validation agent checks.

**Trigger:** Push to any branch, Pull Request to main/develop

**Depends On:** Stage 1: Build & Lint (must pass)

**Jobs:**

#### Job 2.1: Playwright E2E Tests
```yaml
name: Playwright E2E Tests
runs-on: ${{ matrix.os }}
strategy:
  fail-fast: false
  matrix:
    node-version: [22.x]
    os: [ubuntu-latest, macos-latest, windows-latest]
    shard: [1/2, 2/2]
steps:
  - uses: actions/checkout@v4
  - name: Use Node.js ${{ matrix.node-version }}
    uses: actions/setup-node@v4
    with:
      node-version: ${{ matrix.node-version }}
      cache: 'npm'
  - name: Install Dependencies
    run: npm ci
  - name: Download Build Artifacts
    uses: actions/download-artifact@v4
    with:
      name: dist-bundles-22.x
      path: dist/
  - name: Install Playwright Browsers
    run: npx playwright install --with-deps
  - name: Start Express Complex Sample Server
    run: |
        npm run build
        cd sampleApps/express-complex
        node server.js &
        sleep 10
  - name: Run Playwright Tests
    run: npx playwright test --shard=${{ matrix.shard }} --workers=1
    env:
      CI: true
  - name: Upload Test Results
    if: always()
    uses: actions/upload-artifact@v4
    with:
      name: playwright-results-${{ matrix.os }}-${{ matrix.shard }}
      path: test-results/
      retention-days: 7
  - name: Upload Playwright Report
    if: always()
    uses: actions/upload-artifact@v4
    with:
      name: playwright-report-${{ matrix.os }}
      path: playwright-report/
      retention-days: 7
```

#### Job 2.2: FastAPI Backend Tests (Post-Migration)
```yaml
name: FastAPI Backend Tests
runs-on: ubuntu-latest
steps:
  - uses: actions/checkout@v4
  - name: Set up Python 3.11
    uses: actions/setup-python@v5
    with:
      python-version: '3.11'
      cache: 'pip'
  - name: Install Python Dependencies
    run: |
        cd sampleApps/fastapi
        pip install -r requirements.txt
  - name: Run pytest
    run: |
        cd sampleApps/fastapi
        pytest tests/ -v --cov=. --cov-report=xml
  - name: Upload Coverage Report
    uses: codecov/codecov-action@v4
    with:
      file: ./sampleApps/fastapi/coverage.xml
      flags: fastapi-backend
```

#### Job 2.3: Validation Agent Checks
```yaml
name: Validation Agent Checks
runs-on: ubuntu-latest
steps:
  - uses: actions/checkout@v4
    with:
      fetch-depth: 0
  - name: Use Node.js 22.x
    uses: actions/setup-node@v4
    with:
      node-version: 22.x
      cache: 'npm'
  - name: Install Dependencies
    run: npm ci
  - name: Run Validation Agent
    run: npm run validate-pr
    env:
      PR_NUMBER: ${{ github.event.pull_request.number }}
      PR_URL: ${{ github.event.pull_request.html_url }}
  - name: Upload Validation Report
    if: always()
    uses: actions/upload-artifact@v4
    with:
      name: validation-report
      path: validation-results/
      retention-days: 7
```

#### Job 2.4: Bundle Size Validation
```yaml
name: Bundle Size Validation
runs-on: ubuntu-latest
steps:
  - uses: actions/checkout@v4
  - name: Use Node.js 22.x
    uses: actions/setup-node@v4
    with:
      node-version: 22.x
      cache: 'npm'
  - name: Install Dependencies
    run: npm ci
  - name: Download Build Artifacts
    uses: actions/download-artifact@v4
    with:
      name: dist-bundles-22.x
      path: dist/
  - name: Check Bundle Sizes
    run: |
        MAX_SIZE=102400  # 100KB in bytes
        for file in dist/*.{mjs,cjs,js}; do
          size=$(stat -f%s "$file")
          if [ $size -gt $MAX_SIZE ]; then
            echo "ERROR: $file exceeds maximum size (${size} > ${MAX_SIZE})"
            exit 1
          else
            echo "PASS: $file size ${size} bytes (limit: ${MAX_SIZE})"
          fi
        done
  - name: Compare with Baseline
    run: |
        BASELINE_SIZE=51200  # 50KB in bytes
        for file in dist/*.{mjs,cjs,js}; do
          size=$(stat -f%s "$file")
          ratio=$(echo "scale=2; $size * 100 / $BASELINE_SIZE" | bc)
          echo "$file: ${ratio}% of baseline"
        done
```

**Exit Criteria:**
- All Playwright tests pass across all OS/browser combinations
- FastAPI backend tests pass with ≥ 80% coverage
- Validation agent score ≥ 70 (no CRITICAL failures)
- Bundle sizes within constraints

**Artifacts:**
- Playwright test results and reports
- pytest coverage reports
- Validation agent reports

---

### Stage 3: Staging Deploy

**Purpose:** Deploy to npm registry (test/preview) for integration testing and staging validation.

**Trigger:** Merge to `develop` branch

**Depends On:** Stage 2: Test & Validation (must pass)

**Jobs:**

#### Job 3.1: Preview Deploy to npm
```yaml
name: Preview Deploy to npm
runs-on: ubuntu-latest
environment:
  name: npm-preview
  url: https://www.npmjs.com/package/single-page-express
steps:
  - uses: actions/checkout@v4
  - name: Use Node.js 22.x
    uses: actions/setup-node@v4
    with:
      node-version: 22.x
      registry-url: 'https://registry.npmjs.org'
  - name: Install Dependencies
    run: npm ci
  - name: Download Build Artifacts
    uses: actions/download-artifact@v4
    with:
      name: dist-bundles-22.x
      path: dist/
  - name: Bump Version (Preview)
    run: |
        npm version prerelease --preid=preview
        VERSION=$(node -p "require('./package.json').version")
        echo "VERSION=${VERSION}" >> $GITHUB_ENV
  - name: Publish to npm (Preview)
    run: npm publish --tag preview
    env:
      NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
  - name: Comment PR with Preview Link
    uses: actions/github-script@v7
    if: github.event_name == 'pull_request'
    with:
      script: |
        github.rest.issues.createComment({
          issue_number: context.issue.number,
          owner: context.repo.owner,
          repo: context.repo.repo,
          body: '🚀 Preview package published to npm!\n\nVersion: ${{ env.VERSION }}\n\nInstall with: `npm install single-page-express@preview`'
        })
```

#### Job 3.2: Integration Tests Against Preview Package
```yaml
name: Integration Tests Against Preview
runs-on: ubuntu-latest
needs: preview-deploy-to-npm
steps:
  - uses: actions/checkout@v4
  - name: Use Node.js 22.x
    uses: actions/setup-node@v4
    with:
      node-version: 22.x
  - name: Install Preview Package
    run: npm install single-page-express@preview --force
  - name: Create Integration Test Project
    run: |
        mkdir test-integration
        cd test-integration
        npm init -y
        npm install single-page-express@preview
        cat > test-basic.js << 'EOF'
        const singlePageExpress = require('single-page-express')
        const app = singlePageExpress()
        console.log('✅ Basic import works')
        console.log('✅ App instance created')
        console.log('Express version:', app.expressVersion)
        EOF
  - name: Run Integration Tests
    run: node test-integration/test-basic.js
  - name: Test Sample Apps with Preview
    run: |
        cd sampleApps/express-complex
        npm install single-page-express@preview
        npm test
```

**Exit Criteria:**
- Preview package published to npm with `@preview` tag
- Integration tests pass against preview package
- Sample applications work with preview package

**Artifacts:**
- npm package URL and version
- Integration test results

---

### Stage 4: Production Gate (2-Approver)

**Purpose:** Final approval gate requiring 2 authorized approvers before npm production deployment.

**Trigger:** Pull Request from `develop` to `main` branch

**Depends On:** Stage 3: Staging Deploy (must pass)

**Environment:** `npm-production`

**Branch Protection Rules:**
- Requires 2 approving reviews
- Dismisses stale approvals when new commits push
- Requires status checks to pass
- Requires branches to be up to date before merging

**Job:**

#### Job 4.1: Production Gate Checks
```yaml
name: Production Gate Checks
runs-on: ubuntu-latest
environment:
  name: npm-production
  url: https://www.npmjs.com/package/single-page-express
steps:
  - uses: actions/checkout@v4
    with:
      fetch-depth: 0
  - name: Verify 2 Approvals
    run: |
        APPROVALS=$(gh pr view ${{ github.event.pull_request.number }} --json approvals --jq '.approvals | length')
        if [ "$APPROVALS" -lt 2 ]; then
          echo "ERROR: Requires 2 approvals, found $APPROVALS"
          exit 1
        else
          echo "✅ 2 approvals verified"
        fi
    env:
      GH_TOKEN: ${{ github.token }}
  - name: Verify No CRITICAL Failures
    run: |
        # Check if validation agent found any CRITICAL issues
        if [ -f validation-results/critical-failures.txt ]; then
          echo "ERROR: CRITICAL failures detected, cannot proceed to production"
          cat validation-results/critical-failures.txt
          exit 1
        else
          echo "✅ No CRITICAL failures"
        fi
  - name: Verify Validation Score ≥ 70
    run: |
        SCORE=$(cat validation-results/score.txt 2>/dev/null || echo "0")
        if (( $(echo "$SCORE < 70" | bc -l) )); then
          echo "ERROR: Validation score $SCORE < 70, cannot proceed to production"
          exit 1
        else
          echo "✅ Validation score $SCORE ≥ 70"
        fi
  - name: Verify Bundle Size Constraints
    run: |
        MAX_SIZE=102400  # 100KB in bytes
        for file in dist/*.{mjs,cjs,js}; do
          size=$(stat -f%s "$file" 2>/dev/null || echo "0")
          if [ $size -gt $MAX_SIZE ]; then
            echo "ERROR: $file exceeds maximum size"
            exit 1
          fi
        done
        echo "✅ Bundle sizes within constraints"
  - name: Verify All Tests Passed
    run: |
        # Check if test artifacts exist and passed
        if [ ! -f test-results/test-results.json ]; then
          echo "ERROR: Test results not found"
          exit 1
        fi
        FAILURES=$(jq '.numFailedTests' test-results/test-results.json)
        if [ "$FAILURES" -gt 0 ]; then
          echo "ERROR: $FAILURES test failures"
          exit 1
        else
          echo "✅ All tests passed"
        fi
  - name: Production Gate Summary
    run: |
        echo "========================================="
        echo "✅ Production Gate Checks Passed"
        echo "========================================="
        echo "Ready for npm production deployment"
        echo "Requires manual approval from 2 authorized reviewers"
```

#### Job 4.2: Production Deploy (Manual Approval)
```yaml
name: Production Deploy to npm
runs-on: ubuntu-latest
needs: production-gate-checks
environment:
  name: npm-production
  url: https://www.npmjs.com/package/single-page-express
# Requires manual trigger after approvals
if: |
  github.event_name == 'workflow_dispatch' &&
  github.event.inputs.deploy == 'production'
steps:
  - uses: actions/checkout@v4
  - name: Use Node.js 22.x
    uses: actions/setup-node@v4
    with:
      node-version: 22.x
      registry-url: 'https://registry.npmjs.org'
  - name: Install Dependencies
    run: npm ci
  - name: Download Build Artifacts
    uses: actions/download-artifact@v4
    with:
      name: dist-bundles-22.x
      path: dist/
  - name: Bump Version (Production)
    run: |
        VERSION=$(node -p "require('./package.json').version")
        # Remove -preview.X if exists
        NEW_VERSION=$(echo "$VERSION" | sed 's/-preview\.[0-9]*//')
        npm version $NEW_VERSION
        echo "NEW_VERSION=${NEW_VERSION}" >> $GITHUB_ENV
  - name: Update CHANGELOG.md
    run: |
        NEW_VERSION=${{ env.NEW_VERSION }}
        DATE=$(date +%Y-%m-%d)
        cat >> CHANGELOG.md << EOF

        ## [${NEW_VERSION}] - ${DATE}

        ### Added
        - FastAPI backend support with Python 3.8+
        - YAML route configuration format for isomorphic routing
        - Template sharing between Python (FastAPI) and JavaScript
        - Updated sample applications for FastAPI integration

        ### Changed
        - Migration guide from Express to FastAPI backend
        - Updated documentation for dual-backend support

        ### Deprecated
        - Express backend samples (will be removed in v3.0.0)

        ### Fixed
        - N/A
        EOF
  - name: Commit Version Bump & CHANGELOG
    run: |
        git config user.name "CI/CD Bot"
        git config user.email "ci-bot@rooseveltframework.org"
        git add package.json CHANGELOG.md
        git commit -m "chore(release): bump to v${{ env.NEW_VERSION }}"
        git push origin main
  - name: Create Git Tag
    run: |
        git tag -a "v${{ env.NEW_VERSION }}" -m "Release v${{ env.NEW_VERSION }}"
        git push origin "v${{ env.NEW_VERSION }}"
  - name: Publish to npm (Production)
    run: npm publish --access public
    env:
      NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
  - name: Create GitHub Release
    uses: softprops/action-gh-release@v2
    with:
      files: dist/*
      generate_release_notes: true
      tag_name: v${{ env.NEW_VERSION }}
      name: v${{ env.NEW_VERSION }}
      body: |
        ## 🎉 single-page-express v${{ env.NEW_VERSION }}

        ### What's New
        - FastAPI backend support with Python 3.8+
        - YAML route configuration format for isomorphic routing
        - Template sharing between Python (FastAPI) and JavaScript
        - Updated sample applications for FastAPI integration

        ### Migration Guide
        See [MIGRATION.md](https://github.com/rooseveltframework/single-page-express/blob/main/MIGRATION.md) for migrating from Express to FastAPI backend.

        ### Installation
        ```bash
        npm install single-page-express
        ```

        ### Documentation
        - [README](https://github.com/rooseveltframework/single-page-express)
        - [USAGE.md](https://github.com/rooseveltframework/single-page-express/blob/main/USAGE.md)
        - [MIGRATION.md](https://github.com/rooseveltframework/single-page-express/blob/main/MIGRATION.md)

    env:
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  - name: Post Release Notification
    uses: 8398a7/action-slack@v3
    with:
      status: ${{ job.status }}
      text: |
        🎉 single-page-express v${{ env.NEW_VERSION }} released to npm!

        📦 Install: `npm install single-page-express@latest`
        📖 Docs: https://github.com/rooseveltframework/single-page-express
      webhook_url: ${{ secrets.SLACK_WEBHOOK }}
      channel: '#releases'
      username: 'Release Bot'
```

**Exit Criteria:**
- 2 authorized approvers approve the PR
- No CRITICAL validation failures
- Validation score ≥ 70
- All tests passed
- Bundle sizes within constraints
- Manual workflow dispatch for deployment

**Artifacts:**
- npm production package URL
- GitHub release URL
- CHANGELOG.md updates
- Git tag created

---

## Terraform IaC (AWS)

### Overview

Terraform infrastructure for hosting sample applications and documentation sites. Note: The single-page-express library itself is a frontend npm package and does not require AWS infrastructure for distribution. However, sample applications and documentation sites benefit from cloud deployment for demonstration and testing purposes.

### Provider Configuration

```hcl
# terraform/providers.tf

terraform {
  required_version = ">= 1.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }

  backend "s3" {
    bucket = "single-page-express-terraform-state"
    key    = "infrastructure/terraform.tfstate"
    region = "us-east-1"
    encrypt = true
  }
}

provider "aws" {
  region = var.aws_region
  default_tags {
    Project     = "single-page-express"
    Environment = var.environment
    ManagedBy   = "terraform"
  }
}
```

### Variables

```hcl
# terraform/variables.tf

variable "aws_region" {
  description = "AWS region for deployment"
  type        = string
  default     = "us-east-1"
}

variable "environment" {
  description = "Environment (staging/production)"
  type        = string
  default     = "staging"

  validation {
    condition     = contains(["staging", "production"], var.environment)
    error_message = "Environment must be staging or production"
  }
}

variable "eks_cluster_name" {
  description = "Name of EKS cluster"
  type        = string
  default     = "spe-eks-cluster"
}

variable "eks_node_count" {
  description = "Number of worker nodes in EKS cluster"
  type        = number
  default     = 3

  validation {
    condition     = var.eks_node_count > 0 && var.eks_node_count <= 10
    error_message = "Node count must be between 1 and 10"
  }
}

variable "eks_node_instance_type" {
  description = "EC2 instance type for EKS nodes"
  type        = string
  default     = "t3.medium"
}

variable "rds_instance_class" {
  description = "RDS instance class"
  type        = string
  default     = "db.t3.micro"
}

variable "msk_broker_count" {
  description = "Number of Kafka brokers"
  type        = number
  default     = 2

  validation {
    condition     = var.msk_broker_count >= 2 && var.msk_broker_count <= 6
    error_message = "Broker count must be between 2 and 6"
  }
}

variable "redis_node_type" {
  description = "ElastiCache node type"
  type        = string
  default     = "cache.t3.micro"
}
```

### Outputs

```hcl
# terraform/outputs.tf

output "eks_cluster_endpoint" {
  description = "EKS cluster API endpoint"
  value       = module.eks.cluster_endpoint
}

output "eks_cluster_name" {
  description = "EKS cluster name"
  value       = module.eks.cluster_name
}

output "eks_cluster_security_group_id" {
  description = "EKS cluster security group ID"
  value       = module.eks.security_group_id
}

output "rds_endpoint" {
  description = "RDS instance endpoint"
  value       = module.rds.endpoint
}

output "rds_instance_id" {
  description = "RDS instance ID"
  value       = module.rds.instance_id
}

output "msk_bootstrap_brokers" {
  description = "MSK bootstrap brokers"
  value       = module.msk.bootstrap_brokers
}

output "msk_cluster_arn" {
  description = "MSK cluster ARN"
  value       = module.msk.cluster_arn
}

output "redis_endpoint" {
  description = "Redis/ElastiCache endpoint"
  value       = module.redis.endpoint
}

output "redis_cluster_id" {
  description = "Redis cluster ID"
  value       = module.redis.cluster_id
}

output "vpc_id" {
  description = "VPC ID"
  value       = module.network.vpc_id
}

output "subnet_ids" {
  description = "VPC subnet IDs"
  value       = module.network.subnet_ids
}

output "eks_config_command" {
  description = "kubectl config command for EKS"
  value       = "aws eks update-kubeconfig --name ${module.eks.cluster_name} --region ${var.aws_region}"
}
```

### Module Structure

```
terraform/
├── main.tf                 # Root module configuration
├── providers.tf            # AWS provider and backend
├── variables.tf            # Input variables
├── outputs.tf             # Output values
├── network/               # VPC, subnets, security groups
│   ├── main.tf
│   ├── variables.tf
│   └── outputs.tf
├── eks/                   # Amazon EKS cluster
│   ├── main.tf
│   ├── variables.tf
│   └── outputs.tf
├── rds/                   # Amazon RDS PostgreSQL
│   ├── main.tf
│   ├── variables.tf
│   └── outputs.tf
├── msk/                   # Amazon MSK (Kafka)
│   ├── main.tf
│   ├── variables.tf
│   └── outputs.tf
└── redis/                 # Amazon ElastiCache Redis
    ├── main.tf
    ├── variables.tf
    └── outputs.tf
```

### EKS Module (Amazon Elastic Kubernetes Service)

**Purpose:** Container orchestration for sample applications and documentation site.

**Configuration:**

```hcl
# terraform/eks/main.tf

module "eks" {
  source = "./eks"

  cluster_name    = var.eks_cluster_name
  cluster_version = "1.28"
  vpc_id         = module.network.vpc_id
  subnet_ids     = module.network.subnet_ids

  node_count      = var.eks_node_count
  node_instance_type = var.eks_node_instance_type

  cluster_endpoint_private_access = true
  cluster_endpoint_public_access  = true

  tags = {
    Project     = "single-page-express"
    Environment = var.environment
  }
}

# Local module: terraform/eks/main.tf
resource "aws_eks_cluster" "this" {
  name     = var.cluster_name
  version  = var.cluster_version
  role_arn = aws_iam_role.eks_cluster.arn

  vpc_config {
    subnet_ids              = var.subnet_ids
    security_group_ids      = [aws_security_group.eks_cluster.id]
    endpoint_private_access = var.cluster_endpoint_private_access
    endpoint_public_access  = var.cluster_endpoint_public_access
  }

  enabled_cluster_log_types = ["api", "audit", "authenticator", "controllerManager", "scheduler"]
}

resource "aws_eks_node_group" "this" {
  cluster_name    = aws_eks_cluster.this.name
  node_group_name = "${var.cluster_name}-node-group"
  node_role_arn   = aws_iam_role.eks_nodes.arn
  subnet_ids      = var.subnet_ids

  scaling_config {
    desired_size = var.node_count
    max_size     = var.node_count + 2
    min_size     = var.node_count - 1
  }

  instance_types = [var.node_instance_type]

  depends_on = [
    aws_iam_role_policy_attachment.eks_worker_node_policy
    aws_iam_role_policy_attachment.eks_cni_policy
  ]
}
```

**Outputs:**
- `cluster_endpoint`: EKS API endpoint
- `cluster_name`: EKS cluster name
- `security_group_id`: Cluster security group ID
- `node_role_arn`: IAM role ARN for worker nodes

**Inputs:**
- `cluster_name`: "spe-eks-cluster"
- `cluster_version`: "1.28"
- `node_count`: 3
- `node_instance_type`: "t3.medium"
- `vpc_id`: From network module
- `subnet_ids`: From network module

---

### RDS Module (Amazon Relational Database Service)

**Purpose:** PostgreSQL database for sample application data persistence (demo purposes).

**Configuration:**

```hcl
# terraform/rds/main.tf

module "rds" {
  source = "./rds"

  identifier       = "spe-${var.environment}-postgres"
  engine           = "postgres"
  engine_version   = "15.4"
  instance_class   = var.rds_instance_class
  allocated_storage = 20

  db_name  = "speapp"
  username = var.db_username
  password = var.db_password

  vpc_id            = module.network.vpc_id
  subnet_ids         = module.network.database_subnet_ids
  security_group_ids = [aws_security_group.rds.id]

  multi_az               = true
  publicly_accessible    = false
  storage_encrypted     = true
  backup_retention_period = 7

  tags = {
    Project     = "single-page-express"
    Environment = var.environment
  }
}

# Local module: terraform/rds/main.tf
resource "aws_db_instance" "this" {
  identifier             = var.identifier
  engine                = var.engine
  engine_version        = var.engine_version
  instance_class        = var.instance_class
  allocated_storage     = var.allocated_storage
  storage_type         = "gp2"
  storage_encrypted     = var.storage_encrypted
  db_name               = var.db_name
  username              = var.username
  password              = var.password

  db_subnet_group_name   = aws_db_subnet_group.this.name
  vpc_security_group_ids = var.security_group_ids
  multi_az              = var.multi_az
  publicly_accessible    = var.publicly_accessible

  backup_retention_period = var.backup_retention_period
  skip_final_snapshot    = false

  tags = var.tags
}
```

**Outputs:**
- `endpoint`: RDS instance endpoint
- `instance_id`: DB instance identifier
- `db_name`: Database name
- `db_subnet_group_name`: DB subnet group name

**Inputs:**
- `identifier`: "spe-staging-postgres" / "spe-production-postgres"
- `engine`: "postgres"
- `engine_version`: "15.4"
- `instance_class`: "db.t3.micro"
- `allocated_storage`: 20 GB
- `db_name`: "speapp"
- `username`: "speuser"
- `password`: From AWS Secrets Manager
- `vpc_id`: From network module
- `subnet_ids`: Database subnets from network module

---

### MSK Module (Amazon Managed Streaming for Kafka)

**Purpose:** Event streaming for sample application event sourcing (demo purposes).

**Configuration:**

```hcl
# terraform/msk/main.tf

module "msk" {
  source = "./msk"

  cluster_name           = "spe-${var.environment}-kafka"
  kafka_version          = "3.5.1"
  broker_node_count     = var.msk_broker_count
  broker_instance_type = "kafka.m5.large"

  vpc_id            = module.network.vpc_id
  subnet_ids         = module.network.subnet_ids
  security_group_ids = [aws_security_group.msk.id]

  encryption_info {
    encryption_at_rest_kms_key_arn = aws_kms_key.msk.arn
  }

  client_authentication {
    sasl {
      scram = true
    }
  }

  tags = {
    Project     = "single-page-express"
    Environment = var.environment
  }
}

# Local module: terraform/msk/main.tf
resource "aws_msk_cluster" "this" {
  cluster_name           = var.cluster_name
  kafka_version          = var.kafka_version
  number_of_broker_nodes = var.broker_node_count
  broker_node_group_info {
    instance_type   = var.broker_instance_type
    storage_info {
      ebs_storage_info {
        volume_size = 100
      }
    }
    client_subnets = var.subnet_ids
  }

  vpc_config {
    subnet_ids        = var.subnet_ids
    security_group_ids = var.security_group_ids
  }

  encryption_info = var.encryption_info
  client_authentication = var.client_authentication

  logging_info {
    broker_logs {
      cloudwatch_logs {
        enabled   = true
        log_groups = [aws_cloudwatch_log_group.msk.arn]
      }
    }
  }

  tags = var.tags
}
```

**Outputs:**
- `bootstrap_brokers`: Kafka bootstrap brokers string
- `cluster_arn`: MSK cluster ARN
- `zookeeper_connect_string`: Zookeeper connection string

**Inputs:**
- `cluster_name`: "spe-staging-kafka" / "spe-production-kafka"
- `kafka_version`: "3.5.1"
- `broker_node_count`: 2
- `broker_instance_type`: "kafka.m5.large"
- `vpc_id`: From network module
- `subnet_ids`: Private subnets from network module
- `encryption_info`: KMS key ARN for at-rest encryption

---

### Redis Module (Amazon ElastiCache)

**Purpose:** Caching layer for sample application performance (demo purposes).

**Configuration:**

```hcl
# terraform/redis/main.tf

module "redis" {
  source = "./redis"

  cluster_id           = "spe-${var.environment}-redis"
  node_type            = var.redis_node_type
  num_cache_nodes      = 2
  engine               = "redis"
  engine_version        = "7.0"
  parameter_group_name  = "default.redis7"

  vpc_id            = module.network.vpc_id
  subnet_ids         = module.network.cache_subnet_ids
  security_group_ids = [aws_security_group.redis.id]

  port                = 6379
  automatic_failover   = true
  multi_az_enabled    = true
  snapshot_retention_limit = 5
  encryption_at_rest   = true

  tags = {
    Project     = "single-page-express"
    Environment = var.environment
  }
}

# Local module: terraform/redis/main.tf
resource "aws_elasticache_replication_group" "this" {
  replication_group_id = var.cluster_id
  description         = "Redis cluster for single-page-express sample apps"

  node_type            = var.node_type
  num_cache_nodes      = var.num_cache_nodes
  engine               = var.engine
  engine_version        = var.engine_version
  parameter_group_name  = var.parameter_group_name
  port                 = var.port

  subnet_group_name    = aws_elasticache_subnet_group.this.name
  security_group_ids   = var.security_group_ids
  automatic_failover   = var.automatic_failover
  multi_az_enabled    = var.multi_az_enabled
  at_rest_encryption_enabled = var.encryption_at_rest

  snapshot_retention_limit = var.snapshot_retention_limit

  tags = var.tags
}
```

**Outputs:**
- `endpoint`: Redis cluster endpoint
- `cluster_id`: Redis cluster ID
- `port`: Redis port (6379)

**Inputs:**
- `cluster_id`: "spe-staging-redis" / "spe-production-redis"
- `node_type`: "cache.t3.micro"
- `num_cache_nodes`: 2
- `engine`: "redis"
- `engine_version`: "7.0"
- `vpc_id`: From network module
- `subnet_ids`: Cache subnets from network module
- `port`: 6379
- `automatic_failover`: true
- `multi_az_enabled`: true

---

## Helm Charts

### Overview

Helm charts for deploying sample applications and documentation site to EKS cluster.

### Chart Structure

```
helm/
├── single-page-express-sample/
│   ├── Chart.yaml
│   ├── values.yaml
│   ├── values-staging.yaml
│   └── values-production.yaml
│   └── templates/
│       ├── deployment.yaml
│       ├── service.yaml
│       ├── ingress.yaml
│       ├── configmap.yaml
│       └── hpa.yaml
└── single-page-express-docs/
    ├── Chart.yaml
    ├── values.yaml
    ├── values-staging.yaml
    └── values-production.yaml
    └── templates/
        ├── deployment.yaml
        ├── service.yaml
        ├── ingress.yaml
        └── configmap.yaml
```

### single-page-express-sample Chart

**Chart.yaml:**
```yaml
apiVersion: v2
name: single-page-express-sample
description: Sample applications for single-page-express library
type: application
version: 1.0.0
appVersion: "2.0.5"
keywords:
  - spa
  - express
  - routing
  - single-page-app
maintainers:
  - name: Roosevelt Framework Team
    email: rooseveltframework@gmail.com
```

**values.yaml (Common):**
```yaml
replicaCount: 2

image:
  repository: ghcr.io/rooseveltframework/single-page-express-sample
  pullPolicy: IfNotPresent
  tag: "2.0.5"

service:
  type: ClusterIP
  port: 3000

ingress:
  enabled: false
  className: nginx
  host: sample.spe.local
  tls: false

resources:
  limits:
    cpu: 500m
    memory: 256Mi
  requests:
    cpu: 250m
    memory: 128Mi

autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 5
  targetCPUUtilizationPercentage: 70

nodeSelector: {}
tolerations: []
affinity: {}

env: {}
envFrom: []

configMap:
  enabled: true
  data: {}

secrets: []

healthcheck:
  liveness:
    enabled: true
    path: /
    initialDelaySeconds: 10
    periodSeconds: 10
  readiness:
    enabled: true
    path: /
    initialDelaySeconds: 5
    periodSeconds: 5
```

**values-staging.yaml:**
```yaml
replicaCount: 1

image:
  tag: "2.0.5-preview"

resources:
  limits:
    cpu: 250m
    memory: 128Mi
  requests:
    cpu: 100m
    memory: 64Mi

ingress:
  enabled: true
  host: spe-sample-staging.rooseveltframework.org
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-staging

autoscaling:
  enabled: false
```

**values-production.yaml:**
```yaml
replicaCount: 3

image:
  tag: "2.0.5"

resources:
  limits:
    cpu: 1000m
    memory: 512Mi
  requests:
    cpu: 500m
    memory: 256Mi

ingress:
  enabled: true
  host: spe-sample.rooseveltframework.org
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-production

autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 10
  targetCPUUtilizationPercentage: 70
```

### Deployment Template

```yaml
# helm/single-page-express-sample/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "single-page-express-sample.fullname" . }}
  labels:
    {{- include "single-page-express-sample.labels" . | nindent 4 }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      {{- include "single-page-express-sample.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "single-page-express-sample.selectorLabels" . | nindent 8 }}
        version: {{ .Values.image.tag }}
    spec:
      serviceAccountName: {{ include "single-page-express-sample.serviceAccountName" . }}
      securityContext:
        {}
      containers:
      - name: single-page-express-sample
        securityContext:
          {}
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        ports:
        - name: http
          containerPort: {{ .Values.service.port }}
          protocol: TCP
        livenessProbe:
          httpGet:
            path: {{ .Values.healthcheck.liveness.path }}
            port: http
          initialDelaySeconds: {{ .Values.healthcheck.liveness.initialDelaySeconds }}
          periodSeconds: {{ .Values.healthcheck.liveness.periodSeconds }}
        readinessProbe:
          httpGet:
            path: {{ .Values.healthcheck.readiness.path }}
            port: http
          initialDelaySeconds: {{ .Values.healthcheck.readiness.initialDelaySeconds }}
          periodSeconds: {{ .Values.healthcheck.readiness.periodSeconds }}
        resources:
          {{- toYaml .Values.resources | nindent 10 }}
        env:
          {{- if .Values.env }}
          {{- range $key, $val := .Values.env }}
          - name: {{ $key }}
            value: {{ $val | quote }}
          {{- end }}
          {{- end }}
          - name: NODE_ENV
            value: production
          - name: PORT
            value: "3000"
        {{- with .Values.envFrom }}
        envFrom:
          - {{ toYaml . | nindent 12 }}
        {{- end }}
        volumeMounts:
          - name: cache
            mountPath: /app/cache
      volumes:
        - name: cache
          emptyDir: {}
      {{- with .Values.nodeSelector }}
      nodeSelector:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.affinity }}
      affinity:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.tolerations }}
      tolerations:
        {{- toYaml . | nindent 8 }}
      {{- end }}
```

---

## Istio mTLS

### Overview

Istio service mesh configuration for mutual TLS encryption and service-to-service authentication within the EKS cluster.

### PeerAuthentication STRICT Mode

**Global Policy (All Services):**

```yaml
# istio/peer-authentication-global.yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: default
spec:
  mtls:
    mode: STRICT
```

**Scope:** Applied to all pods in `default` namespace.

**Behavior:**
- STRICT: mTLS is required for all service-to-service communication
- Requests without mTLS are rejected with 403 Forbidden
- Workload-to-workload encryption using Istio certificates

---

### PERMISSIVE Mode (Migration/Development)

**Per-Service Policy for Specific Services:**

```yaml
# istio/peer-authentication-permissive.yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: single-page-express-sample
  namespace: default
spec:
  selector:
    matchLabels:
      app: single-page-express-sample
  mtls:
    mode: PERMISSIVE
```

**Scope:** Applied only to pods matching selector `app: single-page-express-sample`.

**Behavior:**
- PERMISSIVE: Services accept both mTLS and plaintext traffic
- Used during migration phase when some services don't have Istio sidecars
- Gradual migration path from no mTLS → PERMISSIVE → STRICT

---

### Destination Rules for mTLS

```yaml
# istio/destination-rule-mtls.yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: single-page-express-sample
  namespace: default
spec:
  host: single-page-express-sample
  trafficPolicy:
    tls:
      mode: ISTIO_MUTUAL
  subsets:
  - name: v1
    labels:
      version: v1
```

**Configuration:**
- `tls.mode: ISTIO_MUTUAL`: Enforce mTLS for traffic to this service
- Traffic from non-Istio clients will be rejected
- Only mTLS traffic from Istio sidecar proxies allowed

---

### Service Entry Configuration

```yaml
# istio/service-entry-sample.yaml
apiVersion: networking.istio.io/v1beta1
kind: ServiceEntry
metadata:
  name: single-page-express-sample-external
  namespace: istio-system
spec:
  hosts:
  - spe-sample.rooseveltframework.org
  location: MESH_EXTERNAL
  ports:
  - number: 443
    name: https
    protocol: HTTPS
  resolution: DNS
```

**Purpose:** Define external service accessible from mesh with mTLS.

---

### Authorization Policy

```yaml
# istio/authorization-policy.yaml
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: single-page-express-sample-viewer
  namespace: default
spec:
  selector:
    matchLabels:
      app: single-page-express-sample
  action: ALLOW
  rules:
  - from:
    - source:
        principals:
        - cluster.local/ns/default/sa/sample-app-reader
    to:
    - operation:
        methods: ["GET", "POST"]
```

**Purpose:** Only allow specific service principals (with valid mTLS identity) to access the sample application.

---

## Argo CD GitOps

### Overview

Argo CD for continuous deployment of Helm charts from Git repository to EKS cluster.

### Application Configuration

**Staging Application:**

```yaml
# argocd/apps/staging-sample-app.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: staging-sample-app
  namespace: argocd
  finalizers:
    - resources-finalizer.argocd.argoproj.io
spec:
  project: single-page-express

  source:
    repoURL: https://github.com/rooseveltframework/single-page-express.git
    targetRevision: develop
    path: helm/single-page-express-sample
    helm:
      valueFiles:
        - values.yaml
        - values-staging.yaml
      releaseName: sample-app-staging

  destination:
    server: https://kubernetes.default.svc
    namespace: staging

  syncPolicy:
    automated:
      prune: true
      selfHeal: true
      allowEmpty: false
    syncOptions:
      - CreateNamespace=true
      - PrunePropagationPolicy=foreground
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m
```

**Production Application:**

```yaml
# argocd/apps/production-sample-app.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: production-sample-app
  namespace: argocd
  finalizers:
    - resources-finalizer.argocd.argoproj.io
  annotations:
    argocd.argoproj.io/sync-wave: "true"
spec:
  project: single-page-express

  source:
    repoURL: https://github.com/rooseveltframework/single-page-express.git
    targetRevision: main
    path: helm/single-page-express-sample
    helm:
      valueFiles:
        - values.yaml
        - values-production.yaml
      releaseName: sample-app-production

  destination:
    server: https://kubernetes.default.svc
    namespace: production

  syncPolicy:
    automated:
      prune: true
      selfHeal: true
      allowEmpty: false
      # Production requires manual sync (no automated sync)
    syncOptions:
      - CreateNamespace=true
      - PrunePropagationPolicy=foreground
      - ServerSideApply=true
    retry:
      limit: 10
      backoff:
        duration: 10s
        factor: 2
        maxDuration: 10m

  # Diff ignore for configmap/secrets (manage separately)
  ignoreDifferences:
    - group: apps
      kind: Deployment
      jsonPointers:
        - /spec/template/spec/containers/0/resources
```

---

### Promotion to Production Workflow

**Manual Approval Process:**

1. **Staging Deployment (Automated)**
   - Argo CD auto-syncs `develop` branch to staging namespace
   - Staging URL: `https://spe-sample-staging.rooseveltframework.org`

2. **QA Validation**
   - Manual testing on staging environment
   - Run integration tests against staging
   - Validation agent checks

3. **Production Promotion Request**
   - Create Pull Request from `develop` to `main`
   - Argo CD Application shows sync status: `OutOfSync` for production

4. **Production Sync (Manual)**
   - Navigate to Argo CD UI
   - Click "Sync" button on production-sample-app
   - Review diff: show changes between current and desired state
   - Click "Synchronize" to deploy to production

5. **Post-Deployment Verification**
   - Production URL: `https://spe-sample.rooseveltframework.org`
   - Run smoke tests on production
   - Monitor Prometheus/Grafana metrics

**Workflow Visualization:**

```
[develop branch] → Argo CD (auto) → [staging namespace]
                                          ↓
                                   [Manual testing & validation]
                                          ↓
[main branch] ←── Merge ←─ Argo CD (manual sync) ←──┘
                      ↓
                  [production namespace]
```

---

### Rollback Strategy

**Argo CD Rollback:**

```bash
# View sync history and revisions
argocd app history production-sample-app

# Rollback to previous revision
argocd app rollback production-sample-app --revision <previous-revision-id>

# Verify rollback status
argocd app get production-sample-app
```

**Manual Rollback via Helm:**

```bash
# List releases
helm list -n production

# Rollback to previous release
helm rollback sample-app-production -n production

# Verify deployment
kubectl get pods -n production -l app=single-page-express-sample
```

---

## Alertmanager

### Overview

Prometheus Alertmanager configuration for monitoring EKS cluster, sample applications, and CI/CD pipeline failures.

### Alert Rules

```yaml
# alertmanager/alerts-sample-app.yaml
groups:
- name: single-page-express-sample
  rules:
  # Application Health
  - alert: SampleAppDown
    expr: up{job="single-page-express-sample"} == 0
    for: 5m
    labels:
      severity: critical
      component: application
      app: single-page-express-sample
    annotations:
      summary: "Sample application is down"
      description: "The single-page-express-sample application has been down for more than 5 minutes."
      runbook_url: "https://docs.rooseveltframework.org/runbooks/sample-app-down"

  - alert: SampleAppHighErrorRate
    expr: |
      (
        sum(rate(http_requests_total{job="single-page-express-sample", status=~"5.."}[5m])) by (job)
        /
        sum(rate(http_requests_total{job="single-page-express-sample"}[5m])) by (job)
      ) > 0.05
    for: 10m
    labels:
      severity: warning
      component: application
      app: single-page-express-sample
    annotations:
      summary: "Sample application high error rate"
      description: "Sample application error rate is {{ $value | humanizePercentage }} (threshold: 5%)"

  # Performance
  - alert: SampleAppHighLatency
    expr: |
      histogram_quantile(0.99, rate(http_request_duration_seconds_bucket{job="single-page-express-sample"}[5m])) > 1
    for: 10m
    labels:
      severity: warning
      component: application
      app: single-page-express-sample
    annotations:
      summary: "Sample application high latency"
      description: "99th percentile latency is {{ $value }}s (threshold: 1s)"

  - alert: SampleAppHighMemoryUsage
    expr: |
      container_memory_usage_bytes{job="single-page-express-sample"}
      /
      container_spec_memory_limit_bytes{job="single-page-express-sample"}
      > 0.9
    for: 10m
    labels:
      severity: warning
      component: application
      app: single-page-express-sample
    annotations:
      summary: "Sample application high memory usage"
      description: "Memory usage is {{ $value | humanizePercentage }} (threshold: 90%)"

  # CI/CD Pipeline
  - alert: CIPipelineFailure
    expr: github_actions_workflow_run_status{workflow="CI",status="failure"} == 1
    for: 0m
    labels:
      severity: critical
      component: cicd
      workflow: CI
    annotations:
      summary: "CI pipeline failed"
      description: "CI workflow run {{ $labels.run_number }} failed"

  - alert: ValidationAgentCriticalFailure
    expr: |
      github_actions_workflow_run_status{workflow="ValidationAgent",status="failure"} == 1
    for: 0m
    labels:
      severity: critical
      component: cicd
      workflow: ValidationAgent
    annotations:
      summary: "Validation agent found CRITICAL failures"
      description: "Validation agent workflow run {{ $labels.run_number }} failed with CRITICAL violations"

  # Infrastructure
  - alert: EKSNodeNotReady
    expr: |
      kube_node_status_condition{condition="Ready",status!="true"} == 1
    for: 10m
    labels:
      severity: critical
      component: infrastructure
      platform: eks
    annotations:
      summary: "EKS node not ready"
      description: "EKS node {{ $labels.node }} has been not ready for 10 minutes"

  - alert: EKSNodeHighCPUUsage
    expr: |
      sum(rate(container_cpu_usage_seconds_total{container=""}[5m])) by (instance)
      /
      sum(kube_node_status_condition{condition="Ready",status="true"}) by (instance)
      > 0.8
    for: 15m
    labels:
      severity: warning
      component: infrastructure
      platform: eks
    annotations:
      summary: "EKS node high CPU usage"
      description: "EKS node {{ $labels.instance }} CPU usage is {{ $value | humanizePercentage }} (threshold: 80%)"

  # Database (RDS)
  - alert: RDSHighCPUUsage
    expr: aws_rds_cpuutilization_average > 80
    for: 15m
    labels:
      severity: warning
      component: database
      platform: rds
    annotations:
      summary: "RDS instance high CPU usage"
      description: "RDS instance {{ $labels.dbinstanceidentifier }} CPU usage is {{ $value }}% (threshold: 80%)"

  - alert: RDSLowFreeStorage
    expr: |
      (aws_rds_free_storage_space_average / aws_rds_total_storage_space) < 0.2
    for: 10m
    labels:
      severity: warning
      component: database
      platform: rds
    annotations:
      summary: "RDS instance low free storage"
      description: "RDS instance {{ $labels.dbinstanceidentifier }} free storage is {{ $value | humanizePercentage }} (threshold: 20%)"

  # MSK (Kafka)
  - alert: MSKBrokerHighConsumerLag
    expr: |
      kafka_consumergroup_lag_sum{job="kafka"} > 10000
    for: 15m
    labels:
      severity: warning
      component: messaging
      platform: msk
    annotations:
      summary: "MSK broker high consumer lag"
      description: "Consumer group {{ $labels.consumergroup }} has lag of {{ $value }} messages (threshold: 10000)"

  - alert: MSKBrokerUnderReplicatedPartitions
    expr: |
      kafka_under_replicated_partitions{job="kafka"} > 0
    for: 10m
    labels:
      severity: warning
      component: messaging
      platform: msk
    annotations:
      summary: "MSK broker under-replicated partitions"
      description: "{{ $value }} partitions are under-replicated on MSK broker {{ $labels.instance }}"

  # Redis
  - alert: RedisHighMemoryUsage
    expr: |
      aws_elasticache_memory_usage_percentage_average > 80
    for: 15m
    labels:
      severity: warning
      component: cache
      platform: redis
    annotations:
      summary: "Redis cluster high memory usage"
      description: "Redis cluster {{ $labels.cacheclusterid }} memory usage is {{ $value }}% (threshold: 80%)"

  - alert: RedisHighEvictionRate
    expr: |
      rate(aws_elasticache_evictions_total[5m]) > 100
    for: 10m
    labels:
      severity: warning
      component: cache
      platform: redis
    annotations:
      summary: "Redis cluster high eviction rate"
      description: "Redis cluster {{ $labels.cacheclusterid }} eviction rate is {{ $value }}/s (threshold: 100/s)"
```

---

### Routing Configuration

**Default Receiver (Routes to all alerts):**

```yaml
# alertmanager/configmap-routing.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: alertmanager
  namespace: monitoring
data:
  alertmanager.yaml: |
    global:
      resolve_timeout: 5m
      slack_api_url: "{{ .Values.slack.webhook_url }}"

    route:
      receiver: 'default-receiver'
      group_wait: 10s
      group_interval: 10s
      repeat_interval: 12h
      routes:

      # Critical alerts - immediate notification
      - match:
          severity: critical
        receiver: 'slack-critical'
        group_wait: 10s
        group_interval: 5m
        repeat_interval: 5m

      # Warning alerts - aggregated
      - match:
          severity: warning
        receiver: 'slack-warning'
        group_wait: 30s
        group_interval: 10m
        repeat_interval: 1h

      # CI/CD failures - immediate notification
      - match_re:
          - component: cicd
        receiver: 'slack-cicd'
        group_wait: 10s
        group_interval: 5m
        repeat_interval: 10m

      # Infrastructure alerts - dedicated channel
      - match:
          component: infrastructure
        receiver: 'slack-infra'
        group_wait: 30s
        group_interval: 15m
        repeat_interval: 2h

    receivers:
    - name: 'default-receiver'
      email_configs:
      - to: 'oncall@rooseveltframework.org'
        send_resolved: true

    - name: 'slack-critical'
      slack_configs:
      - api_url: "{{ .Values.slack.webhook_url }}"
        channel: '#alerts-critical'
        send_resolved: true
        title: '🚨 {{ .GroupLabels.alertname }}'
        text: |
          {{ range .Alerts }}
          **Alert:** {{ .Labels.alertname }}
          **Severity:** {{ .Labels.severity }}
          **Component:** {{ .Labels.component }}
          **Description:** {{ .Annotations.description }}
          **Runbook:** {{ .Annotations.runbook_url }}
          {{ end }}

    - name: 'slack-warning'
      slack_configs:
      - api_url: "{{ .Values.slack.webhook_url }}"
        channel: '#alerts-warning'
        send_resolved: true
        title: '⚠️ {{ .GroupLabels.alertname }}'
        text: |
          {{ range .Alerts }}
          **Alert:** {{ .Labels.alertname }}
          **Component:** {{ .Labels.component }}
          **Description:** {{ .Annotations.description }}
          {{ end }}

    - name: 'slack-cicd'
      slack_configs:
      - api_url: "{{ .Values.slack.webhook_url }}"
        channel: '#cicd-alerts'
        send_resolved: true
        title: '🔧 CI/CD Alert: {{ .GroupLabels.alertname }}'
        text: |
          {{ range .Alerts }}
          **Workflow:** {{ .Labels.workflow }}
          **Status:** {{ .Status }}
          **Run:** {{ .Labels.run_number }}
          {{ end }}

    - name: 'slack-infra'
      slack_configs:
      - api_url: "{{ .Values.slack.webhook_url }}"
        channel: '#infra-alerts'
        send_resolved: true
        title: '🏗️ Infrastructure Alert: {{ .GroupLabels.alertname }}'
        text: |
          {{ range .Alerts }}
          **Platform:** {{ .Labels.platform }}
          **Component:** {{ .Labels.component }}
          **Description:** {{ .Annotations.description }}
          {{ end }}

    inhibit_rules:
    # Inhibit high error rate if app is down
    - source_match:
        alertname: SampleAppHighErrorRate
      target_match:
        alertname: SampleAppDown
      equal: ['app']
```

---

### Inhibition Rules

**Suppress High Error Rate When App is Down:**

```yaml
inhibit_rules:
- source_match:
    alertname: SampleAppHighErrorRate
  target_match:
    alertname: SampleAppDown
  equal: ['app']
```

**Rationale:** If the application is completely down, the high error rate alert is redundant and adds noise.

**Suppress EKS CPU Alerts During Node Maintenance:**

```yaml
inhibit_rules:
- source_match:
    alertname: EKSNodeHighCPUUsage
  target_match_re:
    - node_condition_maintenance: "true"
      - node_condition_unschedulable: "true"
  equal: ['instance']
```

**Rationale:** During node maintenance or unschedulable state, high CPU usage is expected and not actionable.

**Suppress Redis Eviction During Scale-Down:**

```yaml
inhibit_rules:
- source_match:
    alertname: RedisHighEvictionRate
  target_match:
    event: horizontal_pod_autoscaler_scaled_down
  equal: ['cacheclusterid']
```

**Rationale:** During HPA scale-down events, increased cache eviction is expected as cache shrinks.

---

## Summary

### CI/CD Pipeline Stages

| Stage | Purpose | Trigger | Key Checks | Artifacts |
|-------|---------|---------|-------------|-----------|
| Stage 1: Build & Lint | Build bundles, static analysis | Bundle size ≤ 100KB, no lint errors, no HIGH/CRITICAL vulnerabilities | dist/ bundles, lint reports |
| Stage 2: Test & Validation | E2E tests, pytest, validation agent | Playwright pass, pytest ≥ 80% coverage, validation score ≥ 70, no CRITICAL failures | Test results, coverage reports |
| Stage 3: Staging Deploy | Deploy to npm preview | Preview package published, integration tests pass | npm preview URL, integration test results |
| Stage 4: Production Gate | Final approval before npm publish | 2 approvals, no CRITICAL failures, score ≥ 70, all tests pass | npm production URL, GitHub release |

### Terraform Infrastructure

| Resource | Purpose | Key Configuration |
|----------|---------|-------------------|
| EKS | Container orchestration | 3 nodes, t3.medium, mTLS STRICT |
| RDS | PostgreSQL database | db.t3.micro, multi-az, encrypted |
| MSK | Kafka event streaming | 2 brokers, kafka.m5.large, SCRAM auth |
| Redis | Caching layer | 2 nodes, cache.t3.micro, cluster mode |

### Helm Charts

| Chart | Environment | Replicas | CPU Limit | Memory Limit | Ingress |
|-------|-------------|-----------|-----------|--------------|---------|
| single-page-express-sample | Staging | 1 | 250m | 128Mi | Enabled (staging domain) |
| single-page-express-sample | Production | 3 | 1000m | 512Mi | Enabled (production domain) |

### Istio mTLS

| Service | Mode | Scope | Behavior |
|---------|------|--------|----------|
| All services | STRICT | Namespace default | Require mTLS for all traffic |
| Sample app | PERMISSIVE | Selector: app=single-page-express-sample | Accept mTLS and plaintext (migration) |

### Argo CD Applications

| App | Source Branch | Namespace | Sync Policy | URL |
|-----|--------------|-----------|-------------|-----|
| staging-sample-app | develop | staging | Automated | https://spe-sample-staging.rooseveltframework.org |
| production-sample-app | main | production | Manual (2 approvers) | https://spe-sample.rooseveltframework.org |

### Alertmanager Routing

| Severity | Channel | Group Wait | Repeat Interval | Inhibited By |
|----------|---------|------------|----------------|-------------|
| critical | #alerts-critical | 10s | 5m | None |
| warning | #alerts-warning | 30s | 1h | App down |
| cicd | #cicd-alerts | 10s | 10m | None |
| infrastructure | #infra-alerts | 30s | 2h | Node maintenance |

---

**Document Version:** 1.0
**Last Updated:** 2026-03-08
**Status:** Approved and Ready for Implementation
