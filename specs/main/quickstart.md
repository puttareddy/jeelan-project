# Quick Start Guide

## Prerequisites

### Frontend Development
- **Node.js** 18+ (for Webpack and development tools)
- **npm** 8+ (package manager)

### Backend Development (FastAPI)
- **Python** 3.8+ (FastAPI minimum requirement)
- **pip** or **Poetry** (Python package manager)

### Optional Tools
- **Git** (for version control)

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

This installs core dependencies:
- `single-page-express` core library
- `path-to-regexp` (route parsing for Express 5+)
- `path-to-regexp-express4` (Express 4 compatibility)
- `topbar` (loading progress indicator)

### 3. Build Frontend Bundle

```bash
npm run build
```

This generates multi-format bundles in the `dist/` directory:
- `single-page-express.mjs` - ES module (development)
- `single-page-express.cjs` - CommonJS/UMD bundle (development)
- `single-page-express.js` - Standalone UMD bundle (development)
- `single-page-express.min.mjs` - Minified ES module (production)
- `single-page-express.min.js` - Minified standalone UMD bundle (production)

### 4. Set Up Python Backend (Optional - for FastAPI Integration)

```bash
# Navigate to FastAPI sample app
cd sampleApps/fastapi

# Create Python virtual environment (if using Poetry)
python3 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install Python dependencies
pip install fastapi uvicorn[standard] python-multipart pydantic pytest pytest-asyncio httpx
```

**Note:** FastAPI samples are part of the modernization roadmap. See [Implementation Plan](./plan.md) for status.

## Configuration

### Frontend Configuration

The library uses default configuration out of the box. Customization options are documented in [CONFIGURATION.md](../../single-page-express/CONFIGURATION.md):

```javascript
const singlePageExpress = require('single-page-express')

const app = singlePageExpress({
  templatingEngine: teddy,          // Template engine instance
  templates: templates,                // Loaded templates
  defaultTarget: '#page-contents',   // CSS selector for DOM updates
  expressVersion: 5,                 // Express API version (4 or 5)
  caseSensitiveRouting: false,         // Route case sensitivity
  strictRouting: false,               // Trailing slash handling
  queryParser: true,                  // Enable query string parsing
  alwaysScrollTop: false,            // Scroll to top on every render
  alwaysSkipViewTransition: false,     // Disable view transitions
  topbarEnabled: true                 // Enable loading indicator
})
```

### Backend Configuration (Express - Current)

Express sample apps are configured in their respective `server.js` files:

```javascript
// sampleApps/express/server.js
app.use(require('body-parser').urlencoded({ extended: true }))
app.engine('html', require('teddy').__express)
app.set('views', 'mvc/views')
app.set('view engine', 'html')
app.use(express.static('public'))
```

### Backend Configuration (FastAPI - Target)

FastAPI configuration will use `uvicorn` server with ASGI:

```python
# sampleApps/fastapi/server.py (planned)
from fastapi import FastAPI, Request
from fastapi.staticfiles import StaticFiles
import uvicorn

app = FastAPI()
app.mount("/public", StaticFiles(directory="public"), name="public")

if __name__ == "__main__":
    uvicorn.run("server:app", host="0.0.0.0", port=3000, reload=True)
```

## Start Development Server

### Option 1: Basic Frontend-Only Sample

```bash
npm run sample-app-basic-frontend-only
```

Server runs at: http://localhost:3000

### Option 2: Frontend with Templating Sample

```bash
npm run sample-app-basic-frontend-only-with-templating
```

Server runs at: http://localhost:3000

### Option 3: Express Sample (Current Backend)

```bash
npm run express-sample
```

Server runs at: http://localhost:3000

This sample demonstrates:
- Shared routes between server and frontend
- Server-side Teddy templating
- Form handling
- Async data retrieval

### Option 4: Express Complex Sample

```bash
npm run express-complex
```

Server runs at: http://localhost:3000

This sample demonstrates advanced features:
- Route parameters: `/route/:withParam/:anotherParam`
- Head tag management (`res.removeMetaTags`, etc.)
- Scroll position memory
- Focus management
- View transitions
- Partial template rendering
- Verbose output

### Option 5: FastAPI Sample (Target - Post-Migration)

```bash
# Navigate to FastAPI sample
cd sampleApps/fastapi

# Activate virtual environment
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Start Uvicorn server
uvicorn server:app --reload --port 3000
```

**Note:** FastAPI samples are part of the modernization roadmap. See [Implementation Plan](./plan.md) for status.

## Access Application

After starting any sample server:

- **Application:** http://localhost:3000
- **Playwright Tests:** Run with `npm test`

## Project Structure

```
single-page-express/
├── single-page-express.js          # Core library (786 lines)
├── webpack.config.js                # Multi-format build configuration
├── package.json                     # Dependencies and scripts
├── dist/                           # Built library bundles
│   ├── single-page-express.mjs
│   ├── single-page-express.cjs
│   ├── single-page-express.js
│   ├── single-page-express.min.mjs
│   └── single-page-express.min.js
├── sampleApps/                      # Sample applications
│   ├── basicFrontendOnly/            # Frontend-only (no backend)
│   ├── basicFrontendOnlyWithTemplating/  # Frontend with templating
│   ├── express/                      # Express integration (current)
│   │   ├── server.js                 # Express server entry point
│   │   ├── browser.js                # Frontend entry point
│   │   ├── package.json
│   │   ├── styles.css
│   │   └── mvc/
│   │       ├── routes.js              # Shared route definitions
│   │       ├── models/
│   │       │   ├── browser/         # Client-side model functions
│   │       │   └── server/          # Server-side model functions
│   │       └── views/                # Teddy templates
│   ├── express-complex/              # Express with advanced features
│   │   ├── server.js
│   │   ├── browser.js
│   │   ├── package.json
│   │   ├── styles.css
│   │   └── mvc/
│   │       ├── routes.js
│   │       ├── models/
│   │       └── views/
│   ├── fastapi/                      # FastAPI integration (planned)
│   │   ├── server.py                 # FastAPI server entry point
│   │   ├── browser.js                # Frontend entry point (unchanged)
│   │   ├── pyproject.toml            # Python dependencies
│   │   └── mvc/
│   │       ├── routes.yaml             # Route configuration (new format)
│   │       ├── routes.js              # Shared routes (backward compat)
│   │       ├── models/
│   │       │   ├── browser/
│   │       │   └── server/          # Python model functions
│   │       └── views/                # Teddy templates (unchanged)
│   └── fastapi-complex/              # FastAPI with advanced features (planned)
├── test/                            # E2E tests
│   └── tests.spec.js                # Playwright test suite
├── node_modules/                     # npm dependencies
├── README.md                        # Project overview
├── USAGE.md                         # Usage documentation
├── CONFIGURATION.md                 # Configuration reference
├── CHANGELOG.md                     # Version history
├── CONTRIBUTING.md                   # Contribution guidelines
└── LICENSE.md                       # CC-BY-4.0 license
```

## Common Commands

### Development

- `npm run build` - Build frontend library bundles
- `npm test` - Run Playwright E2E tests
- `npm run lint` - Lint JavaScript code with Standard.js
- `npm run lint-fix` - Auto-fix linting issues

### Sample Applications

- `npm run sample-app-basic-frontend-only` - Start basic frontend-only sample
- `npm run sample-app-basic-frontend-only-with-templating` - Start sample with templating
- `npm run express-sample` or `npm run sample3` - Start Express integration sample
- `npm run express-complex` or `npm run sample4` - Start Express complex sample

### Backend (Express)

- `cd sampleApps/express && npm run start` - Start Express server directly

### Backend (FastAPI - Planned)

- `cd sampleApps/fastapi && uvicorn server:app --reload --port 3000` - Start FastAPI server with hot reload

### Testing

- `npm test` - Run full Playwright test suite (launches express-complex sample)

## Troubleshooting

### Port Already in Use

**Error:** `Error: listen EADDRINUSE: address already in use :::3000`

**Solution:**
```bash
# Find process using port 3000
lsof -i :3000  # macOS/Linux
netstat -ano | findstr :3000  # Windows

# Kill the process or use a different port
# Modify server port in server.js or server.py
```

### Build Failures

**Error:** Webpack build fails with module resolution errors

**Solution:**
```bash
# Clear node_modules and reinstall
rm -rf node_modules package-lock.json
npm install

# Clear dist directory
rm -rf dist
npm run build
```

### Test Failures

**Error:** Playwright tests fail with timeout or element not found

**Solution:**
```bash
# Ensure sample server is running
npm run express-complex

# Reinstall Playwright browsers
npx playwright install

# Run tests with debug mode
npx playwright test --debug
```

### Python Environment Issues

**Error:** ModuleNotFoundError or Python version incompatibility

**Solution:**
```bash
# Verify Python version
python --version  # Should be 3.8+

# Recreate virtual environment
rm -rf venv
python3 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt
```

## Next Steps

After setup, proceed with:

1. **Review Documentation**
   - [README.md](../../single-page-express/README.md) - Project overview and philosophy
   - [USAGE.md](../../single-page-express/USAGE.md) - Detailed usage examples
   - [CONFIGURATION.md](../../single-page-express/CONFIGURATION.md) - Configuration options

2. **Explore Sample Apps**
   - Start with `basicFrontendOnly` for minimal setup
   - Progress to `express` for isomorphic routing patterns
   - Review `express-complex` for advanced features

3. **Review Modernization Plan**
   - [Implementation Plan](./plan.md) - Phase-by-phase migration to FastAPI backend
   - [Task List](./tasks.md) - Detailed task breakdown with priorities

4. **Start Implementing**
   - Use `/speckit.implement` command to begin task execution
   - Tasks marked with [P] can be executed in parallel
   - Follow phase order in the implementation plan

5. **Testing Verification**
   - Run existing Playwright tests against new FastAPI backend
   - Create new pytest tests for Python-specific functionality
   - Verify isomorphic flow works end-to-end

## Additional Resources

- **Project Homepage:** https://rooseveltframework.org/docs/single-page-express/latest
- **GitHub Repository:** https://github.com/rooseveltframework/single-page-express
- **npm Package:** https://www.npmjs.com/package/single-page-express
- **FastAPI Documentation:** https://fastapi.tiangolo.com/
- **Uvicorn Documentation:** https://www.uvicorn.org/
