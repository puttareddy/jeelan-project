# Release Checklist

## Phase 1: Spec & Design Review
- [ ] All spec files reviewed and approved — [HUMAN]
  - constitution.md reviewed and approved
  - spec.md reviewed and approved
  - plan.md reviewed and approved
  - tasks.md reviewed and approved
  - data-model.md reviewed and approved
  - quickstart.md reviewed and approved
  - ui-spec.md reviewed and approved
  - validation-agent-spec.md reviewed and approved
- [ ] Architecture and data model reviewed — [HUMAN]
  - Frontend architecture preserved (single-page-express.js unchanged)
  - Backend migration to FastAPI architecture approved
  - Data model migration strategy approved (no persistent database)
  - Route definition sharing strategy approved (YAML/JSON format)
  - Template sharing strategy approved (keep Teddy with Python integration)
- [ ] AI spec summary and risk flags reviewed — [AI]/[HUMAN]
  - AI validation agent specification reviewed
  - Risk flags for route compatibility identified
  - Risk flags for template engine compatibility identified
  - Risk flags for performance regressions identified
**Gate:** Specs locked before development.

## Phase 2: Development Complete
- [ ] All tasks per tasks.md completed — [HUMAN]
  - Phase 1: Infrastructure Setup (5 tasks)
  - Phase 2: Database Migration (4 tasks)
  - Phase 3: Backend Migration (15 tasks)
  - Phase 4: Frontend Migration (5 tasks)
  - Phase 5: Styling Migration (2 tasks)
  - Phase 6: Testing (12 tasks)
  - Phase 7: Documentation (7 tasks)
  - Phase 8: Cleanup & Release (6 tasks)
- [ ] Validation agent: score ≥ threshold, no CRITICAL — [AI]
  - Run validation agent on completed code
  - Overall score ≥ 70 required
  - No CRITICAL check failures allowed
  - All CC-001 through A11Y-008 checks passed
- [ ] Code review completed — [HUMAN]
  - Frontend code review (single-page-express.js unchanged)
  - Backend code review (FastAPI server.py implementations)
  - Route definition review (YAML routes.yaml files)
  - Template bundling review (templates.js generation logic)
  - Model migration review (JavaScript → Python async functions)
- [ ] Unit and integration tests passing — [AI]
  - Frontend tests: npm test (Playwright E2E tests)
  - Backend tests: pytest sampleApps/fastapi/tests/
  - Integration tests: Verify isomorphic flow
  - All test files in test/tests.spec.js passing
**Gate:** Code complete, validated, reviewed.

## Phase 3: QA & Acceptance
- [ ] Test plan executed — [HUMAN]
  - All Playwright tests pass against FastAPI backend
  - Manual testing of all sample apps (basic, complex, FastAPI variants)
  - Cross-browser testing (Chrome, Firefox, Safari, Edge)
  - Mobile browser testing
- [ ] Accessibility audit completed — [HUMAN]
  - WCAG 2.1 AA compliance verified
  - ARIA live region announcements tested with screen readers
  - Focus management verified with keyboard navigation
  - Color contrast ratios validated
  - Semantic HTML structure preserved
- [ ] Security scan completed — [HUMAN]
  - npm audit run on frontend dependencies (no HIGH/CRITICAL vulnerabilities)
  - pip-audit run on Python dependencies (no HIGH/CRITICAL vulnerabilities)
  - Static code analysis with tools like bandit or pylint
  - Dependency lockfiles reviewed for security
- [ ] Performance scan completed — [HUMAN]
  - Bundle size measurement (≤ 2x current size ~50KB)
  - Page transition performance benchmarking (≤ 100ms)
  - Route matching efficiency profiling (≤ 10ms)
  - DOM update efficiency measurement (≤ 50ms)
  - Memory usage profiling during extended navigation sessions
- [ ] User Acceptance Testing (UAT) sign-off — [HUMAN]
  - Sample applications tested end-to-end
  - Documentation reviewed by team
  - Development workflow verified by developers
  - Migration guide tested by existing users
  - Sign-off from project lead/architect
**Gate:** Quality and acceptance met.

## Phase 4: Security & Privacy Review
- [ ] Security review completed — [HUMAN]
  - Template XSS prevention verified (SEC-001)
  - Cookie security flags validated (HttpOnly, Secure, SameSite)
  - CORS configuration reviewed
  - Form input validation verified
  - Static file serving security audited
  - Dependency vulnerabilities assessed
- [ ] Privacy review completed — [HUMAN]
  - No personal data stored or transmitted without user consent
  - Cookie handling reviewed for privacy compliance
  - Data retention policy documented (none - ephemeral only)
  - User data handling reviewed in context of regulations
- [ ] Security and privacy sign-off — [HUMAN]
  - Security officer approval obtained
  - Privacy officer approval obtained
  - Document role and approval date
- [ ] No HIGH/CRITICAL security issues open — [AI]/[HUMAN]
  - All security checks in validation agent pass
  - All HIGH/CRITICAL vulnerabilities remediated
  - Security scan reports archived
**Gate:** Security and privacy sign-off.

## Phase 5: Compliance Officer Sign-Offs (Mandatory Before Production)
- [ ] FINTRAC (AML/KYC) — [HUMAN]
  - **Assessment:** Not applicable (no financial transactions, no user accounts, no AML/KYC requirements)
  - **Rationale:** single-page-express is a client-side routing library with no financial data handling
  - **Sign-off:** Compliance officer acknowledges N/A status — [HUMAN]
- [ ] SR 11-7 (Model Risk) — if ML/AI — [HUMAN]
  - **Assessment:** Not applicable (no ML/AI models in library)
  - **Rationale:** Library provides routing and DOM manipulation, no predictive models or AI algorithms
  - **Sign-off:** Compliance officer acknowledges N/A status — [HUMAN]
- [ ] SOX (Financial controls) — if applicable — [HUMAN]
  - **Assessment:** Not applicable (no financial reporting, no public company requirements)
  - **Rationale:** Project is open-source library, not financial system with SOX obligations
  - **Sign-off:** Compliance officer acknowledges N/A status — [HUMAN]
**Gate:** No production deploy without applicable sign-offs.

## Phase 6: Release & Deployment
- [ ] Runbook prepared — [HUMAN]
  - Deployment steps documented
  - Rollback procedures documented
  - Monitoring and alerting setup documented
  - Incident response procedures documented
- [ ] CI/CD and 2-approver gate configured — [HUMAN]
  - GitHub Actions workflow updated for FastAPI backend tests
  - Branch protection rules: require 2 approvals before merge
  - Validation agent integrated into PR checks
  - Automated tests run on every push/PR
- [ ] Change Advisory Board (CAB) approved — [HUMAN]
  - Release proposal reviewed by CAB
  - Rollback plan approved
  - Release window scheduled
  - Stakeholders notified
- [ ] Deployment executed — [HUMAN]
  - npm package published with version bump
  - Frontend bundles (dist/) published to npm
  - Documentation updated on GitHub
  - Release notes published in CHANGELOG.md and GitHub releases
- [ ] Health checks passed — [AI]/[HUMAN]
  - npm package installation verified
  - Sample applications launched successfully
  - Playwright tests pass against published version
  - Documentation links verified
  - Download metrics monitored (optional)
**Gate:** Deployed with approval.

## Phase 7: Post-Release
- [ ] Smoke tests completed — [HUMAN]
  - Basic navigation tested on fresh install
  - All sample apps verified working
  - Documentation examples tested
  - Error handling verified
- [ ] Monitoring active — [HUMAN]
  - npm download metrics monitored
  - GitHub issues tracked for regression reports
  - Community support channels monitored
  - Performance metrics collected (if implemented)
- [ ] Incidents documented — [HUMAN]
  - Any post-release incidents logged
  - Root cause analysis performed for issues
  - Fixes prioritized and scheduled
  - Communication to users completed
- [ ] Checklist archived — [HUMAN]
  - Release checklist saved to project records
  - Sign-offs and approvals archived
  - Lessons learned documented
  - Evidence retained for audit trail
**Gate:** Stable, evidence retained.

## Notes

### Sign-off Responsibilities

**[AI] = Automated Sign-offs**
- Validation agent score and CRITICAL check failures
- Automated test results
- Bundle size measurements
- Security scan results (automated tools)

**[HUMAN] = Manual Sign-offs**
- Spec review and approval
- Code review
- QA/UAT acceptance
- Security and privacy reviews
- Compliance officer sign-offs
- CAB approval
- Deployment execution
- Post-release monitoring and incident handling

### Document Role and Artifact for Compliance

For each HUMAN sign-off, document:
- **Role:** Developer, QA Engineer, Security Officer, Compliance Officer, Project Lead, etc.
- **Artifact:** Specific evidence of review (e.g., PR number, test report URL, meeting minutes, approval email)
- **Date:** Date of sign-off
- **Comments:** Any conditions or notes (e.g., "Approved with minor documentation updates required")

### Thresholds

**Validation Agent Score:**
- 90-100: Excellent — Ready for merge
- 70-89: Good — Merge after minor fixes
- 50-69: Fair — Address HIGH/MEDIUM issues before merge
- 0-49: Poor — Major refactor needed
- FAIL: CRITICAL violations — No merge allowed

**Bundle Size Constraint:**
- Current bundle size: ~50KB
- Maximum allowed: 100KB (2x current)
- Target: Maintain ~50KB (no significant increase)

**Test Coverage:**
- All existing Playwright tests must pass (test/tests.spec.js)
- New pytest tests for FastAPI backend must pass
- Integration tests must verify isomorphic flow

### Non-Applicable Compliance Items

The following compliance frameworks are explicitly **N/A** for this project:

1. **FINTRAC (AML/KYC)**: No financial transactions, no user accounts, no AML/KYC requirements
2. **SR 11-7 (Model Risk)**: No ML/AI models, no predictive algorithms
3. **SOX (Financial Controls)**: Open-source library, not financial system with SOX obligations

Documentation of N/A status is still required for audit trail.

### Critical Path Dependencies

**Phase Dependencies:**
- Phase 1 must complete before Phase 2
- Phase 2 must complete before Phase 3 (backend requires models)
- Phase 3 must complete before Phase 4 (backend must serve static files)
- Phase 4 must complete before Phase 5 (testing requires working frontend+backend)
- Phase 5 must complete before Phase 6 (cleanup only after tests pass)
- Phase 6 must complete before Phase 7 (validation of final state)
- Phase 7 is independent (can proceed in parallel with documentation)

**Task Dependencies (from tasks.md):**
- Tasks marked with [P] can be executed in parallel
- Other tasks must run sequentially within their phase
- Refer to tasks.md for detailed task dependencies

### Rollback Procedures

If post-release issues are detected:

1. **Frontend Bundle Issues:**
   - Unpublish npm package version
   - Revert to previous version on GitHub
   - Communicate to users via GitHub issues and announcements

2. **Backend Integration Issues:**
   - Document workaround in migration guide
   - Provide Express backend samples as fallback
   - Schedule hotfix release

3. **Documentation Issues:**
   - Update documentation with corrections
   - Provide errata in GitHub releases
   - Update migration guide with known issues

### Evidence Retention

Retain the following evidence for audit trail:

- All PR approvals and review comments
- Validation agent reports (JSON and markdown)
- Test results (Playwright and pytest)
- Security scan reports
- Performance benchmarks
- Sign-off documentation with roles and dates
- CAB meeting minutes
- Incident reports (if any)
- Release notes and CHANGELOG.md

Retention period: Minimum 3 years, or per organization policy.
