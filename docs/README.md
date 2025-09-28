# Touropedia  
**A responsive travel website built with HTML, SCSS, CSS, and JavaScript**  

---  

## Table of Contents  

1. [Overview](#overview)  
2. [Installation](#installation)  
3. [Usage](#usage)  
4. [API Documentation](#api-documentation)  
5. [Examples](#examples)  
6. [Development Scripts](#development-scripts)  
7. [Contributing](#contributing)  
8. [License](#license)  

---  

## Overview  

Touropedia is a modern, mobile‑first travel portal that showcases destinations, itineraries, and travel tips. The project is **purely front‑end** – all pages are static HTML rendered with SCSS‑generated CSS and interactive behavior powered by vanilla JavaScript (ES6+).  

Key features  

| Feature | Description |
|---------|-------------|
| **Responsive layout** | Fluid grid, breakpoints for mobile, tablet, desktop. |
| **SCSS architecture** | BEM‑styled components, variables, mixins, and a modular folder structure. |
| **Dynamic UI** | Lazy‑loaded images, smooth scroll, modal dialogs, and a client‑side search. |
| **Data‑driven** | JSON files (`data/`) feed the UI – easy to replace with a real API later. |
| **Zero‑runtime dependencies** | No frameworks, no build‑time bundlers required for production. |

---  

## Installation  

> **Prerequisites**  
> - Node.js (>= 18) – only needed for development tooling (SCSS compilation, linting, etc.).  
> - Git – to clone the repository.  

### 1. Clone the repo  

```bash
git clone https://github.com/your-username/Touropedia.git
cd Touropedia
```

### 2. Install development dependencies  

```bash
npm ci
```

> `npm ci` installs the exact versions listed in `package-lock.json`.  

### 3. Build the assets (optional – for production)  

```bash
npm run build
```

- Compiles SCSS → `dist/css/style.min.css` (autoprefixed, minified).  
- Copies static assets (`images/`, `fonts/`, `data/`) to `dist/`.  

### 4. Serve the site locally  

```bash
npm start
```

- Starts a lightweight development server (`http://localhost:3000`) with live‑reload.  

> **Note:** For a pure static deployment (GitHub Pages, Netlify, Vercel, etc.) you can simply push the contents of the `dist/` folder. No server‑side runtime is required.  

---  

## Usage  

### Running the site locally  

```bash
npm start
```

- The server watches for changes in `src/` (SCSS, JS, HTML) and automatically rebuilds.  

### Production deployment  

1. Build the assets: `npm run build`.  
2. Upload the `dist/` folder to any static‑hosting provider (e.g., GitHub Pages, Netlify, Vercel, AWS S3 + CloudFront).  

### Customising the theme  

All visual variables live in `src/scss/_variables.scss`.  
Example – change the primary colour:

```scss
// src/scss/_variables.scss
$color-primary: #0066ff; // <-- modify this value
```

After editing, run `npm run build` (or let the dev server rebuild automatically).  

### Adding new destinations  

1. Create a new JSON file in `src/data/` (or edit `destinations.json`).  
   ```json
   {
     "id": "paris",
     "name": "Paris, France",
     "country": "France",
     "heroImage": "images/paris-hero.jpg",
     "summary": "The City of Light…",
     "highlights": [
       "Eiffel Tower",
       "Louvre Museum",
       "Seine River Cruise"
     ],
     "tags": ["city", "culture", "romance"]
   }
   ```
2. The JavaScript module `src/js/modules/destination.js` automatically reads the file and renders the card on the homepage.  

---  

## API Documentation  

Touropedia does **not** expose a back‑end API out of the box, but the front‑end code is written to consume a simple JSON‑based REST‑like interface. This makes it trivial to swap the static JSON files for a real API later.

### Expected Endpoints  

| Method | URL | Description | Sample Response |
|--------|-----|-------------|-----------------|
| `GET` | `/api/destinations` | Returns an array of all destinations. | `[{ "id": "paris", "name": "Paris", … }, …]` |
| `GET` | `/api/destinations/:id` | Returns a single destination object. | `{ "id": "paris", "name": "Paris", … }` |
| `GET` | `/api/search?q=term` | Full‑text search across destination names, tags, and summaries. | `{ "results": [{ "id": "paris", … }] }` |

### Front‑end API Wrapper (`src/js/api.js`)  

```js
/**
 * Simple wrapper around fetch() for Touropedia data.
 * Replace the base URL with your real back‑end when ready.
 */
export const API = (() => {
  const BASE_URL = import.meta.env.MODE === 'production'
    ? '/api'               // production (static JSON folder)
    : '/api';              // dev server (proxy to src/data)

  const handleResponse = async (res) => {
    if (!res.ok) {
      const err = await res.json();
      throw new Error(err.message || 'API error');
    }
    return res.json();
  };

  return {
    /** @returns {Promise<Array>} */
    getDestinations: () => fetch(`${BASE_URL}/destinations`).then(handleResponse),

    /** @param {string} id */
    getDestination: (id) => fetch(`${BASE_URL}/destinations/${id}`).then(handleResponse),

    /** @param {string} query */
    search: (query) => fetch(`${BASE_URL}/search?q=${encodeURIComponent(query)}`).then(handleResponse),
  };
})();
```

**How to replace the data source**  

1. Deploy a back‑end that respects the three endpoints above.  
2. Update `BASE_URL` in `src/js/api.js` to point at the new server (e.g., `https://api.touropedia.com`).  
3. No other code changes are required – the UI automatically consumes the new data.  

---  

## Examples  

Below are a few practical snippets that demonstrate how to use Touropedia’s components and API wrapper.

### 1. Rendering the Destination Grid  

```js
import { API } from './api.js';
import { renderDestinationCard } from './modules/destination.js';

async function initHomePage() {
  try {
    const destinations = await API.getDestinations();
    const container = document.querySelector('#destinations-grid');

    destinations.forEach(dest => {
      const card = renderDestinationCard(dest);
      container.appendChild(card);
    });
  } catch (err) {
    console.error('Failed to load destinations:', err);
  }
}

document.addEventListener('DOMContentLoaded', initHomePage);
```

### 2. Search Component  

```html
<!-- index.html -->
<header class="search-bar">
  <input type="search" id="search-input" placeholder="Search destinations…" />
  <button id="search-btn">🔍</button>
</header>
<section id="search-results" class="grid"></section>
```

```js
// src/js/modules/search.js
import { API } from '../api.js';
import { renderDestinationCard } from './destination.js';

const input = document.getElementById('search-input');
const btn   = document.getElementById('search-btn');
const resultsContainer = document.getElementById('search-results');

async function performSearch() {
  const query = input.value.trim();
  if (!query) return;

  try {
    const { results } = await API.search(query);
    resultsContainer.innerHTML = ''; // clear previous

    results.forEach(dest => {
      resultsContainer.appendChild(renderDestinationCard(dest));
    });
  } catch (e) {
    console.error('Search error:', e);
  }
}

btn.addEventListener('click', performSearch);
input.addEventListener('keypress', e => {
  if (e.key === 'Enter') performSearch();
});
```

### 3. Modal Dialog for Destination Details  

```html
<!-- Somewhere in index.html -->
<div id="modal" class="modal hidden" aria-hidden="true">
  <div class="modal__content" role="dialog" aria-modal="true">
    <button class="modal__close" aria-label="Close">&times;</button>
    <div id="modal-body"></div>
  </div>
</div>
```

```js
// src/js/modules/modal.js
export function openModal(html) {
  const modal = document.getElementById('modal');
  const body  = document.getElementById('modal-body