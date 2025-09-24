# Touropedia  
**A responsive travel website built with HTML, SCSS, CSS and JavaScript**  

---  

## Table of Contents  

1. [Project Overview](#project-overview)  
2. [Installation](#installation)  
3. [Usage](#usage)  
4. [API Documentation](#api-documentation)  
5. [Examples](#examples)  
6. [Development Scripts](#development-scripts)  
7. [Contributing](#contributing)  
8. [License](#license)  

---  

## Project Overview  

Touropedia is a modern, mobile‑first travel portal that showcases destinations, itineraries, and travel tips. The site is built with:

* **HTML5** – semantic markup for accessibility and SEO.  
* **SCSS** – modular, maintainable styling with variables, mixins, and nesting.  
* **CSS** – compiled from SCSS, with a responsive grid and custom animations.  
* **JavaScript (ES6+)** – vanilla JS for UI interactions, data fetching, and client‑side routing.  

The repository is a **static site** – there is no server‑side code, but the front‑end consumes a public JSON API (e.g., a travel‑destinations endpoint) to populate content dynamically.

---  

## Installation  

> **Prerequisites**  
> - **Node.js** (>= 18.x) – includes npm.  
> - **Git** – to clone the repository.  

### 1. Clone the repository  

```bash
git clone https://github.com/yourusername/Touropedia.git
cd Touropedia
```

### 2. Install dependencies  

Touropedia uses a minimal toolchain for SCSS compilation, live‑reloading, and bundling:

| Package | Purpose |
|---------|---------|
| `sass` (Dart Sass) | Compile `.scss` → `.css`. |
| `live-server` | Simple static dev server with hot reload. |
| `eslint` + `prettier` | Code quality & formatting (optional). |
| `webpack` (optional) | If you prefer a bundler for JS modules. |

Install the core dev dependencies:

```bash
npm install
```

> **Note** – All dependencies are listed in `package.json`. If you only need the compiled CSS, you can skip the npm step and use the pre‑built `dist/css` folder.

### 3. Build the SCSS (first time)  

```bash
npm run build:css
```

This command compiles `src/scss/main.scss` into `dist/css/main.css`.

---  

## Usage  

### Development (Live preview)

```bash
npm start
```

- Starts **live‑server** on `http://127.0.0.1:5500` (or the next available port).  
- Watches `src/scss/**/*.scss` and `src/js/**/*.js` for changes.  
- Automatically recompiles SCSS and reloads the browser.

### Production build  

```bash
npm run build
```

- Compiles SCSS → minified CSS (`dist/css/main.min.css`).  
- Copies HTML files from `src/` to `dist/`.  
- Bundles/minifies JavaScript (if you enable the optional Webpack step).  
- The final static site lives in the `dist/` folder, ready to be deployed to any static host (GitHub Pages, Netlify, Vercel, etc.).

### Deploy to GitHub Pages (example)

```bash
npm run deploy
```

> The `deploy` script pushes the contents of `dist/` to the `gh-pages` branch.  
> Make sure the repository has a `gh-pages` branch or create one first.

---  

## API Documentation  

Touropedia does **not** expose its own backend API, but it consumes a public JSON endpoint to retrieve travel data. All API interactions are encapsulated in the `src/js/api.js` module.

### `src/js/api.js`

| Export | Description | Signature | Returns |
|--------|-------------|-----------|---------|
| `fetchDestinations()` | Retrieves the list of travel destinations. | `fetchDestinations(): Promise<Destination[]>` | `Promise` that resolves to an array of `Destination` objects. |
| `fetchDestination(id)` | Retrieves detailed data for a single destination. | `fetchDestination(id: string | number): Promise<DestinationDetail>` | `Promise` that resolves to a `DestinationDetail` object. |
| `searchDestinations(query)` | Performs a client‑side fuzzy search on the cached destination list. | `searchDestinations(query: string): Destination[]` | Synchronous array of matching destinations. |
| `setApiBaseUrl(url)` | Override the default API base URL (useful for testing). | `setApiBaseUrl(url: string): void` | — |

#### Types (JSDoc)

```js
/**
 * @typedef {Object} Destination
 * @property {number} id          - Unique identifier.
 * @property {string} name        - Destination name.
 * @property {string} country     - Country name.
 * @property {string} thumbnail   - URL to a thumbnail image.
 * @property {string[]} tags      - Tags such as ["beach","mountain"].
 */

/**
 * @typedef {Object} DestinationDetail
 * @property {number} id
 * @property {string} name
 * @property {string} country
 * @property {string} description   - Full description (HTML allowed).
 * @property {string[]} images      - Array of image URLs.
 * @property {Object} stats         - Visitor stats, rating, etc.
 */
```

#### Example usage

```js
import { fetchDestinations, fetchDestination } from './api.js';

async function initHomePage() {
  const destinations = await fetchDestinations();
  renderDestinationCards(destinations);
}

async function showDetail(id) {
  const detail = await fetchDestination(id);
  renderDetailPage(detail);
}
```

### Error handling  

All API functions reject with an `Error` object that contains a `status` property (HTTP status code) and a `message`. Example:

```js
try {
  const data = await fetchDestinations();
} catch (err) {
  console.error(`API error (${err.status}): ${err.message}`);
  showToast('Unable to load destinations. Please try again later.');
}
```

---  

## Examples  

Below are common usage patterns that you can copy/paste into your own pages or components.

### 1. Rendering a list of destination cards  

```html
<!-- index.html -->
<section id="destinations" class="grid grid--3col"></section>
```

```js
// src/js/home.js
import { fetchDestinations } from './api.js';

const container = document.getElementById('destinations');

function createCard(dest) {
  const card = document.createElement('article');
  card.className = 'card';
  card.innerHTML = `
    <img src="${dest.thumbnail}" alt="${dest.name}" class="card__img" loading="lazy">
    <h3 class="card__title">${dest.name}</h3>
    <p class="card__country">${dest.country}</p>
    <a href="detail.html?id=${dest.id}" class="card__link">Explore →</a>
  `;
  return card;
}

async function render() {
  try {
    const list = await fetchDestinations();
    list.forEach(dest => container.appendChild(createCard(dest)));
  } catch (e) {
    container.innerHTML = '<p class="error">Failed to load destinations.</p>';
  }
}

render();
```

### 2. Detail page with image carousel  

```html
<!-- detail.html -->
<main id="detail-page" class="detail-page">
  <h1 id="dest-name"></h1>
  <p id="dest-country"></p>
  <div id="carousel" class="carousel"></div>
  <section id="description"></section>
</main>
```

```js
// src/js/detail.js
import { fetchDestination } from './api.js';

function buildCarousel(images) {
  const carousel = document.getElementById('carousel');
  carousel.innerHTML = images
    .map(
      src => `<div class="carousel__slide"><img src="${src}" loading="lazy"></div>`
    )
    .join('');
  // Simple vanilla carousel – you can replace with Swiper, Flickity, etc.
  let index = 0;
  const slides = carousel.children;
  function showSlide(i) {
    Array.from(slides).forEach((s, idx) => {
      s.style.display = idx === i ? 'block' : 'none';
    });
  }
  showSlide(index);
  setInterval(() => {
    index = (index + 1) % slides.length;
    showSlide(index);
  }, 4000);
}

async function initDetail() {
  const params = new URLSearchParams(window.location.search);
  const id =