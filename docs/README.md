# Touropedia  
**A responsive travel website built with HTML, SCSS, CSS and JavaScript**  

---  

## Table of Contents  

| Section | Description |
|---------|-------------|
| **Overview** | What Touropedia is and its core features |
| **Prerequisites** | Tools you need before you start |
| **Installation** | How to get the project on your machine |
| **Development & Usage** | Running a local dev server, hot‑reloading, linting, testing |
| **Build & Deployment** | Production build, static‑site hosting options |
| **API Documentation** | Public JavaScript modules, utility functions, and custom events |
| **Examples** | Sample code snippets that illustrate common use‑cases |
| **Contributing** | How to help improve the project |
| **License** | Open‑source licensing information |

---  

## Overview  

Touropedia is a **responsive, SEO‑friendly travel portal** that showcases destinations, itineraries, and travel tips. It is built with:

* **HTML5** – semantic markup for accessibility and SEO.  
* **SCSS** – modular, maintainable styling with variables, mixins, and BEM naming.  
* **CSS** – compiled from SCSS, with a mobile‑first approach and CSS custom properties for theming.  
* **Vanilla JavaScript (ES6+)** – no framework overhead, but organized as ES modules for easy scaling.  

The site works on all modern browsers and adapts gracefully from mobile phones to large‑screen desktops.

---  

## Prerequisites  

| Tool | Minimum version | Why it’s needed |
|------|----------------|-----------------|
| **Git** | 2.20+ | Clone the repository |
| **Node.js** | 18.x LTS | Run the build scripts, dev server, and SCSS compiler |
| **npm** (or **pnpm** / **yarn**) | 9.x+ | Package manager for dependencies |
| **A modern browser** | Chrome/Firefox/Edge/Safari (latest) | Test the UI locally |

> **Tip:** If you use `pnpm` or `yarn`, replace `npm` commands with the equivalent (`pnpm install`, `pnpm run dev`, …).

---  

## Installation  

```bash
# 1️⃣ Clone the repo
git clone https://github.com/your‑username/Touropedia.git
cd Touropedia

# 2️⃣ Install dependencies
npm ci          # or `npm install` if you prefer a fresh lockfile

# 3️⃣ (Optional) Install the optional pre‑commit hook
npx husky install
```

> **What gets installed?**  
> * `sass` – compiles SCSS → CSS.  
> * `esbuild` – fast bundler for JavaScript modules.  
> * `eslint` + `stylelint` – code quality enforcement.  
> * `live-server` – tiny dev server with live‑reload.  

---  

## Development & Usage  

### 1️⃣ Start the development server  

```bash
npm run dev
```

* Serves `src/` on `http://localhost:3000`.  
* SCSS is compiled on‑the‑fly, and JavaScript modules are bundled with **esbuild** in watch mode.  
* Changes to `.html`, `.scss`, or `.js` files trigger an automatic browser refresh.

### 2️⃣ Lint & format  

```bash
npm run lint          # runs ESLint + Stylelint
npm run format        # runs Prettier on the whole codebase
```

### 3️⃣ Run unit tests (if you add any)  

```bash
npm test
```

> **Note:** The starter repo ships with a minimal Jest configuration for testing pure‑JS utilities. Feel free to extend it for DOM‑related tests with `@testing-library/dom`.

---  

## Build & Deployment  

### Production build  

```bash
npm run build
```

* **Output**: `dist/` – contains minified CSS, bundled JS, and static HTML ready for any static‑host (GitHub Pages, Netlify, Vercel, Cloudflare Pages, etc.).  
* The build process also generates a **critical‑CSS** chunk for the above‑the‑fold content and adds a hash to filenames for cache‑busting.

### Deploy to GitHub Pages (example)  

```bash
npm run deploy
```

> The `deploy` script pushes the `dist/` folder to the `gh-pages` branch.  
> Adjust the `homepage` field in `package.json` if your repo lives under a sub‑path.

### Deploy to Netlify (example)  

1. Connect your GitHub repo to Netlify.  
2. Set **Build command**: `npm run build`  
3. Set **Publish directory**: `dist`  

Netlify will automatically run the build on each push.

---  

## API Documentation  

Touropedia ships with a small, well‑documented JavaScript API that powers interactive UI components (carousel, modal, lazy‑load, etc.). All modules live under `src/js/` and are exported as ES modules.

### 1️⃣ `src/js/utils.js`  

| Export | Type | Description |
|--------|------|-------------|
| `debounce(fn, wait, immediate = false)` | `function` | Returns a debounced version of `fn`. Useful for resize/scroll listeners. |
| `throttle(fn, limit)` | `function` | Returns a throttled version of `fn`. |
| `formatDate(date, locale = 'en-US')` | `function` | Returns a human‑readable date string. |
| `fetchJSON(url, options = {})` | `async function` | Wrapper around `fetch` that automatically parses JSON and throws on non‑2xx responses. |
| `createEvent(name, detail = {})` | `function` | Helper to create a `CustomEvent` with optional `detail`. |

#### Example  

```js
import { debounce, fetchJSON, createEvent } from './utils.js';

const onResize = debounce(() => {
  console.log('Window resized – recalculating layout');
}, 250);

window.addEventListener('resize', onResize);

// Fetch destination data
fetchJSON('/data/destinations.json')
  .then(data => console.log('Loaded', data))
  .catch(err => console.error(err));

// Dispatch a custom event when a destination card is clicked
document.querySelectorAll('.dest-card').forEach(card => {
  card.addEventListener('click', () => {
    const ev = createEvent('dest:select', { id: card.dataset.id });
    document.dispatchEvent(ev);
  });
});
```

---

### 2️⃣ `src/js/carousel.js`  

**Class** `Carousel` – a lightweight, dependency‑free carousel/slider.

| Constructor | Parameters |
|-------------|------------|
| `new Carousel(container, options = {})` | `container` – DOM element that holds the slides. <br> `options` – configuration object (see below). |

#### Options  

| Option | Default | Description |
|--------|---------|-------------|
| `loop` | `true` | Whether the carousel should wrap around. |
| `autoplay` | `false` | Auto‑advance slides. |
| `interval` | `5000` | Milliseconds between auto‑advances. |
| `transition` | `'slide'` | `'slide'` or `'fade'`. |
| `perView` | `1` | Number of slides visible at once (responsive breakpoints can be set via CSS). |

#### Public Methods  

| Method | Returns | Description |
|--------|---------|-------------|
| `next()` | `void` | Move to the next slide. |
| `prev()` | `void` | Move to the previous slide. |
| `goTo(index)` | `void` | Jump to a specific slide (0‑based). |
| `destroy()` | `void` | Remove event listeners and clean up. |

#### Events  

| Event name | `detail` | When emitted |
|------------|----------|--------------|
| `carousel:change` | `{ currentIndex, previousIndex }` | After a slide transition finishes. |
| `carousel:autoplay:start` | `{}` | When autoplay starts. |
| `carousel:autoplay:stop` | `{}` | When autoplay stops. |

#### Example  

```js
import Carousel from './carousel.js';

const carouselEl = document.querySelector('.hero-carousel');
const carousel = new Carousel(carouselEl, {
  loop: true,
  autoplay: true,
  interval: 4000,
  transition: 'fade'
});

// Listen for slide changes
carouselEl.addEventListener('carousel:change', e => {
  console.log('Now showing slide',