# Validation Agent Specification

## Overview

The Validation Agent is an autonomous AI agent that enforces the Modernization Constitution on every Pull Request for the single-page-express modernization project. The agent runs comprehensive validation checks across multiple modules, assigns a 0-100 score, and auto-fails any PR containing CRITICAL violations. The agent ensures that all changes preserve the Express-compatible API surface, maintain data integrity, and avoid regressions during the migration from Express.js to FastAPI.

**Key Responsibilities:**
- Load and enforce the Modernization Constitution on every PR
- Run 5 validation modules with 40+ named checks
- Calculate weighted scores (0-100) across all modules
- Auto-fail PRs with CRITICAL violations
- Generate detailed reports with pass/fail status and remediation guidance

## Validation Modules

### Module 1: Constitution Compliance Module

**Scope:** Validates adherence to the Modernization Constitution governing principles.

**Purpose:** Ensure all changes respect the five governing principles: Preserve Business Logic, Data Integrity, Contract Compliance, Incremental Verification, and No Regressions.

**Output:** Module score (0-100), list of passed/failed checks, remediation for failures.

**Checks Validated:**
- Business logic preservation in route handlers
- Data integrity during template migration
- Frontend API surface compatibility
- Test coverage retention
- Regression prevention in existing features

### Module 2: Security Module

**Scope:** Validates security aspects of the FastAPI backend migration.

**Purpose:** Ensure no security vulnerabilities are introduced during the migration, including XSS prevention, cookie handling, and CORS configuration.

**Output:** Module score (0-100), security findings, recommended remediation.

**Checks Validated:**
- XSS prevention in template rendering
- Secure cookie handling (HttpOnly, Secure, SameSite flags)
- CORS policy configuration
- Input validation on form submissions
- SQL injection prevention (if database added later)
- Authentication handling (if added later)
- Static file serving security
- Dependency vulnerability scanning

### Module 3: Data & Privacy Module

**Scope:** Validates data handling, storage, and privacy compliance.

**Purpose:** Ensure no data loss occurs during migration, cookies are handled correctly, and user data remains private and secure.

**Output:** Module score (0-100), data integrity status, privacy compliance notes.

**Checks Validated:**
- No data loss in route parameter handling
- Query string parsing integrity
- Form submission data preservation
- Cookie reading/writing compatibility
- Scroll position data persistence
- URL history state management
- Model data structure preservation
- Template data binding accuracy

### Module 4: Performance Module

**Scope:** Validates performance characteristics and bundle size constraints.

**Purpose:** Ensure the migration does not introduce performance regressions, bundle size bloat, or navigation speed degradation.

**Output:** Module score (0-100), performance metrics, optimization recommendations.

**Checks Validated:**
- Bundle size constraints (no significant increase)
- Page transition speed
- Route matching performance
- DOM update efficiency
- Async data loading performance
- Scroll restoration speed
- Memory usage in navigation
- View transition API utilization

### Module 5: Accessibility Module

**Scope:** Validates WCAG 2.1 AA compliance and accessibility features.

**Purpose:** Ensure all accessibility features are preserved, including ARIA live regions, focus management, and keyboard navigation.

**Output:** Module score (0-100), accessibility audit results, remediation steps.

**Checks Validated:**
- ARIA live region announcements
- Focus management after page updates
- Keyboard navigation support
- Screen reader compatibility
- Color contrast ratios
- Semantic HTML preservation
- Focus visible indicators
- Skip link presence (if applicable)

## Named Checks

### Constitution Compliance Checks

| Check ID | Name | Severity | Pass Criteria | Remediation (if failed) |
|-----------|------|-----------|---------------|---------------------|
| CC-001 | Route Registration Preserved | CRITICAL | All existing route methods (`app.get`, `app.post`, `app.put`, `app.delete`, `app.all`, `app.route`) work identically | Restore any modified route registration logic in `single-page-express.js` |
| CC-002 | Request Object Properties Intact | CRITICAL | All `req` properties present and functional (`req.params`, `req.query`, `req.body`, `req.cookies`, `req.path`, `req.method`, `req.url`, etc.) | Verify `single-page-express.js` lines 141-180 preserve all request properties |
| CC-003 | Response Object Methods Intact | CRITICAL | All `res` methods work identically (`res.render`, `res.redirect`, `res.cookie`, `res.clearCookie`, `res.json`) | Verify `single-page-express.js` lines 183-273 preserve all response methods |
| CC-004 | SPE-Specific Properties Preserved | CRITICAL | All SPE-specific `res` properties work (`res.title`, `res.target`, `res.focus`, `res.beforeRender`, `res.afterRender`, head tag removal flags) | Check lines 486-736 for SPE-specific property handling |
| CC-005 | Pre/Post Render Hooks Work | HIGH | `app.beforeEveryRender`, `app.afterEveryRender`, `res.beforeRender`, `res.afterRender` execute in correct order | Verify hook execution order in `handleRoute` function |
| CC-006 | Route Parameter Parsing | HIGH | Route parameters (`/users/:id`) parse correctly and populate `req.params` | Test path-to-regexp integration lines 297-305, 400-410 |
| CC-007 | Query String Parsing | HIGH | Query strings (`?key=value`) parse correctly and populate `req.query` | Verify query parsing lines 418-423 |
| CC-008 | Wildcard Route Support | HIGH | Wildcard routes (`*all` for Express 5, `*` for Express 4) work correctly | Test wildcard matching lines 332-342 |
| CC-009 | Case Sensitivity Setting | MEDIUM | `app.appVars['case sensitive routing']` setting respected | Verify case handling lines 287-288 |
| CC-010 | Strict Routing Setting | MEDIUM | `app.appVars['strict routing']` setting respected for trailing slashes | Verify slash handling lines 290-291 |

### Security Checks

| Check ID | Name | Severity | Pass Criteria | Remediation (if failed) |
|-----------|------|-----------|---------------|---------------------|
| SEC-001 | Template XSS Prevention | CRITICAL | Template rendering escapes or properly sanitizes user input | Review `app.templatingEngine.render()` calls, ensure input sanitization |
| SEC-002 | Cookie Security Flags | CRITICAL | `res.cookie()` supports HttpOnly, Secure, SameSite flags | Verify cookie implementation lines 196-248 |
| SEC-003 | Cookie Clearing Security | HIGH | `res.clearCookie()` sets expiration to past date | Verify clearCookie implementation lines 225-248 |
| SEC-004 | Signed Cookie Warning | MEDIUM | Development mode warning logged when `signed` flag used | Check warning logic line 221-222 |
| SEC-005 | CORS Configuration | HIGH | FastAPI CORS middleware properly configured | Review FastAPI server CORS settings |
| SEC-006 | Form Input Validation | HIGH | Form submissions validated before processing | Add validation to FastAPI form handlers |
| SEC-007 | Static File Security | MEDIUM | Static files served without directory traversal vulnerabilities | Verify FastAPI `StaticFiles` configuration |
| SEC-008 | Dependency Security | MEDIUM | No known vulnerabilities in dependencies | Run `npm audit` and `pip-audit` |

### Data & Privacy Checks

| Check ID | Name | Severity | Pass Criteria | Remediation (if failed) |
|-----------|------|-----------|---------------|---------------------|
| DP-001 | Route Parameters Preserved | CRITICAL | `req.params` structure unchanged during migration | Compare Express vs FastAPI parameter extraction |
| DP-002 | Query String Integrity | CRITICAL | `req.query` object structure identical to Express | Verify query parsing logic |
| DP-003 | Form Data Parsing | HIGH | Form submissions parse identically to Express body-parser | Test FastAPI python-multipart vs Express body-parser |
| DP-004 | Cookie Reading | HIGH | `req.cookies` populated from `document.cookie` correctly | Verify cookie parsing lines 390-395 |
| DP-005 | Cookie Writing | HIGH | `res.cookie()` writes to `document.cookie` with correct format | Test cookie string construction lines 210-223 |
| DP-006 | Cookie Clearing | HIGH | `res.clearCookie()` removes cookies correctly | Test cookie expiration logic lines 237-247 |
| DP-007 | Scroll Position Memory | MEDIUM | `app.urls` stores scroll positions per URL | Verify scroll storage lines 352-366 |
| DP-008 | History State Management | MEDIUM | Browser history stack correctly maintained | Verify push/popstate handling lines 368-378 |

### Performance Checks

| Check ID | Name | Severity | Pass Criteria | Remediation (if failed) |
|-----------|------|-----------|---------------|---------------------|
| PER-001 | Bundle Size Constraint | HIGH | Total bundle size ≤ 2x current size (current ~50KB) | Measure `dist/single-page-express.*` files |
| PER-002 | Page Transition Speed | MEDIUM | Navigation completes within 100ms of render | Benchmark DOM update timing |
| PER-003 | Route Matching Efficiency | MEDIUM | Route matching completes within 10ms | Profile route matching loop lines 332-342 |
| PER-004 | DOM Update Efficiency | MEDIUM | DOM updates complete within 50ms | Measure `domUpdate` function timing |
| PER-005 | Async Data Loading | MEDIUM | Async model loads do not block UI | Test `getRandomNumber` async pattern |
| PER-006 | Scroll Restoration Speed | LOW | Scroll position restores within 20ms | Measure `scrollPage` function timing |
| PER-007 | Memory Usage | LOW | Memory footprint does not increase >10% | Profile navigation session memory |
| PER-008 | View Transition API | LOW | Uses native API when available, falls back gracefully | Verify view transition logic line 728 |

### Accessibility Checks

| Check ID | Name | Severity | Pass Criteria | Remediation (if failed) |
|-----------|------|-----------|---------------|---------------------|
| A11Y-001 | ARIA Live Region | CRITICAL | Page changes announced to screen readers | Verify ARIA live region lines 675-694 |
| A11Y-002 | Focus Management | CRITICAL | Focus set to appropriate element after render | Verify focus logic lines 696-711 |
| A11Y-003 | Keyboard Navigation | HIGH | All functionality accessible via keyboard | Test tab order through all routes |
| A11Y-004 | Screen Reader Compatible | HIGH | Semantic HTML preserved, proper headings | Verify heading structure and landmarks |
| A11Y-005 | Color Contrast | HIGH | Text contrast ≥ 4.5:1 (WCAG AA) | Audit CSS colors |
| A11Y-006 | Focus Visible | MEDIUM | Interactive elements have visible focus indicators | Verify validElementsForOutline handling line 697 |
| A11Y-007 | Focus Trap Avoidance | MEDIUM | No custom focus traps blocking navigation | Review custom focus logic |
| A11Y-008 | Semantic HTML | MEDIUM | Proper use of semantic elements (`<nav>`, `<article>`, `<section>`) | Verify template semantic structure |

**Total Checks: 40 named checks**

## Scoring Model (0–100)

### Module Weights

Each validation module contributes to the overall score based on its weight and the number of passed/failed checks:

| Module | Weight | Scoring Method |
|--------|--------|---------------|
| Constitution Compliance | 30% | Weighted average of check scores |
| Security | 25% | Weighted average of check scores |
| Data & Privacy | 20% | Weighted average of check scores |
| Performance | 15% | Weighted average of check scores |
| Accessibility | 10% | Weighted average of check scores |

### Per-Check Scoring

Each check contributes points based on severity:

| Severity | Points (if passed) | Points (if failed) |
|----------|---------------------|-------------------|
| CRITICAL | 3 | 0 (also triggers auto-fail) |
| HIGH | 2 | 0 |
| MEDIUM | 1 | 0 |
| LOW | 0.5 | 0 |

### Overall Score Calculation

```
Overall Score = Σ(Module Weight × Module Score)

Module Score = (Sum of passed check points / Total possible points) × 100

Example:
Constitution Module: 8/10 checks passed = (19/24) × 100 = 79.2
Score contribution: 79.2 × 0.30 = 23.76

Repeat for all modules and sum for final score (0-100)
```

### CRITICAL Failure Condition

**Any CRITICAL check failure results in automatic FAIL status.**

When a CRITICAL check fails:
1. PR status is set to `FAIL`
2. No merge allowed regardless of overall score
3. Detailed remediation required before re-submission
4. Comment posted with all CRITICAL failures and remediation steps

CRITICAL checks:
- CC-001: Route Registration Preserved
- CC-002: Request Object Properties Intact
- CC-003: Response Object Methods Intact
- CC-004: SPE-Specific Properties Preserved
- SEC-001: Template XSS Prevention
- SEC-002: Cookie Security Flags
- DP-001: Route Parameters Preserved
- DP-002: Query String Integrity
- A11Y-001: ARIA Live Region
- A11Y-002: Focus Management

## Auto-Fail on CRITICAL

### FAIL Status Trigger

When any CRITICAL check fails, the Validation Agent:

1. **Sets PR Status to FAIL**
   - GitHub PR check fails with "Validation: CRITICAL violations detected"
   - PR cannot be merged until all CRITICAL issues resolved

2. **Posts Detailed Comment**
   ```markdown
   ## ❌ Validation Failed: CRITICAL Violations Detected

   This PR contains CRITICAL violations that must be fixed before merge.

   ### Critical Issues:

   - **[Check ID]**: [Check Name]
     - **Issue:** [Description of violation]
     - **Remediation:** [Specific fix required]
     - **Location:** [File path and line numbers]

   [Repeat for all critical failures]

   ### Next Steps:
   1. Fix all CRITICAL violations listed above
   2. Re-run validation with `/validate-pr` command
   3. Update PR with fixes
   4. Re-request validation review

   ### Overall Score: [Score]/100 (FAILED due to CRITICAL violations)
   ```

3. **Prevents Merge**
   - Branch protection blocks merge on failed status
   - CI/CD pipeline fails before deployment
   - Team notified of blocking issues

4. **Remediation Requirements**
   - Each CRITICAL failure must be addressed
   - Proof of fix required (test output or code diff)
   - Re-validation must pass all CRITICAL checks
   - Optional: Address HIGH/MEDIUM/LOW issues for higher score

### Auto-Fail Examples

**Example 1: Broken Route Registration**
```
Check: CC-001 - Route Registration Preserved
Issue: app.get() method modified, no longer accepts middleware parameter
Remediation: Restore original signature in single-page-express.js line 116
Location: /Users/.../single-page-express.js:116
```

**Example 2: Missing XSS Protection**
```
Check: SEC-001 - Template XSS Prevention
Issue: Template renders user input without escaping
Remediation: Add input sanitization to FastAPI template rendering
Location: /Users/.../sampleApps/fastapi/server.py
```

**Example 3: Broken Focus Management**
```
Check: A11Y-002 - Focus Management
Issue: Focus not set after page render, breaking keyboard navigation
Remediation: Restore focus logic in single-page-express.js line 696-711
Location: /Users/.../single-page-express.js:696-711
```

## Integration

### Trigger

The Validation Agent triggers on:

1. **Pull Request Events**
   - PR opened: Full validation run
   - PR updated: Incremental validation of changed files
   - PR labeled with `validation-request`: Re-run full validation

2. **Commit Events**
   - Push to branch with open PR: Validate new commits
   - Force push: Re-run validation if suspicious changes

3. **Manual Trigger**
   - Comment `/validate` on PR: Re-run full validation
   - Command: `npm run validate-pr`: Local validation before push

### Inputs

**Required Inputs:**
- PR diff (files added, modified, deleted)
- Branch name (main, develop, feature/*)
- Commit history (for context)
- Test results (if available)

**Optional Inputs:**
- Target branch for comparison
- Custom validation rules override
- Previous validation results (for regression detection)

**Configuration Files:**
- `/Users/puttaiaharugunta/codebase/infi/project-auth/specs/main/constitution.md`
- `/Users/puttaiaharugunta/codebase/infi/project-auth/specs/main/validation-rules.json`
- `.github/validation-config.yml` (CI/CD integration)

### Outputs

**GitHub PR Comment:**
```markdown
## ✅ Validation Results: [PASS/FAIL]

**Overall Score:** [X]/100

### Module Scores:

| Module | Score | Status |
|--------|-------|--------|
| Constitution Compliance | [X]/100 | [✅/❌] |
| Security | [X]/100 | [✅/❌] |
| Data & Privacy | [X]/100 | [✅/❌] |
| Performance | [X]/100 | [✅/❌] |
| Accessibility | [X]/100 | [✅/❌] |

### Detailed Results:

#### Constitution Compliance ([X]/10 passed)
[Summary of passed/failed checks]

#### Security ([X]/8 passed)
[Summary of passed/failed checks]

#### Data & Privacy ([X]/8 passed)
[Summary of passed/failed checks]

#### Performance ([X]/8 passed)
[Summary of passed/failed checks]

#### Accessibility ([X]/6 passed)
[Summary of passed/failed checks]

### Remediation Required:
[List of fixes for failed checks]

### Validation Log:
[Timestamped check execution log]
```

**JSON Output (for CI/CD integration):**
```json
{
  "status": "PASS" | "FAIL",
  "overall_score": 87.5,
  "modules": {
    "constitution": {
      "score": 90.0,
      "passed": 9,
      "failed": 1,
      "checks": [
        {"id": "CC-001", "status": "PASS", "points": 3},
        {"id": "CC-002", "status": "FAIL", "points": 0, "severity": "CRITICAL"}
      ]
    }
  },
  "critical_failures": ["CC-002"],
  "remediation": ["Restore request object properties"],
  "timestamp": "2026-03-08T12:00:00Z"
}
```

### Reporting

**PR Status Checks:**
- GitHub Status API for real-time validation status
- Distinct status for: Pending, Running, Passed, Failed

**Dashboard Integration:**
- Link to full validation report in project dashboard
- Historical validation trends across PRs
- Comparison with previous PRs (regression detection)

**Notifications:**
- Slack/Teams integration for team alerts
- Email for CRITICAL failures
- GitHub notifications for @mentions

### CI/CD Pipeline Integration

**GitHub Actions Workflow:**
```yaml
name: Validation Agent

on:
  pull_request:
    types: [opened, synchronize, reopened]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run Validation Agent
        run: |
          npm install
          npm run validate-pr
        env:
          PR_NUMBER: ${{ github.event.pull_request.number }}
          PR_URL: ${{ github.event.pull_request.html_url }}
      - name: Post Results
        if: always()
        run: |
          npm run post-validation-results
```

**Exit Codes:**
- `0`: PASS (all checks passed, score ≥ 70)
- `1`: FAIL (CRITICAL violation or score < 70)
- `2`: ERROR (validation agent failed to run)

## Local Development Workflow

### Run Validation Locally

```bash
# Install validation agent dependencies
npm install

# Run full validation against current branch
npm run validate-pr

# Validate specific modules
npm run validate-constitution
npm run validate-security
npm run validate-data-privacy
npm run validate-performance
npm run validate-accessibility

# Validate specific checks
npm run validate-check --check=CC-001
npm run validate-check --check=SEC-001

# Generate validation report
npm run validate-report --format=markdown
npm run validate-report --format=json
```

### Pre-Commit Hook

```bash
# .git/hooks/pre-commit
npm run validate-constitution && npm run validate-security
exit $?
```

### VS Code Integration

```json
{
  "tasks": {
    "validate-pr": {
      "label": "Run Validation Agent",
      "type": "shell",
      "command": "npm run validate-pr",
      "problemMatcher": {
        "pattern": "\\[ERROR\\] (.*)",
        "file": 1,
        "location": 2
      }
    }
  }
}
```

## Maintenance

### Adding New Checks

1. Define check in appropriate module section
2. Assign unique Check ID (MODULE-###)
3. Specify severity and pass criteria
4. Add remediation guidance
5. Update scoring weights if needed
6. Add test case for new check

### Updating Constitution

When Constitution changes:
1. Update `/Users/puttaiaharugunta/codebase/infi/project-auth/specs/main/constitution.md`
2. Map new constitution principles to validation checks
3. Update scoring model weights if new principles added
4. Document mapping between constitution and checks

### Version History

- **v1.0** (2026-03-08): Initial validation agent specification
  - 5 modules, 40 checks
  - CRITICAL auto-fail mechanism
  - GitHub Actions integration

## Appendix

### Check Reference by File

**single-page-express.js:**
- CC-001, CC-002, CC-003, CC-004 (lines 5-39, 141-273, 486-736)
- PER-002, PER-003, PER-004 (lines 658-730)
- A11Y-001, A11Y-002, A11Y-006 (lines 675-711)

**sampleApps/fastapi/server.py:**
- SEC-001, SEC-005, SEC-006, SEC-007 (new FastAPI server)
- DP-001, DP-002, DP-003 (FastAPI route handlers)

**sampleApps/fastapi/mvc/routes.yaml:**
- CC-006, CC-007, CC-008 (YAML route definitions)

**sampleApps/*/mvc/views/*.html:**
- SEC-001 (template XSS prevention)
- A11Y-004, A11Y-008 (semantic HTML)

**test/tests.spec.js:**
- CC-005 (pre/post-render hooks testing)
- Integration test coverage validation

### Severity Definitions

- **CRITICAL**: Breaking change to frontend API, security vulnerability, data loss, accessibility blocker
- **HIGH**: Significant functionality regression, security concern, performance degradation
- **MEDIUM**: Minor functionality regression, configuration issue, mild performance impact
- **LOW**: Code quality issue, minor optimization opportunity

### Score Thresholds

- **90-100**: Excellent — Ready for merge
- **70-89**: Good — Merge after minor fixes
- **50-69**: Fair — Address HIGH/MEDIUM issues before merge
- **0-49**: Poor — Major refactor needed
- **FAIL**: CRITICAL violations — No merge allowed

---

**Document Version:** 1.0
**Last Updated:** 2026-03-08
**Status:** Approved and Ready for Implementation
