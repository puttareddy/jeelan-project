# Modernization Plan

## Phase 1: Infrastructure Setup
- [ ] **Task-1.1**: Initialize FastAPI project structure in `/Users/puttaiaharugunta/codebase/infi/single-page-express/sampleApps/fastapi/`
  - Create `server.py` entry point file
  - Create `pyproject.toml` for Python dependency management (Poetry)
  - Create `requirements.txt` for pip compatibility
  - Set up project directory structure: `mvc/`, `public/`, `tests/`

- [ ] **Task-1.2**: Set up Python development environment
  - Create Python 3.8+ virtual environment
  - Install FastAPI, Uvicorn, python-multipart, Pydantic
  - Configure Poetry for dependency management
  - Add development dependencies: pytest, pytest-asyncio, httpx, black

- [ ] **Task-1.3**: Configure build pipeline and development workflow
  - Set up Uvicorn ASGI server with hot reload (`--reload` flag)
  - Create npm scripts for FastAPI sample apps in root `package.json`
  - Configure proxy settings for separate frontend/backend dev servers
  - Set up static file serving configuration for bundled frontend JavaScript

- [ ] **Task-1.4**: Configure testing framework for FastAPI backend
  - Set up pytest with asyncio support
  - Create test directory structure: `sampleApps/fastapi/tests/`
  - Configure TestClient for FastAPI endpoint testing
  - Set up pytest-playwright for integration testing

## Phase 2: Database Migration

- [ ] **Task-2.1**: Design target data models (No persistent database required)
  - Document that no database migration is needed (library uses in-memory data)
  - Define model data pattern for async data loading
  - Create Python equivalents for browser-side model functions

- [ ] **Task-2.2**: Create data model migration scripts
  - Translate JavaScript model functions to Python async functions
  - Example: `getRandomNumber.js` → `get_random_number.py`
  - Create model directory structure: `mvc/models/browser/` and `mvc/models/server/`
  - Implement Python model loading pattern for async operations

- [ ] **Task-2.3**: Set up ORM-like model interface (Not applicable)
  - Document that no ORM is required (no persistent data models)
  - Create simple model pattern for Python async functions
  - Define interface for model functions returning data objects

- [ ] **Task-2.4**: Verify data integrity
  - Create tests to verify Python models return identical data to JavaScript models
  - Test async model loading with FastAPI endpoints
  - Verify data structure compatibility between server and client

## Phase 3: Backend Migration

- [ ] **Task-3.1**: Rewrite API endpoints in FastAPI
  - Create FastAPI application instance in `server.py`
  - Implement static file serving from `/public/` directory using `StaticFiles`
  - Implement form data parsing with `python-multipart`
  - Create Express-compatible route registration system

- [ ] **Task-3.2**: Implement route definition sharing strategy
  - **Task-3.2.1**: Design and implement YAML route configuration format
    - Create `mvc/routes.yaml` with route definitions
    - Support HTTP methods: GET, POST, PUT, DELETE, all
    - Support route parameters: `/users/:id`, query strings, wildcards
    - Include template assignment, async model loading, and custom handlers
  - **Task-3.2.2**: Implement JavaScript bridge for existing routes
    - Create Python loader for `mvc/routes.js` files
    - Use subprocess or PyExecJS to execute JavaScript route definitions
    - Maintain backward compatibility with existing Express route files
  - **Task-3.2.3**: Implement route parser for both Python and JavaScript
    - Create Python route registration system from YAML config
    - Support Express 4 and 5 route syntax compatibility
    - Implement route matching with path parameters

- [ ] **Task-3.3**: Implement template rendering in FastAPI
  - **Task-3.3.1**: Integrate Teddy templating engine with Python
    - Install and configure PyExecJS or similar JavaScript engine
    - Create Teddy template loader from `mvc/views/` directory
    - Implement server-side template rendering using Teddy engine
  - **Task-3.3.2**: Set up template bundling for client-side delivery
    - Create Python equivalent of template bundling logic
    - Bundle templates into `public/templates.js` for frontend
    - Ensure template compatibility between server and client rendering

- [ ] **Task-3.4**: Implement middleware and error handling
  - Create FastAPI middleware for Express-like behavior
  - Implement cookie reading/writing support
  - Implement form body parsing for application/x-www-form-urlencoded
  - Create error handlers with proper HTTP status codes
  - Implement JSON API endpoints (e.g., `/api/randomNumber`)

- [ ] **Task-3.5**: Set up JSON API endpoints
  - Create async endpoints for data retrieval
  - Implement CORS support for cross-origin requests
  - Add automatic OpenAPI documentation (FastAPI feature)
  - Test JSON responses match Express backend behavior

- [ ] **Task-3.6**: Migrate business logic from Express sample apps
  - Migrate `sampleApps/express/server.js` → `sampleApps/fastapi/server.py`
  - Migrate `sampleApps/express-complex/server.js` → `sampleApps/fastapi-complex/server.py`
  - Implement all route handlers with identical behavior
  - Preserve template rendering and data loading patterns

## Phase 4: Frontend Migration

- [ ] **Task-4.1**: Verify frontend bundle loads correctly with FastAPI backend
  - Test Webpack build produces same bundled JavaScript
  - Verify `dist/single-page-express.mjs` loads from FastAPI static files
  - Ensure frontend JavaScript works with Python backend
  - Test basic routing functionality

- [ ] **Task-4.2**: Create FastAPI sample app entry points
  - **Task-4.2.1**: Create `sampleApps/fastapi/browser.js`
    - Copy from `sampleApps/express/browser.js` (unchanged)
    - Verify Teddy client-side templating loads correctly
    - Test route registration and rendering
  - **Task-4.2.2**: Create `sampleApps/fastapi-complex/browser.js`
    - Copy from `sampleApps/express-complex/browser.js` (unchanged)
    - Verify all complex features work (head tags, scroll, focus, etc.)

- [ ] **Task-4.3**: Implement routing with FastAPI backend
  - Test shared routes work between Python backend and JavaScript frontend
  - Verify route parameters pass correctly
  - Test query string parsing
  - Test wildcard routes
  - Verify form submission handling

- [ ] **Task-4.4**: Apply styling and maintain CSS
  - Copy `styles.css` from Express samples to FastAPI samples
  - Ensure CSS loads correctly from static files
  - Test responsive styling remains intact
  - Verify no CSS conflicts or missing assets

- [ ] **Task-4.5**: Update build scripts in package.json
  - Add npm script: `"fastapi-sample": "cd sampleApps/fastapi && uvicorn server:app --reload --port 3000"`
  - Add npm script: `"fastapi-complex": "cd sampleApps/fastapi-complex && uvicorn server:app --reload --port 3000"`
  - Update existing scripts to reference new FastAPI samples where appropriate
  - Document new development workflow in README

## Phase 5: Testing & Verification

- [ ] **Task-5.1**: Write unit tests for migrated backend code
  - Create pytest test suite for FastAPI endpoints
  - Test route registration from YAML config
  - Test template rendering with Python Teddy integration
  - Test form data parsing and validation
  - Test async model loading functions
  - Test cookie handling

- [ ] **Task-5.2**: Write integration tests for API endpoints
  - Create tests using FastAPI TestClient
  - Test full request/response cycles
  - Verify data models return correct JSON
  - Test error scenarios (404, 500, etc.)
  - Compare output between Express and FastAPI implementations

- [ ] **Task-5.3**: Verify all existing Playwright tests pass with FastAPI backend
  - Update `test/tests.spec.js` to launch FastAPI server instead of Express
  - Run all existing Playwright tests against FastAPI backend
  - Verify homepage renders correctly
  - Verify second page renders with variable passing
  - Verify form submission handling works
  - Verify async data retrieval works
  - Verify route parameters pass correctly
  - Verify scroll position memory works
  - Verify focus management works
  - Verify accessibility features work (ARIA announcements)
  - Verify view transitions work

- [ ] **Task-5.4**: Create backend-specific tests for FastAPI features
  - Test Python Teddy template rendering
  - Test async model loading
  - Test form data parsing with python-multipart
  - Test static file serving
  - Test error handling and status codes

- [ ] **Task-5.5**: Verify all business rules are preserved
  - Test case-insensitive routing (default: disabled)
  - Test trailing slash handling (default: removed)
  - Test wildcard routes (`*all` for Express 5, `*` for Express 4)
  - Test query string parsing
  - Test subdomain parsing (default offset: 2)
  - Test form submission with button submitter data
  - Test cookie writing and clearing
  - Test pre/post-render hooks execution order

- [ ] **Task-5.6**: Performance testing
  - Benchmark page load times against Express backend
  - Measure bundle size impact (should be unchanged)
  - Test navigation performance with view transitions
  - Verify no performance regressions in DOM updates
  - Test scroll restoration performance

## Phase 6: Cleanup & Documentation

- [ ] **Task-6.1**: Update project documentation
  - **Task-6.1.1**: Update `README.md`
    - Add FastAPI backend section to README
    - Document FastAPI integration approach
    - Update "Why this instead of..." comparisons for Python ecosystem
    - Add FastAPI-specific examples and usage patterns
  - **Task-6.1.2**: Update `USAGE.md`
    - Add FastAPI backend integration examples
    - Document route and template sharing with Python
    - Provide FastAPI-specific configuration options
    - Add code examples for Python/FastAPI usage
  - **Task-6.1.3**: Create `MIGRATION.md` migration guide
    - Document migration from Express to FastAPI backend
    - Step-by-step guide for existing users
    - Document route definition changes (YAML vs JavaScript)
    - Document template sharing strategies
    - Provide troubleshooting guide for common issues

- [ ] **Task-6.2**: Remove or deprecate legacy Express backend samples
  - **Task-6.2.1**: Deprecate Express sample apps
    - Add deprecation notices to `sampleApps/express/` and `sampleApps/express-complex/`
    - Document migration path to FastAPI samples
    - Set timeline for removal (e.g., 6 months)
  - **Task-6.2.2**: Update npm scripts
    - Update scripts to use FastAPI samples by default
    - Keep Express sample scripts with deprecation warnings
    - Document script changes in changelog

- [ ] **Task-6.3**: Configure deployment settings
  - Create Dockerfile for FastAPI backend deployment
  - Document Uvicorn production configuration
  - Document static file serving in production
  - Create deployment guide for Python/FastAPI stack
  - Document environment variable configuration

- [ ] **Task-6.4**: Final code cleanup and optimization
  - Remove unused Express dependencies from FastAPI samples
  - Optimize Python code with black formatter
  - Run linters (ruff or similar) on Python code
  - Ensure consistent code style across backend
  - Remove debug logging from production code

- [ ] **Task-6.5**: Update CHANGELOG.md
  - Document version with FastAPI backend support
  - List breaking changes (none intended for frontend API)
  - Document new features: FastAPI integration, YAML routes, Python models
  - Document deprecations: Express backend samples
  - Provide upgrade guide for users

## Phase 7: Validation and Release

- [ ] **Task-7.1**: End-to-end validation
  - Run complete test suite: frontend + backend + integration tests
  - Manual testing of all sample apps (basic, complex, FastAPI variants)
  - Verify isomorphic flow works end-to-end
  - Test with multiple browsers (Chrome, Firefox, Safari)
  - Verify accessibility features work with screen readers

- [ ] **Task-7.2**: Performance and bundle size verification
  - Measure final bundle size (should match existing)
  - Compare performance metrics: Express vs FastAPI backend
  - Verify no regressions in navigation speed
  - Test memory usage during extended sessions

- [ ] **Task-7.3**: Prepare release
  - Update version in `package.json` and `pyproject.toml`
  - Tag release in git
  - Publish to npm (if frontend changes)
  - Document release notes in CHANGELOG.md
  - Update GitHub releases page

## Phase 8: Post-Launch Support

- [ ] **Task-8.1**: Create troubleshooting documentation
  - Document common issues with FastAPI backend setup
  - Create FAQ for Python/FastAPI integration
  - Provide solutions for template engine issues
  - Document troubleshooting for CORS errors

- [ ] **Task-8.2**: Community support preparation
  - Update issue templates with FastAPI backend tags
  - Create example issues and solutions
  - Prepare migration support resources
  - Document how to report bugs for FastAPI backend

---

**Estimated Timeline:**
- Phase 1: 2-3 days
- Phase 2: 1-2 days
- Phase 3: 5-7 days
- Phase 4: 2-3 days
- Phase 5: 3-5 days
- Phase 6: 2-3 days
- Phase 7: 2-3 days
- Phase 8: 1-2 days

**Total Estimated Duration: 18-28 days**

**Dependencies Between Phases:**
- Phase 1 must complete before Phase 2
- Phase 2 must complete before Phase 3 (backend requires models)
- Phase 3 must complete before Phase 4 (backend must serve static files)
- Phase 4 must complete before Phase 5 (testing requires working frontend+backend)
- Phase 5 must complete before Phase 6 (cleanup only after tests pass)
- Phase 6 must complete before Phase 7 (validation of final state)
- Phase 7 is independent (can proceed in parallel with documentation)
