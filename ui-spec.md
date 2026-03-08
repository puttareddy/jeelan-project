# UI Specification

Front-end contract: screens, design tokens, wireframes, state, accessibility, theming, and loading/error/empty states.

## Screens

### 1. Homepage (`/`)
- **Route:** `GET /`
- **Template:** `index.html`
- **Purpose:** Landing page demonstrating basic routing functionality with "hello world" content
- **Location:** `/sampleApps/{express|express-complex|fastapi|fastapi-complex}/mvc/views/index.html`

### 2. Second Page (`/secondPage`)
- **Route:** `GET /secondPage`
- **Template:** `secondPage.html`
- **Purpose:** Demonstrates variable passing from route handlers to templates
- **Model:** `{ variable: 'variable with contents: "hi there!"' }`
- **Location:** `/sampleApps/{express|express-complex|fastapi|fastapi-complex}/mvc/views/secondPage.html`

### 3. Page with Form (`/pageWithForm`)
- **Route:** `GET /pageWithForm`, `POST /pageWithForm`
- **Template:** `pageWithForm.html`
- **Purpose:** Demonstrates form submission handling and data echoing
- **Model:** Displays submitted form data on POST response
- **Location:** `/sampleApps/{express|express-complex|fastapi|fastapi-complex}/mvc/views/pageWithForm.html`

### 4. Page with Data Retrieval (`/pageWithDataRetrieval`)
- **Route:** `GET /pageWithDataRetrieval`
- **Template:** `pageWithDataRetrieval.html`
- **Purpose:** Demonstrates async model loading from server
- **Model:** `{ randomNumber: <async value 1-10> }`
- **Location:** `/sampleApps/{express|express-complex|fastapi|fastapi-complex}/mvc/views/pageWithDataRetrieval.html`

### 5. Page with Route Parameters (`/route/:withParam/:anotherParam`)
- **Route:** `GET /route/:withParam/:anotherParam`
- **Template:** `params.html`
- **Purpose:** Demonstrates dynamic route parameter handling
- **Model:** `{ withParam: <param1>, anotherParam: <param2> }`
- **Location:** `/sampleApps/express-complex|fastapi-complex/mvc/views/params.html`

### 6. Page with Title Override (`/pageWithTitleOverride`)
- **Route:** `GET /pageWithTitleOverride`
- **Template:** Uses `secondPage.html` template with `res.title` override
- **Purpose:** Demonstrates `res.title` property for page title override
- **Location:** Reuses `secondPage.html` template

### 7. Verbose Page (`/verbosePage`)
- **Route:** `GET /verbosePage`
- **Template:** `verbosePage.html`
- **Purpose:** Demonstrates scroll position memory with long content
- **Features:** Contains scrolling container (`#scrolling-container`) to test scroll restoration
- **Location:** `/sampleApps/express-complex|fastapi-complex/mvc/views/verbosePage.html`

### 8. Page with Additional Head Tags (`/additionalHeadTags`)
- **Route:** `GET /additionalHeadTags`
- **Template:** `additionalHeadTags.html`
- **Purpose:** Demonstrates dynamic head tag management (meta, link, script)
- **Features:** Introduces new `<meta>`, `<link>`, and `<script>` tags
- **Location:** `/sampleApps/express-complex|fastapi-complex/mvc/views/additionalHeadTags.html`

### 9. Partial Template Route (`/routeUsingTarget`)
- **Route:** `GET /routeUsingTarget`
- **Template:** `partial.html`
- **Purpose:** Demonstrates partial template rendering without layout
- **Target:** Updates `body > article > *:first-child` element only
- **Location:** `/sampleApps/express-complex|fastapi-complex/mvc/views/partial.html`

## Design Tokens

### Color Palette
```css
:root {
  /* Navigation Colors */
  --nav-bg: #ccc;
  --nav-border: #666;

  /* Background Colors */
  --bg-default: #ffffff;
  --bg-scrolling: #c0c0c0;

  /* View Transition Colors */
  --transition-color-start: rgba(0, 0, 0, .7);
  --transition-color-end: rgba(0, 0, 0, .7);

  /* Topbar Colors (configurable via app.topbarConfig) */
  --topbar-color-0: rgba(0, 0, 0, .7);
  --topbar-color-100: rgba(0, 0, 0, .7);
}
```

### Typography
```css
:root {
  --font-family-base: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  --font-size-base: 16px;
  --line-height-base: 1.5;
  --font-weight-regular: 400;
  --font-weight-bold: 700;
}
```

### Spacing Scale
```css
:root {
  --spacing-xs: 4px;
  --spacing-sm: 8px;
  --spacing-md: 16px;
  --spacing-lg: 24px;
  --spacing-xl: 32px;
}
```

### Border Radius
```css
:root {
  --radius-sm: 2px;
  --radius-md: 4px;
  --radius-lg: 8px;
}
```

### Shadows
```css
:root {
  --shadow-sm: 0 1px 2px rgba(0, 0, 0, 0.1);
  --shadow-md: 0 2px 4px rgba(0, 0, 0, 0.15);
  --shadow-lg: 0 4px 8px rgba(0, 0, 0, 0.2);
}
```

### View Transitions (CSS Variables)
```css
:root {
  --fade-out: 90ms cubic-bezier(0.4, 0, 1, 1) both fade-out;
  --fade-in: 210ms cubic-bezier(0, 0, 0.2, 1) 90ms both fade-in;
  --slide-to-left: 300ms cubic-bezier(0.4, 0, 0.2, 1) both slide-to-left;
  --slide-from-right: 300ms cubic-bezier(0.4, 0, 0.2, 1) both slide-from-right;
  --slide-to-right: 300ms cubic-bezier(0.4, 0, 0.2, 1) both slide-to-right;
  --slide-from-left: 300ms cubic-bezier(0.4, 0, 0.2, 1) both slide-from-left;
}
```

### Z-Index Layering
```css
:root {
  --z-live-region: -9999;
  --z-content: 1;
  --z-nav: 2;
}
```

## ASCII Wireframes

### Homepage (`/`)
```
┌────────────────────────────────────────────────────────────────────────────────────┐
│ Header                                                                    │
│ ┌──────────────────────────────────────────────────────────────────────────┐   │
│ │ Sample app for single-page-express                                │   │
│ │ — Homepage                                                            │   │
│ └──────────────────────────────────────────────────────────────────────────┘   │
│                                                                          │
│ Open browser console to see logs that illustrate how single page app       │
│ is handling navigation events.                                             │
│                                                                          │
│ Navigation                                                                │
│ ┌──────────────────────────────────────────────────────────────────────────┐   │
│ │ Homepage | Second page | Page with form | Page with data...    │   │
│ │ Route targeting... | Route with params | Verbose page | Additional   │   │
│ │ head tags                                                             │   │
│ └──────────────────────────────────────────────────────────────────────────┘   │
│                                                                          │
│ ┌──────────────────────────────────────────────────────────────────────────┐   │
│ │ hello world                                                           │   │
│ └──────────────────────────────────────────────────────────────────────────┘   │
│                                                                          │
└────────────────────────────────────────────────────────────────────────────────────┘
```

### Second Page (`/secondPage`)
```
┌────────────────────────────────────────────────────────────────────────────────────┐
│ Header                                                                    │
│ ┌──────────────────────────────────────────────────────────────────────────┐   │
│ │ Sample app for single-page-express                                │   │
│ │ — Second page                                                         │   │
│ └──────────────────────────────────────────────────────────────────────────┘   │
│                                                                          │
│ Navigation                                                                │
│ ┌──────────────────────────────────────────────────────────────────────────┐   │
│ │ Homepage | Second page | Page with form | Page with data...    │   │
│ │ Route targeting... | Route with params | Verbose page | Additional   │   │
│ │ head tags                                                             │   │
│ └──────────────────────────────────────────────────────────────────────────┘   │
│                                                                          │
│ ┌──────────────────────────────────────────────────────────────────────────┐   │
│ │ this is a second page and prints a variable with contents: "hi there!" │   │
│ └──────────────────────────────────────────────────────────────────────────┘   │
│                                                                          │
└────────────────────────────────────────────────────────────────────────────────────┘
```

### Page with Form (`/pageWithForm`)
```
┌────────────────────────────────────────────────────────────────────────────────────┐
│ Header                                                                    │
│ ┌──────────────────────────────────────────────────────────────────────────┐   │
│ │ Sample app for single-page-express                                │   │
│ │ — Page with form                                                      │   │
│ └──────────────────────────────────────────────────────────────────────────┘   │
│                                                                          │
│ Navigation                                                                │
│ ┌──────────────────────────────────────────────────────────────────────────┐   │
│ │ Homepage | Second page | Page with form | Page with data...    │   │
│ │ Route targeting... | Route with params | Verbose page | Additional   │   │
│ │ head tags                                                             │   │
│ └──────────────────────────────────────────────────────────────────────────┘   │
│                                                                          │
│ ┌──────────────────────────────────────────────────────────────────────────┐   │
│ │ Form                                                                   │   │
│ │ ┌──────────────────────────────────────────────────────────────┐     │   │
│ │ │ First name:                                                 │     │   │
│ │ │ [________________]                                       │     │   │
│ │ │                                                             │     │   │
│ │ │ [ Button 1 ]  [ Button 2 ]                              │     │   │
│ │ └──────────────────────────────────────────────────────────────┘     │   │
│ │                                                                  │   │
│ │ (POST to /pageWithForm)                                       │   │
│ └──────────────────────────────────────────────────────────────────────────┘   │
│                                                                          │
└────────────────────────────────────────────────────────────────────────────────────┘
```

### Page with Data Retrieval (`/pageWithDataRetrieval`)
```
┌────────────────────────────────────────────────────────────────────────────────────┐
│ Header                                                                    │
│ ┌──────────────────────────────────────────────────────────────────────────┐   │
│ │ Sample app for single-page-express                                │   │
│ │ — Page with data retrieval                                            │   │
│ └──────────────────────────────────────────────────────────────────────────┘   │
│                                                                          │
│ Navigation                                                                │
│ ┌──────────────────────────────────────────────────────────────────────────┐   │
│ │ Homepage | Second page | Page with form | Page with data...    │   │
│ │ Route targeting... | Route with params | Verbose page | Additional   │   │
│ │ head tags                                                             │   │
│ └──────────────────────────────────────────────────────────────────────────┘   │
│                                                                          │
│ ┌──────────────────────────────────────────────────────────────────────────┐   │
│ │ Random number fetched from server: [7]                            │   │
│ │ (async model loading)                                             │   │
│ └──────────────────────────────────────────────────────────────────────────┘   │
│                                                                          │
└────────────────────────────────────────────────────────────────────────────────────┘
```

### Page with Route Parameters (`/route/:withParam/:anotherParam`)
```
┌────────────────────────────────────────────────────────────────────────────────────┐
│ Header                                                                    │
│ ┌──────────────────────────────────────────────────────────────────────────┐   │
│ │ Sample app for single-page-express                                │   │
│ │ — Page with params                                                   │   │
│ └──────────────────────────────────────────────────────────────────────────┘   │
│                                                                          │
│ Navigation                                                                │
│ ┌──────────────────────────────────────────────────────────────────────────┐   │
│ │ Homepage | Second page | Page with form | Page with data...    │   │
│ │ Route targeting... | Route with params | Verbose page | Additional   │   │
│ │ head tags                                                             │   │
│ └──────────────────────────────────────────────────────────────────────────┘   │
│                                                                          │
│ ┌──────────────────────────────────────────────────────────────────────────┐   │
│ │ withParam: [value1]                                            │   │
│ │ anotherParam: [value2]                                         │   │
│ │ (from /route/value1/value2)                                    │   │
│ └──────────────────────────────────────────────────────────────────────────┘   │
│                                                                          │
└────────────────────────────────────────────────────────────────────────────────────┘
```

### Verbose Page (`/verbosePage`)
```
┌────────────────────────────────────────────────────────────────────────────────────┐
│ Header                                                                    │
│ ┌──────────────────────────────────────────────────────────────────────────┐   │
│ │ Sample app for single-page-express                                │   │
│ │ — Verbose page                                                       │   │
│ └──────────────────────────────────────────────────────────────────────────┘   │
│                                                                          │
│ Navigation                                                                │
│ ┌──────────────────────────────────────────────────────────────────────────┐   │
│ │ Homepage | Second page | Page with form | Page with data...    │   │
│ │ Route targeting... | Route with params | Verbose page | Additional   │   │
│ │ head tags                                                             │   │
│ └──────────────────────────────────────────────────────────────────────────┘   │
│                                                                          │
│ ┌──────────────────────────────────────────────────────────────────────────┐   │
│ │ Lorem ipsum dolor sit amet...                                          │   │
│ │ [Long content - multiple paragraphs]                                  │   │
│ │ ...                                                                │   │
│ └──────────────────────────────────────────────────────────────────────────┘   │
│                                                                          │
│ ┌──────────────────────────────────────────────────────────────────────────┐   │
│ │ Scrolling Container (scroll test area)                             │   │
│ │ ┌────────────────────────────────────────────────────────────┐     │   │
│ │ │ Praesent at est at metus congue...                [scroll]│     │   │
│ │ │ Cras posuere magna ut elit dictum...                     │     │   │
│ │ │ Curabitur non convallis justo...                            │     │   │
│ │ │ [More scrollable content...]                              │     │   │
│ │ └────────────────────────────────────────────────────────────┘     │   │
│ └──────────────────────────────────────────────────────────────────────────┘   │
│                                                                          │
└────────────────────────────────────────────────────────────────────────────────────┘
```

### Page with Additional Head Tags (`/additionalHeadTags`)
```
┌────────────────────────────────────────────────────────────────────────────────────┐
│ Header                                                                    │
│ ┌──────────────────────────────────────────────────────────────────────────┐   │
│ │ Sample app for single-page-express                                │   │
│ │ — New meta tag test                                               │   │
│ └──────────────────────────────────────────────────────────────────────────┘   │
│                                                                          │
│ Navigation                                                                │
│ ┌──────────────────────────────────────────────────────────────────────────┐   │
│ │ Homepage | Second page | Page with form | Page with data...    │   │
│ │ Route targeting... | Route with params | Verbose page | Additional   │   │
│ │ head tags                                                             │   │
│ └──────────────────────────────────────────────────────────────────────────┘   │
│                                                                          │
│ ┌──────────────────────────────────────────────────────────────────────────┐   │
│ │ this page introduces a new meta tag                                │   │
│ │                                                                  │   │
│ │ Head elements added dynamically:                                      │   │
│ │ • <meta name="new-meta-tag" content="hello world">                │   │
│ │ • <link rel="stylesheet" href="/extra.css">                       │   │
│ │ • <script src="/extra.js"></script>                               │   │
│ └──────────────────────────────────────────────────────────────────────────┘   │
│                                                                          │
└────────────────────────────────────────────────────────────────────────────────────┘
```

### Partial Template Route (`/routeUsingTarget`)
```
┌────────────────────────────────────────────────────────────────────────────────────┐
│ Header (unchanged)                                                     │
│ ┌──────────────────────────────────────────────────────────────────────────┐   │
│ │ Sample app for single-page-express                                │   │
│ │ — Homepage (title not updated)                                      │   │
│ └──────────────────────────────────────────────────────────────────────────┘   │
│                                                                          │
│ Navigation (unchanged)                                                    │
│ ┌──────────────────────────────────────────────────────────────────────────┐   │
│ │ Homepage | Second page | Page with form | Page with data...    │   │
│ │ Route targeting... | Route with params | Verbose page | Additional   │   │
│ │ head tags                                                             │   │
│ └──────────────────────────────────────────────────────────────────────────┘   │
│                                                                          │
│ ┌──────────────────────────────────────────────────────────────────────────┐   │
│ │ partial template without layout (does not update page title)          │   │
│ │ (Only article > *:first-child is replaced)                        │   │
│ └──────────────────────────────────────────────────────────────────────────┘   │
│                                                                          │
└────────────────────────────────────────────────────────────────────────────────────┘
```

## State Architecture (Zustand or equivalent)

### Note
This is a client-side routing library, not a traditional application with state management. However, the library maintains internal state for navigation, scroll positions, and route handling.

### App Instance State
```javascript
const app = {
  // Configuration
  expressVersion: number,                    // Express API version (4 or 5)
  appVars: object,                          // Application settings via app.set()/app.get()
  templatingEngine: object,                   // Template engine instance
  templates: object,                        // Loaded templates keyed by template name
  defaultTarget: string,                    // CSS selector for default DOM update target
  defaultTargets: array,                    // Array of CSS selectors for multiple targets
  beforeEveryRender: function,               // Global pre-render hook
  updateDelay: number,                       // Delay before DOM update in milliseconds
  afterEveryRender: function,                // Global post-render hook
  postRenderCallbacks: object,              // Per-template post-render callbacks

  // Topbar Configuration
  topbarEnabled: boolean,                    // Enable/disable loading indicator
  topBarRoutes: array,                       // Routes to apply topbar to
  topbar: object,                           // Topbar instance

  // View Transitions
  alwaysSkipViewTransition: boolean,          // Never use View Transitions API

  // Scroll Management
  alwaysScrollTop: boolean,                  // Always scroll to top after render
  urls: object,                             // Visited URLs with scroll metadata
}
```

### URLs History State Structure
```javascript
app.urls = {
  '/path': {
    scrollX: number,                           // Horizontal scroll position
    scrollY: number,                           // Vertical scroll position
    scrollingChildContainers: {
      'containerId': {
        scrollX: number,
        scrollY: number
      }
    }
  }
}
```

### Request/Response Lifecycle State
```javascript
// Created per "request" (navigation event)
const req = {
  app: object,                              // Reference to app instance
  body: object | undefined,                // Form submission data or JSON body
  cookies: object,                            // Parsed cookies from document.cookie
  method: string,                           // HTTP method (GET, POST, etc.)
  originalUrl: string,                      // Original URL including query string
  params: object,                            // Route parameters from path pattern
  path: string,                             // URL pathname
  protocol: string,                          // URL protocol (http: or https:)
  query: object,                             // Query string parameters
  route: object,                             // Matched route information
  secure: boolean,                            // True if protocol is https:
  subdomains: array,                          // Parsed subdomains
  singlePageExpress: boolean,               // True indicates SPE context
  backButtonPressed: boolean,               // True if back button was pressed
  forwardButtonPressed: boolean             // True if forward button was pressed
  res: object                               // Reference to response object
}

const res = {
  app: object,                              // Reference to app instance
  req: object,                              // Reference to request object
  title: string | null,                     // Override page title
  target: string | null,                    // CSS selector for DOM update target
  focus: string | null,                     // CSS selector for focus element
  beforeRender: function | null,              // Execute before DOM update
  afterRender: function | null,               // Execute after DOM update
  removeMetaTags: boolean | null,           // Remove all meta tags from head
  removeStyleTags: boolean | null,          // Remove all style tags from head
  removeLinkTags: boolean | null,           // Remove all link tags from head
  removeScriptTags: boolean | null,         // Remove all script tags from head
  removeBaseTags: boolean | null,           // Remove all base tags from head
  removeTemplateTags: boolean | null,        // Remove all template tags from head
  removeHeadTags: boolean | null,           // Remove all non-title head tags
  skipViewTransition: boolean | null,        // Skip view transition animation
  updateDelay: number | null,                // Delay DOM update (milliseconds)
  resetScroll: boolean | null,               // Reset scroll position for this request
}
```

### Route Registration State
```javascript
app.routes = {
  'method#route': {
    method: string,        // HTTP method (GET, POST, etc.)
    route: string,          // Route path pattern (e.g., '/', '/users/:id')
    matcher: function        // Route matching function from path-to-regexp
  }
}

app.routeCallbacks = {
  'method#route': function(req, res)  // Route handler function
}
```

### History Stack (Internal)
```javascript
const historyStack = [
  { index: number }  // History state for back/forward detection
]
let currentIndex = -1  // Current position in history stack
```

### Current View Transition Reference
```javascript
let currentViewTransition  // Global reference to current view transition
                              // for knowing when transition has ended
```

### Live Region State (ARIA)
```javascript
// Created dynamically on first render if not present
const liveRegion = {
  id: 'singlePageExpressDefaultRenderMethodAriaLiveRegion',
  aria-live: 'assertive',
  aria-atomic: 'true',
  position: 'absolute',
  top: '-9999px',
  left: '-9999px',
  width: '1px',
  height: '1px',
  overflow: 'hidden',
  border: '0',
  margin: '-1px',
  padding: '0',
  clipPath: 'inset(50%)',
  whiteSpace: 'nowrap'
}
```

## WCAG 2.1 AA

### Contrast Requirements
- All text must have a minimum contrast ratio of 4.5:1 for normal text (18pt/24px) and 3:1 for large text (14pt/18px and bold, or 18pt/24px)
- **Current Implementation:** Default color scheme provides adequate contrast (black on white, dark gray nav on light gray background)

### Focus Visible
- All interactive elements must have visible focus indicator
- **Implementation:**
  - Browser default focus styles are preserved
  - Non-interactive elements that receive programmatic focus have outline removed via CSS
  - Valid interactive elements (`A`, `INPUT`, `TEXTAREA`, `SELECT`, `BUTTON`, `FIELDSET`) maintain visible focus

### Keyboard Navigation
- All functionality must be accessible via keyboard
- **Implementation:**
  - Navigation links are native `<a>` tags, fully keyboard accessible
  - Form elements are native inputs, fully keyboard accessible
  - Tab order follows DOM order
  - No custom key handlers that block native navigation

### Labels
- All form inputs must have associated labels
- **Implementation:**
  - Form page uses `<label for="first_name">` properly associated with `<input id="first_name">`

### Error Identification
- Errors must be identified and described to users
- **Implementation:**
  - Console logging provides error messages for developers
  - User-facing errors could be enhanced with error display (future enhancement)

### No Focus Trap
- Users must be able to navigate away from focusable elements
- **Implementation:**
  - No custom modal or focus trap patterns
  - Native browser navigation fully functional

### Skip Link
- Provide a method to skip repeated navigation
- **Implementation:**
  - Currently not implemented; could be added as future enhancement for accessibility

### Headings
- Headings must be in logical order
- **Implementation:**
  - Header uses `<h1>` for app title, `<h2>` for page title
  - Semantic heading structure maintained across all templates

### Reduced Motion
- Respect user's motion preferences
- **Implementation:**
  - View transitions use CSS animations that respect `prefers-reduced-motion` media query (browser-native)
  - Fade and slide animations are applied via CSS, can be disabled via user agent stylesheet

### ARIA Live Regions
- Page changes must be announced to screen readers
- **Implementation:**
  - Hidden `<p>` element with `aria-live="assertive"` and `aria-atomic="true"`
  - Content announcement triggered on each page render
  - Page title read from `[data-page-title]` attribute or `<h1>` or `<title>` tag

### ARIA Attributes
- **data-page-title:** Used for screen reader announcements
  ```html
  <header data-page-title>
    <h1>Sample app for single-page-express</h1>
    <p hidden>—</p> <!-- Dash separator for screen readers -->
    <h2>{pageTitle}</h2>
  </header>
  ```

### Semantic HTML
- Proper semantic elements used throughout
- `<header>`, `<nav>`, `<article>`, `<section>` as appropriate
- Lists use `<ul>` and `<menu>` elements
- Definitions use `<dl>`, `<dt>`, `<dd>` elements

## White-Label Theming

### CSS Variables for Brand Customization
```css
:root {
  /* Brand Colors */
  --brand-primary: #000000;
  --brand-secondary: #666666;
  --brand-accent: #4a90e2;

  /* Brand Logo */
  --brand-logo-url: url('/logo.png');

  /* Brand Name */
  --brand-name: 'single-page-express';
}
```

### Application of Brand Tokens
```css
/* Navigation Background */
nav {
  background-color: var(--brand-secondary, #ccc);
  border: 1px var(--brand-secondary, #666) solid;
}

/* View Transitions */
body > header > h2,
body > article {
  view-transition-name: page-content-slide;
  /* Brand-accent could be used for transition colors */
}

/* Page Title */
body > header > h1 {
  color: var(--brand-primary, #000);
}

/* Links */
nav a {
  color: var(--brand-primary, #000);
}
```

### No Hardcoded Product Name
- Application title should use `{pageTitle}` variable from templates
- Layout template uses `{pageTitle}` for dynamic page titles
- Header app name can be customized via `--brand-name` variable

### Example: Applying Custom Brand
```javascript
// In browser.js or index.js
const singlePageExpress = require('single-page-express')

// Set brand variables via inline styles
const brandStyles = document.createElement('style')
brandStyles.textContent = `
  :root {
    --brand-primary: #2563eb;
    --brand-secondary: #f3f4f6;
    --brand-accent: #10b981;
    --brand-logo-url: url('/custom-logo.svg');
    --brand-name: 'My Custom App';
  }
`
document.head.appendChild(brandStyles)

// Initialize app with brand-specific config
const app = singlePageExpress({
  // ... other options
})
```

## Error and Empty States

### Per-Screen Loading States

#### Loading Indicator (Topbar)
- **Purpose:** Show progress during page transitions
- **Implementation:** Topbar library integration
- **Configuration:**
  ```javascript
  const app = singlePageExpress({
    topbarEnabled: true,                        // Default: true
    topBarRoutes: ['/', '/pageWithDataRetrieval'],  // Routes to apply topbar
    topbarConfig: {
      barColors: {
        0: 'rgba(0, 0, 0, .7)',
        '1.0': 'rgba(0, 0, 0, .7)'
      }
    }
  })
  ```
- **Visual:** Progress bar appears at top of viewport during navigation

#### Async Data Loading
- **Screen:** `pageWithDataRetrieval`
- **State:** Fetching random number from server
- **Visual:** Existing content remains visible during fetch (non-blocking)

### Per-Screen Error States

#### Template Not Found
- **Condition:** Attempted to render template that doesn't exist
- **Implementation:** Console error logged
  ```javascript
  console.error(`single-page-express: attempted to render template which does not exist: ${template}`)
  ```
- **User Feedback:** Page content unchanged, error logged to console
- **Enhancement Opportunity:** Display error message in UI

#### Template Engine Error
- **Condition:** Template parsing/rendering fails
- **Implementation:** Console error with details
  ```javascript
  console.error('single-page-express: error parsing post-rendered template:', template)
  console.error(error.message)
  ```
- **User Feedback:** Page content unchanged, error logged to console

#### Route Not Found (404)
- **Condition:** Browser navigates to unregistered route
- **Implementation:** Let browser handle natively (falls through to browser 404)
- **Behavior:** Browser displays default "Page not found" error

#### Route Registration Error
- **Condition:** Route parsing fails (e.g., invalid wildcard syntax)
- **Implementation:** Console error logged
  ```javascript
  console.error(`single-page-express: failed to register route '${route}' because it could not be parsed.`)
  console.error('single-page-express: routes with * in them should be written like *all instead in Express 5+ syntax.')
  console.error(error)
  ```

### Per-Screen Empty States

#### No Templates Loaded
- **Condition:** App initialized without templates
- **Implementation:** Warning logged to console
  ```javascript
  console.warn('single-page-express: no templates are loaded; as such as default render method will just print template name and model to console.')
  ```
- **Render Behavior:** Logs template name and model to console instead of rendering

#### Empty Form Data
- **Condition:** Form submitted without data (Express 5+ behavior)
- **Implementation:** `req.body` set to `undefined`
- **Behavior:** Handler receives undefined body instead of empty object

#### Empty Query String
- **Condition:** URL with no query parameters
- **Implementation:** `req.query` set to empty object `{}`
- **Behavior:** Handler receives empty query object

### Global Error States

#### No Template Engine
- **Condition:** App initialized without template engine or engine without `render()` method
- **Implementation:** Error logged to console
  ```javascript
  console.error('single-page-express: no template engine is loaded or engine supplied does not have a render method; please use a templating engine that is compatible with Express')
  ```

#### Invalid Target Selector
- **Condition:** `res.target` specifies invalid CSS selector
- **Implementation:** Error logged to console
  ```javascript
  console.error(`single-page-express: invalid target supplied: ${target}`)
  ```

#### Invalid DOM Target
- **Condition:** Template has element with same ID as target container
- **Implementation:** Target container replaced with template's target contents
- **Behavior:** Graceful replacement, no error

#### JavaScript Error in Route Handler
- **Condition:** Route handler throws error
- **Implementation:** Error propagates to browser console
- **Behavior:** Navigation may be interrupted, page may not update

### Empty States for Sample Applications

#### Basic Frontend-Only
- **Location:** `/sampleApps/basicFrontendOnly/index.html`
- **Content:** Minimal "hello world" example
- **State:** No templates, minimal configuration

#### Basic Frontend-Only with Templating
- **Location:** `/sampleApps/basicFrontendOnlyWithTemplating/index.html`
- **Content:** Teddy templates with minimal setup
- **State:** Templates loaded via Teddy client-side

#### Form Submission Empty State
- **Screen:** `pageWithForm` on first visit (GET)
- **Content:** Empty form with no submitted data
- **Behavior:** Form displays input field without values

### Error Recovery Patterns

#### Console Logging
- All errors logged to console with prefix `single-page-express:`
- Errors include contextual information (template name, route, error message)

#### Graceful Degradation
- Template errors don't crash the application
- Route registration errors prevent registration but don't stop app initialization
- Invalid targets log error but continue operation

#### Development vs. Production
- Development mode shows warnings for unsupported features (e.g., signed cookies)
- Production mode silently ignores unsupported features

### Enhancement Opportunities

#### User-Facing Error Display
- Add error banner/modal to display template errors
- Show friendly message when route not found (custom 404 page)

#### Loading Skeletons
- Add skeleton screens for async data loading
- Show placeholder content during fetch operations

#### Empty State UI
- Add "no data" message for empty lists or missing content
- Illustrate empty states with helpful graphics or text
