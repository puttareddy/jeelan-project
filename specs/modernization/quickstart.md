# Quick Start Guide

## Prerequisites

**Frontend (Node.js):**
- Node.js 18+ (ES module support required for Webpack)
- npm 9+ (comes with Node.js)

**Backend (Python - FastAPI):**
- Python 3.8+ (minimum for FastAPI)
- pip or Poetry for dependency management
- Uvicorn (ASGI server)

**Development Tools:**
- Git (for cloning repository)
- Modern web browser with View Transitions API support (Chrome 111+, Edge 111+, Safari 18+)

**Optional:**
- Docker (for containerized deployment)
- PostgreSQL 15+ (if adding database persistence in future)

## Installation

### 1. Clone Repository

```bash
git clone https://github.com/rooseveltframework/single-page-express.git
cd single-page-express
```

### 2. Install Frontend Dependencies

```bash
npm install
```

This installs:
- `path-to-regexp` (route parsing)
- `path-to-regexp-express4` (Express 4 compatibility)
- `topbar` (loading indicator)
- Dev dependencies: `@playwright/test`, `express`, `standard`, `teddy`, `webpack-cli`

### 3. Configure Environment

Frontend (no .env file required):
- Frontend runs in browser, no environment configuration needed
- Optional: Set `PORT` environment variable if using sample apps

Backend (FastAPI sample apps):

```bash
# For Poetry (recommended)
cd sampleApps/fastapi
poetry install

# OR for pip
cd sampleApps/fastapi
pip install -r requirements.txt
```

FastAPI dependencies include:
- `fastapi` (latest stable)
- `uvicorn[standard]` (ASGI server with hot reload)
- `python-multipart` (form data parsing)
- `pydantic` (data validation)
- `jinja2` (template engine, optional for Teddy alternative)

### 4. Build Frontend Bundle

```bash
npm run build
```

This generates 5 bundle formats in `dist/`:
- `single-page-express.mjs` - ES module (development)
- `single-page-express.cjs` - CommonJS/UMD bundle (development)
- `single-page-express.js` - Standalone UMD bundle (development)
- `single-page-express.min.mjs` - Minified ES module (production)
- `single-page-express.min.js` - Minified standalone UMD bundle (production)

### 5. Set Up Database

**No database setup required.**

Single-page-express is a client-side routing library with optional server integration. Data is ephemeral and passed through request/response cycles. If you add database persistence in the future, follow your chosen database's setup instructions.

### 6. Start Development Server

**Option A: Frontend-Only Development**

```bash
# Basic frontend-only sample
npm run sample-app-basic-frontend-only
# Or: npm run sample1

# Basic frontend-only with templating
npm run sample-app-basic-frontend-only-with-templating
# Or: npm run sample2
```

**Option B: Express Backend (Legacy)**

```bash
# Express sample app
npm run express-sample
# Or: npm run sample3
# Or: cd sampleApps/express && npm ci && npm start

# Express complex sample app
npm run express-complex
# Or: npm run sample4
```

**Option C: FastAPI Backend (New - After Migration)**

```bash
# Terminal 1: Backend (FastAPI)
cd sampleApps/fastapi
uvicorn server:app --reload --port 3000

# Terminal 2: Frontend (if separate dev server needed)
npm run build
```

**Option D: Full Stack with Hot Reload**

```bash
# Terminal 1: Backend (FastAPI with hot reload)
cd sampleApps/fastapi
uvicorn server:app --reload --port 3000

# Terminal 2: Frontend (watch mode)
npm run build -- --watch
```

### 7. Access Application

- **Frontend-Only Sample:** http://localhost:3000
- **Express Sample:** http://localhost:3000
- **Express Complex Sample:** http://localhost:3000
- **FastAPI Sample:** http://localhost:3000 (after migration)
- **API Documentation (FastAPI only):** http://localhost:3000/docs (auto-generated OpenAPI docs)

## Project Structure

```
single-page-express/
├── dist/                          # Frontend bundle artifacts
│   ├── single-page-express.mjs       # ES module
│   ├── single-page-express.cjs       # CommonJS
│   ├── single-page-express.js        # Standalone UMD
│   ├── single-page-express.min.mjs   # Minified ES module
│   └── single-page-express.min.js    # Minified UMD
├── sampleApps/
│   ├── basicFrontendOnly/            # Frontend-only demo
│   │   ├── index.html
│   │   └── index.js
│   ├── basicFrontendOnlyWithTemplating/
│   │   ├── index.html
│   │   └── index.js
│   ├── express/                      # Express backend (legacy)
│   │   ├── server.js                  # Express server entry point
│   │   ├── browser.js                 # Frontend entry point
│   │   ├── package.json
│   │   ├── styles.css
│   │   └── mvc/
│   │       ├── routes.js                # Shared route definitions
│   │       ├── models/
│   │       │   ├── browser/
│   │       │   │   └── getRandomNumber.js
│   │       │   └── server/
│   │       │       └── getRandomNumber.js
│   │       └── views/
│   │           ├── index.html
│   │           ├── layout.html
│   │           ├── pageWithDataRetrieval.html
│   │           ├── pageWithForm.html
│   │           └── secondPage.html
│   ├── express-complex/              # Express complex backend (legacy)
│   │   ├── server.js
│   │   ├── browser.js
│   │   ├── package.json
│   │   ├── styles.css
│   │   └── mvc/
│   │       ├── routes.js
│   │       ├── models/
│   │       └── views/
│   ├── fastapi/                     # FastAPI backend (new - after migration)
│   │   ├── server.py                  # FastAPI server entry point
│   │   ├── browser.js                 # Frontend entry point (unchanged)
│   │   ├── pyproject.toml              # Python dependencies
│   │   ├── requirements.txt            # pip requirements
│   │   ├── styles.css
│   │   └── mvc/
│   │       ├── routes.js                # Shared route definitions (unchanged)
│   │       ├── routes.yaml              # New: Route configuration for FastAPI
│   │       ├── models/
│   │       │   ├── browser/
│   │       │   │   └── getRandomNumber.js
│   │       │   └── server/
│   │       │       └── getRandomNumber.py  # Python model equivalent
│   │       └── views/
│   │           ├── index.html
│   │           ├── layout.html
│   │           ├── pageWithDataRetrieval.html
│   │           ├── pageWithForm.html
│   │           └── secondPage.html
│   └── fastapi-complex/              # FastAPI complex backend (new)
│       ├── server.py
│       ├── browser.js
│       ├── pyproject.toml
│       ├── styles.css
│       └── mvc/
│           ├── routes.js
│           ├── routes.yaml
│           ├── models/
│           └── views/
├── test/                           # Playwright E2E tests
│   └── tests.spec.js                # Test specifications
├── specs/                           # Project specifications
│   └── modernization/               # Modernization project docs
│       ├── constitution.md
│       ├── spec.md
│       ├── plan.md
│       ├── tasks.md
│       ├── data-model.md
│       └── quickstart.md             # This file
├── single-page-express.js           # Core library (786 lines)
├── webpack.config.js                # Multi-format build configuration
├── package.json                     # Frontend dependencies and scripts
├── README.md                        # Project overview
├── USAGE.md                         # Usage documentation
├── CONFIGURATION.md                 # Configuration documentation
├── CONTRIBUTING.md                  # Contribution guidelines
├── CHANGELOG.md                     # Version history
├── LICENSE.md                       # CC-BY-4.0 license
└── playwright.config.js              # Playwright test configuration
```

## Common Commands

### Frontend Development

- `npm run build` - Build frontend library (generates dist/ bundles)
- `npm run sample-app-basic-frontend-only` - Run basic frontend-only sample
- `npm run sample-app-basic-frontend-only-with-templating` - Run sample with templating
- `npm run express-sample` - Run Express sample app (legacy)
- `npm run express-complex` - Run Express complex sample (legacy)

### Backend Development (Express - Legacy)

- `cd sampleApps/express && npm start` - Start Express server
- `cd sampleApps/express-complex && npm start` - Start Express complex server

### Backend Development (FastAPI - New)

- `cd sampleApps/fastapi && uvicorn server:app --reload --port 3000` - Start FastAPI server with hot reload
- `cd sampleApps/fastapi-complex && uvicorn server:app --reload --port 3000` - Start FastAPI complex server
- `cd sampleApps/fastapi && poetry install` - Install Python dependencies with Poetry
- `cd sampleApps/fastapi && pip install -r requirements.txt` - Install Python dependencies with pip

### Testing

- `npm test` - Run Playwright E2E tests (builds first, then tests)
- `npm run lint` - Run Standard.js linter
- `npm run lint-fix` - Fix linting issues automatically
- `cd sampleApps/fastapi && pytest` - Run Python backend tests (after migration)

### Building

- `npm run build` - Build frontend library for distribution
- Webpack generates 5 bundle formats automatically

### Database

**No database commands required** - library uses in-memory data only.

## Troubleshooting

### Port Already in Use

**Symptom:** `Error: listen EADDRINUSE: address already in use :::3000`

**Solutions:**
- Find and kill the process using port 3000:
  ```bash
  # macOS/Linux
  lsof -ti:3000 | xargs kill -9

  # Windows
  netstat -ano | findstr :3000
  taskkill /PID <PID> /F
  ```
- Change port in server startup command:
  ```bash
  uvicorn server:app --reload --port 3001
  ```

### Webpack Build Fails

**Symptom:** Build errors or missing dependencies

**Solutions:**
- Clear node_modules and reinstall:
  ```bash
  rm -rf node_modules package-lock.json
  npm install
  ```
- Ensure Node.js version is 18+:
  ```bash
  node --version  # Should be v18.x.x or higher
  ```
- Check for missing peer dependencies

### Python Import Errors (FastAPI)

**Symptom:** `ModuleNotFoundError: No module named 'fastapi'`

**Solutions:**
- Activate virtual environment (if using Poetry):
  ```bash
  poetry shell
  ```
- Install dependencies:
  ```bash
  pip install -r requirements.txt
  # OR
  poetry install
  ```
- Verify Python version:
  ```bash
  python --version  # Should be 3.8+
  ```

### Template Rendering Issues

**Symptom:** Templates not rendering or errors on page load

**Solutions:**
- Verify templates are bundled correctly:
  ```bash
  # Check templates.js exists
  ls sampleApps/express/public/templates.js
  ```
- For FastAPI, ensure Teddy integration works:
  - Check PyExecJS is installed (if using Teddy with Python)
  - Verify template paths are correct in `routes.yaml`

### Playwright Tests Fail

**Symptom:** Tests timeout or fail to connect

**Solutions:**
- Install Playwright browsers:
  ```bash
  npx playwright install
  ```
- Ensure sample app server is running:
  ```bash
  npm run express-complex
  ```
- Increase timeout in test if needed (modify `tests.spec.js`)

### View Transitions Not Working

**Symptom:** Page transitions not animating

**Solutions:**
- Check browser supports View Transitions API:
  ```javascript
  console.log(document.startViewTransition) // Should be a function
  ```
- Use modern browser (Chrome 111+, Edge 111+, Safari 18+)
- Fallback to CSS transitions is automatic if View Transitions not supported

## Next Steps

After setup:

1. **Review Documentation**
   - Read [Implementation Plan](./plan.md) for migration phases
   - Check [Task List](./tasks.md) for detailed work items
   - Review [Data Model](./data-model.md) for entity relationships
   - Study [Constitution](./constitution.md) for governing principles

2. **Explore Sample Apps**
   - Run basic frontend-only sample to understand core functionality
   - Run Express sample to see isomorphic routing (current implementation)
   - Run Express complex sample for advanced features
   - (After migration) Run FastAPI sample to see Python backend integration

3. **Start Development**
   - Choose backend: Express (legacy) or FastAPI (new)
   - Define routes using Express-compatible syntax
   - Implement templates with Teddy or Jinja2
   - Use `/speckit.implement` command to begin implementation

4. **Testing**
   - Run `npm test` to verify all Playwright tests pass
   - Create custom tests for your specific use case
   - Test on multiple browsers (Chrome, Firefox, Safari)

5. **Migration Resources**
   - Read [USAGE.md](../../USAGE.md) for library usage patterns
   - Review [CONFIGURATION.md](../../CONFIGURATION.md) for configuration options
   - Check [MIGRATION.md](./MIGRATION.md) (after creation) for Express to FastAPI migration guide
   - Visit https://rooseveltframework.org/docs/single-page-express/latest for additional docs

## Getting Help

- **Documentation:** https://rooseveltframework.org/docs/single-page-express/latest
- **Issues:** https://github.com/rooseveltframework/single-page-express/issues
- **Contributing:** See [CONTRIBUTING.md](../../CONTRIBUTING.md)
- **License:** CC-BY-4.0 (see [LICENSE.md](../../LICENSE.md))

## Architecture Overview

**Frontend (Unchanged):**
- Single-Page-Express library (`single-page-express.js`) provides Express-like routing in browser
- Webpack bundles library into multiple formats (ESM, CommonJS, UMD)
- Teddy templating engine for client-side rendering
- Playwright for E2E testing

**Backend Options:**

**Express (Legacy - Current):**
- Node.js with Express.js 5.2.1
- Teddy templating engine
- MVC pattern in sample apps
- Shared routes between server and client

**FastAPI (New - After Migration):**
- Python 3.8+ with FastAPI
- Teddy or Jinja2 templating
- YAML route configuration (new)
- Python model functions (async)
- Uvicorn ASGI server with hot reload

**Isomorphic Pattern:**
- Routes defined once, used on both server and client
- Templates shared between server and client
- Model functions duplicated per language (JavaScript/Python)
- Request/response objects compatible on both sides
