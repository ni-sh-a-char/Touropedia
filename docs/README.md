# Touropedia  
**A responsive travel website built with HTML, SCSS, CSS and JavaScript**  

---  

## Table of Contents  

| # | Section | Link |
|---|---------|------|
| 1 | [About the Project](#about-the-project) | |
| 2 | [Demo](#demo) | |
| 3 | [Installation](#installation) | |
| 4 | [Development Workflow](#development-workflow) | |
| 5 | [Usage](#usage) | |
| 6 | [API Documentation](#api-documentation) | |
| 7 | [Examples](#examples) | |
| 8 | [Project Structure](#project-structure) | |
| 9 | [Contributing](#contributing) | |
|10 | [License](#license) | |

---  

## About the Project  

Touropedia is a **responsive, single‑page travel guide** that showcases destinations, itineraries, and travel tips. It is built with:

* **HTML5** – semantic markup for accessibility and SEO.  
* **SCSS** – modular, maintainable styling with variables, mixins, and nesting.  
* **CSS** – compiled from SCSS, with a mobile‑first responsive grid.  
* **Vanilla JavaScript (ES6+)** – fetches data from a public travel API, handles UI interactions, and provides a small client‑side routing system.

The site works offline after the first load (service‑worker powered) and is fully responsive from mobile phones to 4K displays.

---  

## Demo  

| Platform | Link |
|----------|------|
| Live Demo (GitHub Pages) | https://yourusername.github.io/touropedia/ |
| Netlify Preview | https://touropedia.netlify.app/ |

---  

## Installation  

> **Prerequisites**  
> * Node.js **≥ 18.x** (includes npm)  
> * Git (optional, for cloning)  

```bash
# 1️⃣ Clone the repository
git clone https://github.com/yourusername/touropedia.git
cd touropedia

# 2️⃣ Install development dependencies
npm ci   # or `npm install` if you prefer

# 3️⃣ Build the assets (SCSS → CSS, bundle JS)
npm run build   # creates a `dist/` folder

# 4️⃣ (Optional) Serve the built site locally
npm run preview   # opens http://localhost:5000
```

### npm Scripts  

| Script | Description |
|--------|-------------|
| `npm run dev` | Starts a hot‑reloading dev server (`vite`) on `http://localhost:5173`. |
| `npm run build` | Compiles SCSS, bundles JS, and outputs a production‑ready `dist/` folder. |
| `npm run lint` | Runs ESLint + Stylelint on the source files. |
| `npm run preview` | Serves the `dist/` folder with a tiny static server (useful for final testing). |
| `npm run clean` | Removes `node_modules` and `dist/` for a fresh start. |

---  

## Development Workflow  

1. **Start the dev server**  

   ```bash
   npm run dev
   ```

   *Changes to `.scss`, `.js`, or `.html` files trigger live reload.*  

2. **Write SCSS**  

   * All SCSS lives under `src/scss/`.  
   * Use the `variables.scss`, `mixins.scss`, and `breakpoints.scss` files to keep the design consistent.  

3. **Add JavaScript modules**  

   * Place modules in `src/js/`.  
   * Export functions as ES modules (`export function foo() {}`) and import them where needed.  

4. **Testing**  

   * Run `npm run lint` before committing.  
   * Use the built version (`npm run build && npm run preview`) to verify that the production bundle works.  

5. **Deploy**  

   * Push to the `main` branch → GitHub Actions automatically builds and deploys to GitHub Pages.  
   * Or run `npm run build` and push the `dist/` folder to any static host (Netlify, Vercel, Cloudflare Pages, etc.).  

---  

## Usage  

### Running the site locally (development)

```bash
npm run dev
# → Open http://localhost:5173 in your browser
```

### Running the production build locally  

```bash
npm run build
npm run preview
# → Open http://localhost:5000
```

### Customising the theme  

All colour, spacing, and typography values are defined in `src/scss/_variables.scss`.  
Edit the variables and re‑run `npm run dev` (or `npm run build` for production) to see the changes instantly.

```scss
// Example: change primary colour
$color-primary: #0066ff;   // default blue
```

### Adding a new destination  

1. Create a JSON file in `src/data/destinations/` (e.g., `paris.json`).  
2. Follow the schema described in the **API Documentation** section.  
3. The site automatically picks up new files on the next build because `src/data/` is imported by `src/js/data-loader.js`.  

---  

## API Documentation  

Touropedia is a **static front‑end** that consumes a small, self‑hosted JSON API located in `src/data/`. The API is exposed through the `DataLoader` module (`src/js/data-loader.js`). Below is the public interface that other modules (or external scripts) can use.

### `DataLoader` (ES module)

```js
/**
 * DataLoader – a tiny wrapper around fetch() that loads static JSON files.
 *
 * @class
 */
export default class DataLoader {
  /**
   * Load a list of destinations.
   *
   * @returns {Promise<Array<Destination>>}
   *   Resolves with an array of destination objects.
   *
   * Destination schema:
   * {
   *   id: string,               // unique slug, e.g. "paris"
   *   name: string,             // display name
   *   country: string,
   *   description: string,      // HTML string (sanitized)
   *   images: string[],         // array of image URLs (relative to /assets/)
   *   tags: string[],           // e.g. ["culture", "food"]
   *   rating: number,           // 0‑5 (float)
   *   coordinates: { lat: number, lng: number }
   * }
   */
  static async getDestinations();

  /**
   * Load a single destination by its slug.
   *
   * @param {string} slug - The destination id (e.g. "paris").
   * @returns {Promise<Destination>}
   */
  static async getDestination(slug);

  /**
   * Load a list of blog posts.
   *
   * @returns {Promise<Array<Post>>}
   *
   * Post schema:
   * {
   *   id: string,
   *   title: string,
   *   author: string,
   *   date: string,          // ISO 8601
   *   excerpt: string,
   *   content: string,       // HTML
   *   tags: string[]
   * }
   */
  static async getPosts();

  /**
   * Search destinations by keyword (case‑insensitive) and/or tags.
   *
   * @param {Object} options
   * @param {string} [options.q]   – free‑text query.
   * @param {string[]} [options.tags] – array of tag strings.
   * @returns {Promise<Array<Destination>>}
   */
  static async search(options);
}
```

### Example usage  

```js
import DataLoader from './js/data-loader.js';

// Load all destinations and render a card list
DataLoader.getDestinations()
  .then(destinations => {
    const container = document.querySelector('#destinations');
    container.innerHTML = destinations.map(dest => `
      <article class="card">
        <img src="${dest.images[0]}" alt="${dest.name}" loading="lazy">
        <h2>${dest.name}</h2>
        <p>${dest.description.slice(0, 120)}…</p>
        <a href="/destinations/${dest.id}" class="btn">Read more</a>
      </article>
    `).join('');
  })
  .catch(err => console.error('Failed to load destinations', err));
```

### Service Worker (PWA)  

The file `src/sw.js` registers a **Cache‑First** strategy for all static assets and a **Network‑First** fallback for the JSON data. The public API for the