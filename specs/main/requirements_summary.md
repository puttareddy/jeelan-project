# Single-Page-Express Modernization: FastAPI Backend Migration

## Project Name and Purpose

**Project:** Single-Page-Express with FastAPI Backend Migration

**Purpose:** Modernize the single-page-express library by replacing the Node.js/Express backend with Python/FastAPI while preserving all existing frontend functionalities. The goal is to enable isomorphic routing with a Python backend while maintaining the Express-compatible API surface for frontend developers.

---

## Existing Codebase Analysis

### Languages and Frameworks

**Frontend:**
- **Language:** JavaScript (ES6+)
- **Core Library:** `single-page-express.js` (786 lines)
- **Build System:** Webpack (produces ESM, CommonJS, and standalone UMD bundles)
- **Testing:** Playwright for E2E tests
- **Linting:** Standard.js

**Backend (Current - To Be Replaced):**
- **Language:** Node.js
- **Framework:** Express.js
- **Templating:** Teddy (Express-compatible) in sample apps

**Backend (Target):**
- **Language:** Python
- **Framework:** FastAPI

### Architecture

**Frontend Architecture:**
- **Core Module:** Single entry point that implements Express-like routing in browser
- **Route Handler:** Hijacks `<a>` tag clicks and `<form>` submissions
- **Request/Response Objects:** Partial implementations of Express `req` and `res` objects adapted for browser context
- **Default Render Method:** Intelligent DOM manipulation with:
  - Targeted element replacement
  - Head tag management (adding new `<link>`, `<script>`, `<meta>`, `<style>` tags)
  - View Transitions API integration
  - Scroll position memory and restoration
  - Focus management for accessibility
  - Screen reader announcements via ARIA live regions
- **Event Listeners:**
  - Click events on anchors
  - Submit events on forms
  - Popstate events for back/forward button handling
- **Hooks:** Pre-render (`beforeEveryRender`, `beforeRender`) and post-render (`afterEveryRender`, `afterRender`, `postRenderCallbacks`)
- **Loading Indicator:** Optional topbar loading progress bar

**Backend Architecture (Current - Express):**
- MVC pattern in sample apps (models, views, routes)
- Shared routes between server and frontend (`routes.js`)
- Template bundling for client-side delivery
- Webpack bundling for frontend JavaScript
- Static file serving
- Body parsing for form submissions

**Key Components:**
- `single-page-express.js` - Core library implementing Express API on client
- `webpack.config.js` - Multi-format build configuration
- Sample apps demonstrating:
  1. Basic frontend-only
  2. Frontend with templating
  3. Express integration (shared routes/templates)
  4. Express complex integration

### Data Models

No persistent data models - this is a client-side routing library with optional server integration. Data is passed through:
- Request parameters (`req.params`, `req.query`, `req.body`)
- Template models passed to render functions
- Route-specific data retrieval (async model loading)

### APIs

**Frontend API (Express-Compatible):**
- `app.get()`, `app.post()`, `app.put()`, `app.delete()`, `app.all()`
- `app.route()` - Chainable route registration
- `app.triggerRoute()` - Programmatic route triggering
- `req.params`, `req.query`, `req.body`, `req.cookies`, `req.path`, `req.method`, `req.url`, etc.
- `res.render()`, `res.redirect()`, `res.cookie()`, `res.clearCookie()`
- `res.title`, `res.target`, `res.focus`, `res.removeHeadTags`, etc. (SPE-specific)
- `app.beforeEveryRender()`, `app.afterEveryRender()`
- `app.set()`, `app.get()` for app-level configuration

**Backend API (Current - Express):**
- HTTP endpoint routing
- Template rendering with Express-compatible engines
- Static file serving
- Form body parsing
- JSON API endpoints (in sample apps)

**Backend API (Target - FastAPI):**
- FastAPI route decorators
- Template rendering (Jinja2 or custom)
- Static file serving
- Form data parsing
- JSON API endpoints

### Testing Approach

- **Playwright** for end-to-end browser testing
- Tests verify routing, DOM updates, scroll behavior, accessibility
- Sample apps serve as integration tests
- 4 sample applications demonstrating different usage patterns

---

## Problem Statement and Target Users

### Problem Statement

The single-page-express library currently demonstrates isomorphic routing using Node.js/Express on both server and client. However, many developers prefer Python for backend development, particularly with modern frameworks like FastAPI that offer performance benefits and native async/await support. There is no clear path to use single-page-express with a Python backend while preserving the isomorphic architecture.

### Target Users

1. **Frontend Developers** who want to build SPAs using Express-like routing patterns
2. **Full-Stack Python Developers** who prefer FastAPI over Node.js/Express
3. **Teams** with Python backend expertise seeking to share routing logic between server and client
4. **Developers Building Isomorphic Applications** who want code reuse across server and client

---

## What Must Be Preserved vs. What Can Change

### Must Preserve (Non-Negotiable)

**Frontend Functionality:**
1. All Express-compatible routing methods (`get`, `post`, `put`, `delete`, `all`, `route`)
2. Route parameter parsing (`/route/:id`)
3. Query string parsing (`?key=value`)
4. Wildcard routes (Express 5: `*all`, Express 4: `*`)
5. Form submission handling and body parsing
6. Cookie reading and writing (`req.cookies`, `res.cookie()`, `res.clearCookie()`)
7. Default render method with DOM manipulation
8. View Transitions API integration
9. Scroll position memory and restoration
10. Focus management and accessibility features (ARIA announcements)
11. Head tag management (adding/updating `<link>`, `<script>`, `<meta>`, `<style>` tags)
12. Pre/post-render hooks (`beforeEveryRender`, `afterEveryRender`, etc.)
13. Multiple DOM targets support
14. Topbar loading indicator
15. Back/forward button handling
16. Custom render method support
17. Middleware support for routes
18. Request properties: `req.app`, `req.body`, `req.cookies`, `req.hostname`, `req.method`, `req.originalUrl`, `req.params`, `req.path`, `req.protocol`, `req.query`, `req.res`, `req.route`, `req.secure`, `req.subdomains`
19. Response methods: `res.cookie()`, `res.clearCookie()`, `res.redirect()`, `res.render()`, `res.json()`
20. All SPE-specific `res` properties: `res.title`, `res.target`, `res.focus`, `res.beforeRender`, `res.afterRender`, `res.removeMetaTags`, `res.removeStyleTags`, `res.removeLinkTags`, `res.removeScriptTags`, `res.removeBaseTags`, `res.removeTemplateTags`, `res.removeHeadTags`, `res.skipViewTransition`, `res.updateDelay`, `res.resetScroll`

**API Surface:**
1. Same constructor interface
2. Same app, request, and response object methods/properties
3. Same behavior for all documented features

**Testing:**
1. All existing Playwright tests must pass
2. Same level of test coverage

**Documentation:**
1. All existing features must remain documented
2. New FastAPI integration must be documented

### Can Change (Acceptable)

1. **Backend implementation language:** Node.js/Express → Python/FastAPI
2. **Backend build process:** Webpack → Python packaging tools
3. **Backend test execution:** Node.js → Python
4. **Template engine approach:** May adapt for Python/FastAPI compatibility (see Requirements)
5. **Sample app implementations:** Replace Express samples with FastAPI equivalents
6. **Shared route definition format:** May introduce new format for Python/JavaScript sharing
7. **API endpoints:** New FastAPI endpoints for server-side data fetching

---

## Key Features and User Stories

### Existing Features to Preserve (Frontend)

1. **Express-Compatible Routing**
   - As a frontend developer, I want to define routes using `app.get()`, `app.post()`, `app.route()`, etc., so I can use familiar Express syntax
   - Support for route parameters: `app.get('/users/:id', handler)`
   - Support for query strings: `app.get('/search', handler)` with access to `req.query`
   - Support for wildcard routes: `app.route('*all').get(handler)`

2. **Form and Link Hijacking**
   - As a developer, I want link clicks and form submits to be intercepted and handled by the router, so navigation happens without page reloads
   - Automatic detection of `<a>` tags with `href` attributes
   - Automatic detection of `<form>` tags with `action` attributes
   - Support for GET and POST methods

3. **Request Object**
   - As a route handler, I want access to `req.params`, `req.query`, `req.body`, `req.cookies`, `req.path`, etc., matching Express API
   - Support for all documented `req` properties and methods

4. **Response Object**
   - As a route handler, I want to call `res.render()`, `res.redirect()`, `res.cookie()`, etc., matching Express API
   - Support for all documented `res` properties and methods

5. **Default Render Method**
   - As a developer, I want intelligent DOM updates without writing custom render logic
   - Automatic title updates from `res.title` or `<title>` tags
   - Targeted element replacement via `res.target` or `app.defaultTarget`
   - Support for multiple DOM targets

6. **Head Tag Management**
   - As a developer, I want new `<link>`, `<script>`, `<meta>`, `<style>` tags to be added to head automatically
   - Support for removing old tags via `res.removeMetaTags`, `res.removeStyleTags`, etc.
   - Delay DOM update until external resources load to prevent FOUC

7. **View Transitions**
   - As a developer, I want smooth page transitions using browser's View Transitions API
   - Automatic integration when supported, fallback to CSS transitions
   - Option to skip view transitions via `res.skipViewTransition` or `app.alwaysSkipViewTransition`

8. **Scroll Position Memory**
   - As a user, I want the page to remember my scroll position when I return to a previously visited page
   - Support for scroll position in child containers with `id` attributes
   - Option to disable via `app.alwaysScrollTop` or `res.resetScroll`

9. **Accessibility Features**
   - As a user with screen reader, I want page changes to be announced
   - Automatic focus management after page updates
   - Support for custom focus via `res.focus`

10. **History Management**
    - As a user, I want back/forward buttons to work correctly
    - Support for `req.backButtonPressed` and `req.forwardButtonPressed`
    - Classes added to `<html>` element: `backButtonPressed`, `forwardButtonPressed`

11. **Loading Indicator**
    - As a developer, I want to show a loading bar during page transitions
    - Topbar integration with configuration options
    - Option to enable/disable for specific routes

12. **Middleware Support**
    - As a developer, I want to apply middleware to routes like Express
    - Support for middleware chains

13. **Custom Render Method**
    - As a developer, I want to provide my own render logic if the default doesn't suit my needs
    - Replace default DOM manipulation entirely

14. **Programmatic Route Triggering**
    - As a developer, I want to trigger routes programmatically via `app.triggerRoute()`
    - Support for specifying route, method, and body

15. **Cookie Management**
    - As a developer, I want to read and write cookies using Express-like methods
    - `res.cookie()` and `res.clearCookie()` with standard options

16. **Pre/Post Render Hooks**
    - As a developer, I want to execute code before and after DOM updates
    - `app.beforeEveryRender()`, `app.afterEveryRender()` for global hooks
    - `res.beforeRender()`, `res.afterRender()` for per-request hooks
    - `postRenderCallbacks` for per-template hooks

17. **Express Version Compatibility**
    - As a developer, I want to choose between Express 4 and 5 route syntax
    - Configurable via `expressVersion` parameter

### New Features to Add (Backend Migration)

18. **FastAPI Backend Server**
    - As a backend developer, I want a Python/FastAPI server that can serve templates and handle HTTP requests
    - FastAPI application with route handlers
    - Template rendering capability (Jinja2 or custom)

19. **Isomorphic Route Sharing**
    - As a developer, I want to define routes once and use them in both Python backend and JavaScript frontend
    - Route definition format consumable by both Python and JavaScript
    - Option A: YAML/JSON route configuration file parsed by both
    - Option B: JavaScript routes file with Python wrapper that loads and interprets it
    - Option C: Code generation tool to translate routes between languages

20. **Isomorphic Template Sharing**
    - As a developer, I want to use the same HTML templates in both Python backend and JavaScript frontend
    - Option A: Use JavaScript-based templating engine (Teddy) on both sides
    - Option B: Use Jinja2 templates with client-side Jinja2 implementation
    - Option C: Template compilation to share compiled templates
    - Template bundling for client-side delivery

21. **Data Fetching Integration**
    - As a developer, I want frontend routes to fetch data from FastAPI backend
    - Async model loading functions
    - API endpoints for data retrieval
    - Consistent data models between server and client

22. **FastAPI Sample Applications**
    - As a developer exploring the library, I want sample apps demonstrating FastAPI integration
    - Basic frontend-only sample (unchanged)
    - Frontend with templating sample (unchanged)
    - FastAPI integration sample (replaces Express sample)
    - FastAPI complex integration sample (replaces Express complex sample)

23. **Type Hints (Optional)**
    - As a TypeScript developer, I want type definitions for the frontend library
    - TypeScript definitions for `single-page-express`

24. **Documentation Updates**
    - As a new user, I want documentation explaining FastAPI integration
    - Migration guide from Express to FastAPI backend
    - Updated sample app documentation

---

## Technical Requirements and Constraints

### Frontend Requirements

1. **No Breaking Changes to Public API**
   - All existing public methods, properties, and behaviors must remain compatible
   - Existing applications using `single-page-express` must continue to work without modification

2. **Browser Compatibility**
   - Continue support for modern browsers with View Transitions API
   - Graceful degradation for browsers without View Transitions support
   - No increase in minimum browser requirements

3. **Bundle Size**
   - Maintain small bundle size (tens of KB, not hundreds or thousands of KB)
   - No significant increase in bundle size from current implementation

4. **Performance**
   - Page transitions must remain fast and smooth
   - No regressions in navigation performance
   - Efficient DOM updates

5. **Accessibility**
   - Maintain WCAG compliance for all features
   - Screen reader announcements must continue to work
   - Focus management must remain functional

### Backend Requirements (FastAPI)

1. **Python Version**
   - Support Python 3.8+ (FastAPI minimum requirement)

2. **FastAPI Version**
   - Use latest stable FastAPI version

3. **Template Engine**
   - Must be compatible with frontend template rendering
   - Options to evaluate:
     - Jinja2 (standard FastAPI choice) with client-side Jinja2
     - Keep Teddy (JavaScript) with Python integration via subprocess or embedding
     - Custom JavaScript template engine with Python bridge

4. **Route Compatibility**
   - Must support Express-like route patterns:
     - Path parameters: `/users/:id`
     - Query strings: `?key=value`
     - Wildcards: `*all` (Express 5) or `*` (Express 4)
   - May require custom route matching logic or Express compatibility layer

5. **Static File Serving**
   - Serve bundled frontend JavaScript
   - Serve templates
   - Serve CSS and other static assets

6. **Form Handling**
   - Parse form submissions
   - Support for application/x-www-form-urlencoded
   - Support for multipart/form-data

7. **JSON API**
   - Provide JSON endpoints for data fetching
   - CORS support for cross-origin requests (if needed)

8. **Error Handling**
   - Consistent error handling between frontend and backend
   - Proper HTTP status codes

### Shared Requirements

1. **Route Definition Format**
   - Define a route configuration format that works for both Python and JavaScript
   - Consider: YAML, JSON, or DSL that both languages can parse

2. **Template Sharing Strategy**
   - Choose and implement one of the template sharing options
   - Ensure templates work identically on both server and client

3. **Development Experience**
   - Hot reloading during development (both frontend and backend)
   - Clear error messages
   - Debugging support

4. **Testing**
   - All existing Playwright tests must pass
   - New tests for FastAPI backend
   - Integration tests for full isomorphic flow

5. **Documentation**
   - Update README with FastAPI integration instructions
   - Create migration guide from Express to FastAPI
   - Document route and template sharing approach

---

## Tech Stack

### Current Stack

**Frontend:**
- JavaScript (ES6+)
- Webpack (bundler)
- Playwright (testing)
- Standard.js (linting)
- Teddy (templating in samples)
- Topbar (loading indicator)
- path-to-regexp (route parsing)

**Backend (To Be Replaced):**
- Node.js
- Express.js
- Teddy (templating)

### Target Stack

**Frontend (Preserved):**
- JavaScript (ES6+)
- Webpack (bundler) - unchanged
- Playwright (testing) - unchanged
- Standard.js (linting) - unchanged
- Topbar (loading indicator) - unchanged
- path-to-regexp (route parsing) - unchanged
- Template engine - TBD (see Requirements)

**Backend (New):**
- Python 3.8+
- FastAPI (latest stable)
- Uvicorn (ASGI server)
- Template engine - TBD (Jinja2 or alternative)
- Pytest or unittest (testing)
- Black or Ruff (linting/formatting)
- Poetry or pip-tools (dependency management)

---

## Scale and Deployment Considerations

1. **Frontend Bundle Distribution**
   - Continue publishing to npm
   - Maintain multiple output formats (ESM, CommonJS, UMD)
   - No changes to npm package structure

2. **Backend Deployment**
   - Standard Python/FastAPI deployment patterns
   - Docker support (new Dockerfile for FastAPI backend)
   - WSGI/ASGI server (Uvicorn)

3. **Development Workflow**
   - Separate frontend and backend development servers
   - Proxy configuration for API calls during development
   - Hot reload for both frontend and backend

4. **Performance**
   - FastAPI should be performant (async/await, uvicorn)
   - No performance regressions in frontend
   - Optimize template loading and rendering

5. **Monitoring and Logging**
   - FastAPI logging
   - Error tracking integration
   - Performance monitoring (optional)

---

## Integrations and External Dependencies

### Frontend (Preserved)

1. **Path-to-RegExp**
   - `path-to-regexp` (Express 5+)
   - `path-to-regexp-express4` (Express 4 compatibility)

2. **Topbar**
   - Loading progress bar
   - No changes needed

### Backend (New - FastAPI)

1. **FastAPI**
   - Core web framework
   - Type hints support
   - Automatic OpenAPI documentation

2. **Uvicorn**
   - ASGI server
   - Hot reload support

3. **Template Engine** (To Be Determined)
   - Option A: Jinja2
     - Python standard
     - Client-side: `jinja-js` or similar
   - Option B: Keep Teddy
     - JavaScript-based
     - Python integration via subprocess or JavaScript engine (PyExecJS or similar)
   - Option C: Custom solution

4. **Optional Integrations**
   - `python-multipart` (for file uploads)
   - `python-jose` (JWT, if auth needed later)
   - `pydantic` (data validation)

---

## Authentication / Authorization

### Current State

No authentication/authorization features in the base library. The library provides:
- `res.cookie()` and `res.clearCookie()` for manual cookie management
- `req.cookies` for reading cookies
- Stubbed out `req.signedCookies` (not supported in browser context)

### Target State

No authentication/authorization changes required. The library will continue to:
- Support manual cookie management
- Not include built-in auth (as it's a routing library, not a full framework)

If future authentication is needed, it would be implemented as:
- User-supplied middleware on routes
- Custom integration with auth providers
- Outside the scope of this migration

---

## Migration Approach and Considerations

### Route Definition Format

**Challenge:** How to share routes between Python and JavaScript?

**Options to Evaluate:**

1. **YAML Configuration**
   ```yaml
   routes:
     - method: GET
       path: /
       template: index
     - method: GET
       path: /users/:id
       template: userDetail
       handler: getUserData
   ```
   - Pros: Language-agnostic, human-readable
   - Cons: Limited flexibility, may not support complex handler logic

2. **JSON Configuration**
   ```json
   {
     "routes": [
       {"method": "GET", "path": "/", "template": "index"},
       {"method": "GET", "path": "/users/:id", "template": "userDetail", "handler": "getUserData"}
     ]
   }
   ```
   - Pros: Easy to parse in both languages
   - Cons: Same limitations as YAML

3. **JavaScript Routes with Python Bridge**
   - Keep `routes.js` file for frontend
   - Python subprocess to load and execute JavaScript routes
   - Or use a JavaScript engine in Python (PyExecJS, V8 wrapper)
   - Pros: Full JavaScript flexibility preserved
   - Cons: Performance overhead, complexity

4. **Code Generation**
   - Define routes in Python, generate JavaScript
   - Or define in JavaScript, generate Python
   - Pros: Optimized for each language
   - Cons: Two codebases to maintain, generation step required

**Recommended Approach for MVP:**
Start with **Option 1 (YAML)** for simple route definitions, with fallback to language-specific handlers for complex logic. This provides:
- Clear route definition format
- Easy sharing between languages
- Separation of concerns (routing vs. business logic)

### Template Sharing Strategy

**Challenge:** How to share templates between Python and JavaScript?

**Options to Evaluate:**

1. **Jinja2 Templates with Client-Side Jinja2**
   - Use Jinja2 templates on Python backend
   - Use `jinja-js` or similar on JavaScript frontend
   - Pros: Standard Python stack, good tooling
   - Cons: Jinja2 and client-side Jinja2 may have subtle differences

2. **Keep Teddy with Python Integration**
   - Continue using Teddy templates
   - Python integrates Teddy via subprocess or JavaScript engine
   - Pros: No template changes, proven approach
   - Cons: Python integration complexity

3. **Template Compilation**
   - Pre-compile templates to a shared intermediate format
   - Both Python and JavaScript load and execute compiled templates
   - Pros: Best of both worlds
   - Cons: Complex to implement, may not be possible for all engines

**Recommended Approach for MVP:**
Start with **Option 2 (Keep Teddy with Python Integration)** to minimize changes and ensure compatibility. Use a lightweight JavaScript engine in Python (PyExecJS or similar) to render Teddy templates server-side.

### Development Workflow

**Proposed Workflow:**

1. **Frontend Development:**
   ```bash
   npm run build  # Build single-page-express
   npm run sample-app-basic-frontend-only  # Run frontend-only sample
   ```

2. **Backend Development:**
   ```bash
   cd sampleApps/fastapi
   pip install -r requirements.txt
   uvicorn server:app --reload
   ```

3. **Full Stack Development:**
   ```bash
   # Terminal 1: Backend
   cd sampleApps/fastapi
   uvicorn server:app --reload --port 3000

   # Terminal 2: Frontend (if separate dev server needed)
   npm run dev
   ```

4. **Testing:**
   ```bash
   # Frontend tests
   npm test

   # Backend tests (new)
   pytest sampleApps/fastapi/tests/

   # Integration tests
   npm run test-integration
   ```

---

## Success Criteria

1. **All existing Playwright tests pass** with the new backend
2. **Frontend API remains unchanged** - no breaking changes
3. **Sample apps work identically** with FastAPI backend as they did with Express
4. **Bundle size does not increase significantly**
5. **Performance is maintained** - no regressions in navigation speed
6. **Documentation is complete** - users can understand how to use FastAPI backend
7. **Route and template sharing works** - isomorphic architecture achieved
8. **Development workflow is smooth** - hot reload, clear errors

---

## Next Steps / Implementation Phases

### Phase 1: Foundation
- Set up FastAPI project structure
- Create basic FastAPI server with static file serving
- Verify frontend bundle loads correctly

### Phase 2: Route Sharing
- Define route configuration format (YAML)
- Implement route parser for both Python and JavaScript
- Create route registration in FastAPI

### Phase 3: Template Sharing
- Choose and implement template sharing strategy
- Implement template bundling for client-side
- Integrate template rendering in FastAPI

### Phase 4: Feature Parity
- Implement all Express features in FastAPI
- Form handling, query params, path params, cookies
- Data fetching and async model loading

### Phase 5: Sample Apps
- Replace Express samples with FastAPI equivalents
- Verify all sample apps work correctly

### Phase 6: Testing and Documentation
- Run all Playwright tests against new backend
- Create FastAPI-specific tests
- Write migration guide and update documentation

### Phase 7: Optimization and Polish
- Performance tuning
- Error handling improvements
- Final documentation review

---

## Risks and Mitigation

| Risk | Impact | Mitigation |
|------|---------|------------|
| Route pattern differences between Express and FastAPI | High | Implement custom route matcher or compatibility layer |
| Template engine incompatibility | High | Evaluate multiple options early, choose most compatible |
| Performance regression | Medium | Benchmark current implementation, compare after migration |
| Breaking changes to frontend API | High | Strict compatibility testing, version management |
| Complexity of Python/JavaScript integration | Medium | Choose simplest viable approach, document clearly |
| Maintaining two backend implementations | Low | Deprecate Express backend over time, provide migration path |

---

## Open Questions for Technical Discussion

1. **Route Definition Format:** Should we start with YAML configuration, or is there a preferred format?
2. **Template Engine:** Do you have preference for Jinja2 vs. keeping Teddy vs. other options?
3. **Python/JavaScript Integration:** Any preference on approach (subprocess, JS engine, or code generation)?
4. **TypeScript:** Should we add TypeScript definitions for the frontend library?
5. **Express Deprecation:** Should we deprecate the Express backend immediately or maintain it for a transition period?

---

**Document Version:** 1.0
**Last Updated:** 2026-03-08
**Status:** Ready for Review
