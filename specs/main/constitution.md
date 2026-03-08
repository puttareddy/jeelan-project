# Modernization Constitution

## Project Identity
- **Project Name:** single-page-express-fastapi-migration
- **Source Path:** /Users/puttaiaharugunta/codebase/infi/single-page-express
- **Strategy:** hybrid (preserve frontend JavaScript library, replace backend Express with FastAPI)

## Governing Principles
1. **Preserve Business Logic** — All existing Express-compatible routing behavior, request/response objects, and DOM manipulation logic must be faithfully reproduced in the new backend
2. **Data Integrity** — No data loss during migration; all route definitions, templates, and user flows must be migrated correctly
3. **Contract Compliance** — The frontend JavaScript API surface must remain 100% compatible; external consumers of `single-page-express` npm package must experience zero breaking changes
4. **Incremental Verification** — Each migration step must be independently testable via existing Playwright test suite and new FastAPI-specific tests
5. **No Regressions** — All existing functionality including routing, DOM updates, scroll position memory, view transitions, accessibility features, and form handling must work identically in the new stack

## Current State
- **Languages:** JavaScript (ES6+), Node.js (backend)
- **Frontend:** JavaScript with Webpack bundler, Playwright for E2E testing, Standard.js linting, Topbar loading indicator, Teddy templating (client-side)
- **Backend:** Node.js with Express.js 5.2.1, Teddy templating engine
- **Database:** None (client-side routing library with optional server integration)
- **Styling:** CSS (no CSS framework)

## Target State
- **Frontend:** JavaScript (ES6+) — unchanged from current state
- **Backend:** Python 3.8+ with FastAPI (latest stable), Uvicorn ASGI server
- **Database:** None — unchanged from current state
- **Styling:** CSS — unchanged from current state

## Migration Boundaries

### MUST Preserve
**Frontend API Surface:**
- All Express-compatible routing methods: `app.get()`, `app.post()`, `app.put()`, `app.delete()`, `app.all()`, `app.route()`
- Route parameter parsing (`/users/:id`), query string parsing (`?key=value`), wildcard routes (`*all`, `*`)
- Request object properties: `req.params`, `req.query`, `req.body`, `req.cookies`, `req.path`, `req.method`, `req.url`, `req.app`, `req.hostname`, `req.originalUrl`, `req.route`, `req.secure`, `req.subdomains`
- Response methods: `res.render()`, `res.redirect()`, `res.cookie()`, `res.clearCookie()`, `res.json()`
- All SPE-specific `res` properties: `res.title`, `res.target`, `res.focus`, `res.beforeRender`, `res.afterRender`, `res.removeMetaTags`, `res.removeStyleTags`, `res.removeLinkTags`, `res.removeScriptTags`, `res.removeBaseTags`, `res.removeTemplateTags`, `res.removeHeadTags`, `res.skipViewTransition`, `res.updateDelay`, `res.resetScroll`
- Pre/post-render hooks: `app.beforeEveryRender()`, `app.afterEveryRender()`, `res.beforeRender()`, `res.afterRender()`
- Form submission handling and body parsing
- Cookie reading and writing (`req.cookies`, `res.cookie()`, `res.clearCookie()`)
- Default render method with intelligent DOM manipulation
- View Transitions API integration with fallback to CSS transitions
- Scroll position memory and restoration
- Focus management and ARIA live region announcements for accessibility
- Head tag management (adding/updating `<link>`, `<script>`, `<meta>`, `<style>` tags)
- Multiple DOM targets support
- Topbar loading indicator integration
- Back/forward button handling with `req.backButtonPressed` and `req.forwardButtonPressed`
- Custom render method support
- Middleware support for routes
- Programmatic route triggering via `app.triggerRoute()`

**Build and Distribution:**
- Webpack multi-format build configuration (ESM, CommonJS, UMD, minified variants)
- npm package structure and export format
- Source maps generation

**Testing:**
- All existing Playwright test specifications in `/test/tests.spec.js`
- Same test coverage and verification approach
- Sample applications serve as integration tests

**Documentation:**
- All existing features documented in USAGE.md and CONFIGURATION.md
- README.md overview and "Why this instead of..." comparisons

### MAY Change
**Backend Implementation:**
- Backend implementation language: Node.js/Express → Python/FastAPI
- Backend build process: Webpack → Python packaging tools (Poetry/pip)
- Backend test execution: Node.js → Python (pytest/unittest)
- Server startup and initialization patterns
- Development server hot reload mechanism (Uvicorn --reload)

**Sample Applications:**
- Replace Express sample apps (`sampleApps/express/`, `sampleApps/express-complex/`) with FastAPI equivalents
- New sample app directory structure for FastAPI samples
- Python-based server entry points (`server.py` instead of `server.js`)

**Shared Route/Template Format:**
- Introduce new format for route definition sharing between Python and JavaScript (YAML/JSON configuration file or DSL)
- Template bundling approach for client-side delivery
- Data fetching patterns for async model loading

**API Endpoints:**
- New FastAPI endpoints for server-side data fetching
- Different API endpoint paths if needed for FastAPI patterns

**Development Workflow:**
- Separate frontend and backend development servers
- Proxy configuration for API calls during development
- Project scripts in package.json updated for FastAPI workflow

### WILL Drop
- Node.js/Express backend implementation (will be replaced, not migrated)
- Express-based sample app servers (`sampleApps/express/server.js`, `sampleApps/express-complex/server.js`)
- Teddy backend integration on server-side (if using alternative template engine)
- Webpack bundling for sample apps (Python bundling instead)
- Any legacy patterns deprecated in Express 5 that are no longer relevant

## Agent Guidelines

### Code Reading and Analysis
- Agents MUST read existing source files before rewriting or replacing functionality
- Key files to reference: `single-page-express.js` (main library), `sampleApps/express/server.js`, `sampleApps/express/mvc/routes.js`, `webpack.config.js`, `test/tests.spec.js`
- Understand current routing patterns, request/response object implementations, and DOM manipulation logic before backend migration

### Verification and Testing
- Each feature MUST include verification that the migrated FastAPI backend produces the same behavior as Express backend
- All existing Playwright tests MUST pass against new FastAPI backend
- Create new Python-specific tests for backend functionality (pytest)
- Integration tests must verify full isomorphic flow (server-rendered initial page, client-side navigation)
- Compare output HTML, headers, and behavior between Express and FastAPI implementations

### Database Migration
- No database migration required (no persistent data models)
- Template migration must ensure identical output between server and client rendering
- Route configuration migration must preserve all route definitions and handler logic

### API Compatibility
- Frontend JavaScript API MUST maintain backwards compatibility
- No changes to `single-page-express.js` public methods, properties, or behaviors
- External API contracts (for npm consumers) must remain unchanged
- Any FastAPI-specific features MUST be additions, not modifications to existing behavior

### Documentation Requirements
- Document FastAPI integration in USAGE.md
- Create migration guide from Express to FastAPI backend
- Update README.md to reflect FastAPI backend option
- Document route and template sharing approach chosen
- Provide clear setup instructions for FastAPI development environment
- approved
