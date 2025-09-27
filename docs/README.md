# Touropedia  
**A responsive travel website built with HTML, SCSS, CSS and JavaScript**  

---  

## Table of Contents  

| Section | Description |
|---------|-------------|
| **[Overview](#overview)** | What Touropedia is and its main features |
| **[Installation](#installation)** | How to get the project up and running locally |
| **[Usage](#usage)** | Running the site, development workflow, and production build |
| **[API Documentation](#api-documentation)** | Public JavaScript API (data fetching, UI helpers, utilities) |
| **[Examples](#examples)** | Code snippets that demonstrate common tasks |
| **[Contributing](#contributing)** | How to contribute to the project |
| **[License](#license)** | Open‑source license information |
| **[Changelog](#changelog)** | Quick view of major releases |

---  

## Overview  

Touropedia is a **responsive travel portal** that lets users explore destinations, view photo galleries, read travel guides, and plan itineraries—all without leaving the browser.  

Key highlights:  

- **Mobile‑first responsive layout** built with SCSS and CSS Grid/Flexbox.  
- **Dynamic content** loaded via the built‑in JavaScript API (fetches JSON from the `/data` folder or a remote endpoint).  
- **Search & filter** with debounced input and multi‑criteria filtering (continent, price range, activity type).  
- **Map integration** (Leaflet.js) for interactive location previews.  
- **Modular architecture** – each UI component lives in its own folder (`src/components/`).  
- **Zero‑runtime dependencies** (vanilla JS) – only dev‑time tooling (Node, Sass, ESLint, Prettier).  

---  

## Installation  

> **Prerequisites**  
> - **Node.js ≥ 18** (for the build tools)  
> - **Git** (to clone the repo)  

### 1. Clone the repository  

```bash
git clone https://github.com/your‑username/Touropedia.git
cd Touropedia
```

### 2. Install development dependencies  

```bash
npm ci
```

> `npm ci` installs the exact versions listed in `package-lock.json`, guaranteeing reproducible builds.

### 3. Compile SCSS  

Touropedia ships with SCSS source files located in `src/scss/`. The build script compiles them to `dist/css/`.

```bash
npm run build:css   # one‑off compile
# or
npm run watch:css   # watch for changes and re‑compile automatically
```

### 4. (Optional) Lint & format  

```bash
npm run lint        # ESLint for JS
npm run format      # Prettier for all source files
```

### 5. Serve the site locally  

```bash
npm start           # starts a local dev server at http://localhost:3000
```

The dev server supports live‑reloading for HTML, SCSS, and JS changes.

---  

## Usage  

### Development workflow  

| Command | What it does |
|---------|--------------|
| `npm start` | Starts a **development server** (`webpack-dev-server` under the hood) with hot‑module replacement. |
| `npm run watch:css` | Watches SCSS files and recompiles to CSS on change. |
| `npm run build` | Produces a **production‑ready** bundle in `dist/` (minified CSS/JS, hashed filenames). |
| `npm run test` | Runs unit tests (Jest) for the JavaScript utilities. |
| `npm run lint` | Lints JavaScript with ESLint (Airbnb style). |
| `npm run format` | Formats code with Prettier. |

### Production deployment  

1. Build the static assets  

   ```bash
   npm run build
   ```

2. Upload the contents of the `dist/` folder to any static‑hosting provider (GitHub Pages, Netlify, Vercel, AWS S3 + CloudFront, etc.).  

3. Ensure the server serves `index.html` for **fallback routing** (e.g., for deep links like `/destinations/paris`).  

### Configuration  

All runtime configuration lives in `src/config.js`. Example:

```js
export const API_ENDPOINT = 'https://api.touropedia.com/v1';
export const DEFAULT_THEME = 'light'; // or 'dark'
export const MAP_PROVIDER = 'leaflet'; // currently only Leaflet is supported
```

You can override these values by creating a `config.local.js` file (ignored by Git) and importing it in `src/main.js`:

```js
import { API_ENDPOINT } from './config.local.js' // falls back to config.js if missing
```

---  

## API Documentation  

Touropedia’s public JavaScript API lives under the global namespace `Touropedia`. It is deliberately lightweight so that developers can embed the library in other projects or extend it with custom components.

### Namespace: `Touropedia`

| Member | Type | Description |
|--------|------|-------------|
| `Touropedia.init(options)` | `function` | Initializes the app. Accepts an optional `options` object (see below). |
| `Touropedia.fetchDestinations(params)` | `function` | Returns a `Promise<Array<Destination>>` with filtered destinations. |
| `Touropedia.search(query)` | `function` | Debounced search that resolves to an array of matching destinations. |
| `Touropedia.renderGallery(container, images)` | `function` | Renders a responsive image carousel inside `container`. |
| `Touropedia.map.showLocation(lat, lng, options)` | `function` | Centers the Leaflet map on a coordinate and optionally adds a marker. |
| `Touropedia.utils` | `object` | Miscellaneous helper functions (date formatting, price conversion, etc.). |

---

### `Touropedia.init(options)`

Initializes the UI, attaches event listeners, and loads the first batch of data.

**Parameters**

| Name | Type | Default | Description |
|------|------|---------|-------------|
| `options.container` | `string` | `'#app'` | CSS selector for the root element. |
| `options.apiEndpoint` | `string` | `config.API_ENDPOINT` | Base URL for the JSON API. |
| `options.theme` | `'light' \| 'dark'` | `config.DEFAULT_THEME` | Theme to apply on start. |
| `options.mapOptions` | `object` | `{ zoom: 5, center: [0,0] }` | Leaflet map defaults. |

**Example**

```js
Touropedia.init({
  container: '#touropedia-root',
  apiEndpoint: 'https://api.touropedia.com/v2',
  theme: 'dark',
});
```

---

### `Touropedia.fetchDestinations(params)`

Fetches destination data from the API (or from the local `data/` folder when running locally).

**Parameters**

| Name | Type | Description |
|------|------|-------------|
| `params` | `object` | Filtering options. |
| `params.continent` | `string` (optional) | e.g., `'Europe'`. |
| `params.priceRange` | `{ min: number, max: number }` (optional) | Filter by price. |
| `params.activities` | `Array<string>` (optional) | e.g., `['hiking', 'beach']`. |
| `params.limit` | `number` (optional) | Max number of results (default `20`). |
| `params.offset` | `number` (optional) | Pagination offset (default `0`). |

**Returns**: `Promise<Array<Destination>>`

**Destination type**

```ts
interface Destination {
  id: string;
  name: string;
  slug: string; // URL‑friendly name
  continent: string;
  country: string;
  description: string;
  price: number; // USD
  activities: string[];
  images: string[]; // URLs
  coordinates: { lat: number; lng: number };
}
```

**Example**

```js
Touropedia.fetchDestinations({
  continent: 'Asia',
  priceRange: { min: 500, max: 2000 },
  activities: ['culture', 'food'],
}).then(destinations => {
  console.log('Found', destinations.length, 'destinations');
});
```

---

### `Touropedia.search(query)`

Performs a **debounced** full‑text search across destination names and descriptions.

**Parameters**

| Name | Type | Description |
|------|------|-------------|
| `query` | `string` | Search term (minimum 2 characters). |

**Returns**: `Promise<Array<Destination>>`

**Example**

```js
const searchBox = document.querySelector('#search-input');
searchBox.addEventListener('input