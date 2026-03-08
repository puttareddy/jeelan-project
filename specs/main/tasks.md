# Modernization Tasks

## Infrastructure Setup (5 tasks)
- [ ] **Task-1** [P] Initialize FastAPI project structure in `/Users/puttaiaharugunta/codebase/infi/single-page-express/sampleApps/fastapi/`
- [ ] **Task-2** [P] Configure Python development environment with Poetry
- [ ] **Task-3** Set up Uvicorn ASGI server with hot reload configuration
- [ ] **Task-4** Configure build pipeline for FastAPI sample apps (package.json scripts, proxy settings)
- [ ] **Task-5** [P] Set up pytest testing framework with asyncio and httpx support

## Database Migration (4 tasks)
- [ ] **Task-6** Document that no persistent database migration is required (library uses in-memory data)
- [ ] **Task-7** [P] Translate JavaScript model functions to Python async functions (getRandomNumber.js → get_random_number.py)
- [ ] **Task-8** Create Python model directory structure in `sampleApps/fastapi/mvc/models/server/`
- [ ] **Task-9** Verify data integrity between JavaScript and Python model implementations

## Backend Migration (15 tasks)
- [ ] **Task-10** Create FastAPI application instance in `sampleApps/fastapi/server.py`
- [ ] **Task-11** Implement static file serving from `/public/` directory using FastAPI StaticFiles
- [ ] **Task-12** Implement form data parsing with python-multipart for application/x-www-form-urlencoded
- [ ] **Task-13** Design and implement YAML route configuration format in `sampleApps/fastapi/mvc/routes.yaml`
- [ ] **Task-14** [P] Create Python route registration system from YAML config
- [ ] **Task-15** Implement Express-compatible route parser for path parameters (/route/:id), query strings (?key=value), wildcards (*all)
- [ ] **Task-16** Integrate Teddy templating engine with Python via subprocess or JavaScript engine
- [ ] **Task-17** Create Teddy template loader from `mvc/views/` directory
- [ ] **Task-18** Implement server-side template rendering using Teddy engine
- [ ] **Task-19** [P] Create Python equivalent of template bundling logic (templates.js generation)
- [ ] **Task-20** [P] Create FastAPI middleware for Express-like behavior
- [ ] **Task-21** Implement cookie reading/writing support (req.cookies, res.cookie(), res.clearCookie())
- [ ] **Task-22** Implement JSON API endpoint `/api/randomNumber` with FastAPI
- [ ] **Task-23** [P] Implement async model loading pattern for data retrieval routes
- [ ] **Task-24** Create error handlers with proper HTTP status codes (404, 500, etc.)

## Frontend Migration (5 tasks)
- [ ] **Task-25** [P] Verify Webpack build produces same bundled JavaScript in `dist/` directory
- [ ] **Task-26** [P] Create `sampleApps/fastapi/browser.js` (copy from express/browser.js, unchanged)
- [ ] **Task-27** [P] Create `sampleApps/fastapi-complex/browser.js` (copy from express-complex/browser.js, unchanged)
- [ ] **Task-28** Verify frontend JavaScript loads correctly from FastAPI static files
- [ ] **Task-29** Update npm scripts in root `package.json` for FastAPI sample apps

## Styling Migration (2 tasks)
- [ ] **Task-30** [P] Copy `styles.css` from `sampleApps/express/` to `sampleApps/fastapi/`
- [ ] **Task-31** [P] Copy `styles.css` from `sampleApps/express-complex/` to `sampleApps/fastapi-complex/`

## Testing (12 tasks)
- [ ] **Task-32** Create pytest test suite in `sampleApps/fastapi/tests/`
- [ ] **Task-33** [P] Write unit tests for FastAPI route registration from YAML config
- [ ] **Task-34** [P] Write unit tests for template rendering with Python Teddy integration
- [ ] **Task-35** [P] Write unit tests for form data parsing and validation
- [ ] **Task-36** [P] Write unit tests for async model loading functions
- [ ] **Task-37** [P] Write unit tests for cookie handling (req.cookies, res.cookie(), res.clearCookie())
- [ ] **Task-38** Create integration tests using FastAPI TestClient for full request/response cycles
- [ ] **Task-39** Update `test/tests.spec.js` to launch FastAPI server instead of Express server
- [ ] **Task-40** Verify homepage renders correctly with FastAPI backend
- [ ] **Task-41** Verify second page renders with variable passing
- [ ] **Task-42** Verify form submission handling works
- [ ] **Task-43** Verify async data retrieval works

## Documentation (7 tasks)
- [ ] **Task-44** Update `README.md` with FastAPI backend section and integration approach
- [ ] **Task-45** Add FastAPI backend integration examples to `USAGE.md`
- [ ] **Task-46** Document route and template sharing with Python in `USAGE.md`
- [ ] **Task-47** Create `MIGRATION.md` with step-by-step guide from Express to FastAPI backend
- [ ] **Task-48** Document YAML route definition format in `MIGRATION.md`
- [ ] **Task-49** Document development workflow (separate frontend/backend dev servers)
- [ ] **Task-50** Update `CHANGELOG.md` with FastAPI backend support, new features, and deprecations

## Cleanup & Release (6 tasks)
- [ ] **Task-51** Add deprecation notices to `sampleApps/express/` and `sampleApps/express-complex/` directories
- [ ] **Task-52** Document migration path to FastAPI samples in deprecation notices
- [ ] **Task-53** Create Dockerfile for FastAPI backend deployment
- [ ] **Task-54** Document Uvicorn production configuration
- [ ] **Task-55** Run black or ruff linter on Python code for consistent style
- [ ] **Task-56** Prepare release notes with version update in package.json and pyproject.toml

## Total: ~56 tasks

---
**Note:**
- Tasks marked with [P] can be executed in parallel. Others must run sequentially.
- Each task has a unique **Task-N** identifier for better tracking and reference.
