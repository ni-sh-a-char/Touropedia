# Touropedia  
**A responsive travel website built with HTML, SCSS, CSS and JavaScript**  

---  

## Table of Contents
1. [Project Overview](#project-overview)  
2. [Prerequisites](#prerequisites)  
3. [Installation](#installation)  
4. [Development & Build Workflow](#development--build-workflow)  
5. [Usage](#usage)  
6. [API Documentation](#api-documentation)  
7. [Examples](#examples)  
8. [Contributing](#contributing)  
9. [License](#license)  

---  

## Project Overview
Touropedia is a **responsive, client‑side travel portal** that showcases destinations, itineraries, and travel tips. The site is built with:

| Technology | Purpose |
|------------|---------|
| **HTML5**  | Semantic markup and page structure |
| **SCSS**   | Modular, maintainable styling with variables, mixins, and nesting |
| **CSS**    | Compiled output for the browser |
| **JavaScript (ES6+)** | Interactive UI components, data fetching, and client‑side routing |

The repository ships with a lightweight development server, SCSS compilation, and a simple JSON‑based mock API that can be swapped for a real backend.

---  

## Prerequisites
| Tool | Minimum Version | Why |
|------|----------------|-----|
| **Node.js** | `>= 18.x` | Provides `npm`/`pnpm` and runs the build scripts |
| **Git** | any | To clone the repository |
| **A modern browser** | Chrome/Firefox/Edge (latest) | For testing the responsive UI |
| **Optional** – **Docker** | `>= 20.x` | Run the dev environment in a container (see Docker section) |

---  

## Installation  

### 1. Clone the repository
```bash
git clone https://github.com/yourusername/Touropedia.git
cd Touropedia
```

### 2. Install dependencies
Touropedia uses **npm** (you can also use `pnpm` or `yarn` if you prefer).

```bash
npm ci
```

> **Why `npm ci`?**  
> It installs exactly the versions listed in `package-lock.json`, guaranteeing reproducible builds.

### 3. Set up environment variables (optional)
If you want to point the front‑end to a real API instead of the bundled mock data, create a `.env` file at the project root:

```dotenv
# .env
API_BASE_URL=https://api.yourtravelservice.com/v1
```

The build scripts automatically inject `process.env.API_BASE_URL` into the client code via Vite’s built‑in env handling.

### 4. (Optional) Docker setup
```bash
docker compose up --build
```
The compose file starts a container with Node, installs dependencies, and runs the dev server on `http://localhost:5173`.

---  

## Development & Build Workflow  

| Command | Description |
|---------|-------------|
| `npm run dev` | Starts a hot‑reloading development server (Vite). |
| `npm run build` | Produces an optimized production build in `dist/`. |
| `npm run preview` | Serves the production build locally for QA. |
| `npm run lint` | Runs ESLint + Stylelint on the codebase. |
| `npm run format` | Formats files with Prettier. |
| `npm run clean` | Removes `node_modules` and `dist/` for a fresh start. |

> **Tip:** The SCSS files live under `src/scss/`. Any change triggers an automatic recompilation to `src/css/` (handled by Vite’s plugin).  

---  

## Usage  

### Running locally (development)

```bash
npm run dev
```

Open your browser at `http://localhost:5173`. The site will automatically reload when you edit HTML, SCSS, or JS files.

### Deploying to production  

1. Build the static assets:

   ```bash
   npm run build
   ```

2. Upload the contents of the `dist/` folder to any static‑file host (GitHub Pages, Netlify, Vercel, AWS S3 + CloudFront, etc.).  

   Example for **Netlify**:

   ```bash
   netlify deploy --dir=dist --prod
   ```

3. If you are using a custom domain, configure DNS to point to your host and enable HTTPS.

### Configuration flags  

| Flag | Effect |
|------|--------|
| `VITE_API_BASE_URL` | Overrides the API endpoint at runtime (see `.env`). |
| `VITE_THEME=dark` | Forces the dark theme on first load (overrides user‑prefers‑color‑scheme). |
| `VITE_ANALYTICS_ID` | Enables Google Analytics (or any analytics) integration. |

You can pass these via the command line:

```bash
VITE_THEME=dark npm run dev
```

---  

## API Documentation  

Touropedia’s front‑end talks to a **REST‑like JSON API**. The default mock API lives in `public/mock-data/`. You can replace it with a real backend that respects the same contract.

### Base URL
```
{API_BASE_URL}/
```
If `API_BASE_URL` is not defined, the site falls back to `/api/` (served from the static `public/` folder).

### Endpoints  

| Method | Endpoint | Description | Query Parameters | Example Response |
|--------|----------|-------------|------------------|------------------|
| `GET` | `/destinations` | Returns a list of travel destinations. | `?page=1&limit=20` | ```json [{ "id": 1, "name": "Paris", "slug": "paris", "coverImage": "/images/paris.jpg", "rating": 4.8 }]``` |
| `GET` | `/destinations/:slug` | Details for a single destination. | — | ```json { "id": 1, "name": "Paris", "description": "...", "gallery": [...], "highlights": [...] }``` |
| `GET` | `/itineraries?dest=:slug` | Suggested itineraries for a destination. | `dest` (slug) | ```json [{ "title": "3‑Day Paris Highlights", "days": 3, "price": 1200 }]``` |
| `GET` | `/search` | Full‑text search across destinations & articles. | `q` (string) | ```json [{ "type": "destination", "id": 5, "title": "Tokyo" }]``` |
| `POST` | `/contact` | Submit a contact form. | Body: `{ name, email, message }` | ```json { "status": "ok", "messageId": "abc123" }``` |

> **Note:** All responses are JSON and include a top‑level `status` field (`"success"` or `"error"`). Errors contain an `error` object with `code` and `message`.

### JavaScript Helper Module (`src/js/api.js`)

```js
/**
 * @module api
 * @description Small wrapper around fetch() that handles base URL,
 *              JSON parsing, and error handling.
 */

const BASE_URL = import.meta.env.VITE_API_BASE_URL || '/api';

/**
 * Generic GET request.
 * @param {string} path - API path (e.g., '/destinations')
 * @param {Object} [params] - Query parameters
 * @returns {Promise<any>}
 */
export async function get(path, params = {}) {
  const url = new URL(BASE_URL + path, location.origin);
  Object.entries(params).forEach(([k, v]) => url.searchParams.append(k, v));

  const response = await fetch(url, { credentials: 'same-origin' });
  const data = await response.json();

  if (!response.ok) {
    throw new Error(data.error?.message || 'API request failed');
  }
  return data;
}

/**
 * Generic POST request.
 * @param {string} path
 * @param {Object} body
 * @returns {Promise<any>}
 */
export async function post(path, body) {
  const response = await fetch(BASE_URL + path, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(body),
    credentials: 'same-origin',
  });

  const data = await response.json();
  if (!response.ok) {
    throw new Error(data.error?.message || 'API request failed');
  }
  return data;
}
```

#### Example usage in a component

```js
import { get, post } from './api.js';

// Load destinations for the homepage carousel
async function loadDestinations() {
  try {
    const { status, data } = await get('/destinations', { limit: 8 });
    if (status === 'success') render