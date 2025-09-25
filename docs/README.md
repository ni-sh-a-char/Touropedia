# Touropedia  
**A responsive travel website built with HTML, SCSS, CSS and JavaScript**  

---  

## Table of Contents  

| # | Section |
|---|---------|
| 1 | [Overview](#overview) |
| 2 | [Installation](#installation) |
| 3 | [Usage](#usage) |
| 4 | [Project Structure](#project-structure) |
| 5 | [API Documentation](#api-documentation) |
| 6 | [Examples](#examples) |
| 7 | [Testing & Linting](#testing--linting) |
| 8 | [Contributing](#contributing) |
| 9 | [License](#license) |

---  

## Overview  

Touropedia is a **responsive, single‑page travel guide** that showcases destinations, itineraries, and travel tips. The site is built with:

* **HTML5** – semantic markup for accessibility.  
* **SCSS** – modular, component‑based styling with variables, mixins, and a responsive grid.  
* **CSS** – compiled from SCSS, with a small reset and utility classes.  
* **JavaScript (ES6+)** – vanilla JS modules for data fetching, UI interactions, and client‑side routing.  

The repository is set up for **zero‑config development** using npm scripts, a live‑reload dev server, and a production build pipeline that minifies assets.

---  

## Installation  

> **Prerequisites**  
> * Node.js **≥ 18.x** (LTS)  
> * npm **≥ 9.x** (comes with Node)  

```bash
# 1️⃣ Clone the repo
git clone https://github.com/yourusername/Touropedia.git
cd Touropedia

# 2️⃣ Install dependencies
npm ci          # installs exact versions from package-lock.json
# or
npm install     # if you want to update the lockfile

# 3️⃣ (Optional) Install the live‑server globally for quick testing
npm i -g live-server
```

### Development dependencies  

| Package | Purpose |
|---------|---------|
| `sass` | Compiles SCSS → CSS |
| `postcss-cli` + `autoprefixer` | Adds vendor prefixes |
| `esbuild` | Fast bundling & minification of JS |
| `eslint` + `prettier` | Code quality & formatting |
| `browser-sync` | Live‑reload dev server |
| `jest` | Unit testing for JS utilities |
| `cypress` | End‑to‑end testing (optional) |

---  

## Usage  

### Development (watch & hot‑reload)

```bash
npm run dev
```

* Compiles `src/scss/**/*.scss` → `dist/css/style.css` (with source maps).  
* Bundles `src/js/**/*.js` → `dist/js/bundle.js`.  
* Starts **BrowserSync** on `http://localhost:3000` with live‑reload on file changes.  

### Production Build  

```bash
npm run build
```

* SCSS → minified CSS (`dist/css/style.min.css`).  
* JS → minified bundle (`dist/js/bundle.min.js`).  
* Copies static assets (`images/`, `fonts/`, `favicon.ico`) to `dist/`.  
* Generates a **hash‑based cache‑busting** filename for CSS/JS (e.g., `style.3f2a1c.css`).  

### Serve the built site  

```bash
npm run serve
# → http://localhost:5000
```

> **Tip:** Deploy the contents of the `dist/` folder to any static‑host (GitHub Pages, Netlify, Vercel, Firebase Hosting, etc.).  

---  

## Project Structure  

```
Touropedia/
├─ .github/                # CI workflows, issue templates
├─ src/
│   ├─ index.html          # Entry point (HTML5)
│   ├─ scss/
│   │   ├─ base/           # Reset, typography, variables
│   │   ├─ components/     # Buttons, cards, nav, modal, etc.
│   │   ├─ layout/         # Grid, header, footer, utilities
│   │   └─ main.scss       # Imports all partials → compiled CSS
│   ├─ js/
│   │   ├─ api.js          # Wrapper around the public travel API
│   │   ├─ router.js       # Simple client‑side router (hash‑based)
│   │   ├─ ui/
│   │   │   ├─ modal.js
│   │   │   └─ carousel.js
│   │   └─ app.js          # App bootstrap
│   └─ assets/
│       ├─ images/
│       └─ fonts/
├─ dist/                   # Generated files (git‑ignored)
├─ tests/
│   ├─ unit/
│   └─ e2e/
├─ .eslintrc.json
├─ .prettierrc
├─ package.json
└─ README.md               # (this file)
```

---  

## API Documentation  

Touropedia consumes a **public travel‑information API** (the demo uses the free `https://api.touropedia.dev`). All network calls are encapsulated in `src/js/api.js`. Below is the public surface of the module.

### `api.js` (ES6 module)

```js
/**
 * @module api
 * @description Helper functions to fetch travel data.
 */

const BASE_URL = 'https://api.touropedia.dev/v1';

/**
 * Generic GET request.
 *
 * @param {string} endpoint - API endpoint (e.g., '/destinations')
 * @param {Object} [params={}] - Query parameters as key/value pairs
 * @returns {Promise<any>} Resolves with JSON payload or rejects with an Error.
 */
export async function get(endpoint, params = {}) {
  const url = new URL(`${BASE_URL}${endpoint}`);
  Object.entries(params).forEach(([k, v]) => url.searchParams.append(k, v));

  const response = await fetch(url, {
    method: 'GET',
    headers: { 'Accept': 'application/json' },
  });

  if (!response.ok) {
    const err = await response.json();
    throw new Error(err.message || 'API request failed');
  }
  return response.json();
}

/**
 * Fetch a list of destinations.
 *
 * @param {Object} [options] - Pagination & filter options
 * @param {number} [options.page=1]
 * @param {number} [options.limit=12]
 * @param {string} [options.country] - Optional country filter
 * @returns {Promise<{data: Destination[], meta: PaginationMeta}>}
 */
export function getDestinations(options = {}) {
  return get('/destinations', options);
}

/**
 * Fetch details for a single destination.
 *
 * @param {string|number} id - Destination identifier
 * @returns {Promise<Destination>}
 */
export function getDestinationById(id) {
  return get(`/destinations/${id}`);
}

/**
 * Search destinations by keyword.
 *
 * @param {string} query - Search term
 * @param {Object} [options] - Pagination options
 * @returns {Promise<{data: Destination[], meta: PaginationMeta}>}
 */
export function searchDestinations(query, options = {}) {
  return get('/search', { q: query, ...options });
}

/**
 * Types (for documentation / TypeScript users)
 *
 * @typedef {Object} Destination
 * @property {string|number} id
 * @property {string} name
 * @property {string} country
 * @property {string} description
 * @property {string[]} images
 * @property {number} rating
 *
 * @typedef {Object} PaginationMeta
 * @property {number} page
 * @property {number} limit
 * @property {number} total
 */
```

### Error handling  

All API functions **reject** with a native `Error`. The UI layer (`app.js`) catches these errors and displays a toast notification:

```js
import * as API from './api.js';
import { showToast } from './ui/modal.js';

async function loadHome() {
  try {
    const { data } = await API.getDestinations({ limit: 8 });
    renderCards(data);
  } catch (err) {
    showToast('⚠️ Unable to load destinations', { type: 'error' });
    console.error(err);
  }
}
```

---  

## Examples  

### 1️⃣ Rendering a Destination Card  

```js
import { createElement } from './utils/dom.js';

/**
 * @param {Destination} dest
 * @returns {HTMLElement}
 */
export function renderDestinationCard(dest) {
  const card = createElement('article', { class: 'card'