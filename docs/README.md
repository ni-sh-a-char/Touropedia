# Touropedia  
**A responsive travel website built with HTML, SCSS, CSS and JavaScript**  

---  

## Table of Contents  

| Section | Description |
|---------|-------------|
| **[Installation](#installation)** | How to get the project up and running locally |
| **[Usage](#usage)** | Development workflow, building for production and running the site |
| **[API Documentation](#api-documentation)** | Overview of the public JavaScript API (modules, classes, utilities) |
| **[Examples](#examples)** | Real‑world snippets that show how to extend or customise Touropedia |
| **[Contributing](#contributing)** | Guidelines for submitting pull‑requests |
| **[License](#license)** | Open‑source license information |

---  

## Installation  

> **Prerequisites**  
> - **Node.js** (>= 18.x) – includes npm/yarn.  
> - **Git** – to clone the repository.  
> - **A modern browser** (Chrome, Firefox, Edge, Safari) for testing.  

### 1. Clone the repository  

```bash
git clone https://github.com/your‑username/Touropedia.git
cd Touropedia
```

### 2. Install dependencies  

Touropedia uses a lightweight toolchain based on **Vite** (dev server & bundler) and **Sass** for SCSS compilation.

```bash
# Using npm
npm ci

# Or using Yarn
yarn install --frozen-lockfile
```

> **Why `npm ci`?**  
> It installs exactly the versions listed in `package-lock.json`, guaranteeing reproducible builds.

### 3. Compile SCSS (optional – Vite does this automatically in dev mode)  

If you want to generate the CSS once, run:

```bash
npm run build:css   # or `yarn build:css`
```

The compiled CSS will be placed in `dist/assets/css/`.

### 4. Verify the installation  

```bash
npm run dev   # starts a hot‑reloading dev server at http://localhost:5173
```

Open the URL in your browser – you should see the **Touropedia** landing page.

---  

## Usage  

### Development  

| Command | Description |
|---------|-------------|
| `npm run dev` | Starts Vite’s development server with hot‑module replacement (HMR). |
| `npm run lint` | Runs ESLint + Stylelint on the source files. |
| `npm run test` | Executes unit tests (Jest + Testing Library). |
| `npm run format` | Runs Prettier to auto‑format code. |

> **Tip:** Add `--open` to `npm run dev` if you want the browser to launch automatically: `npm run dev -- --open`.

### Building for Production  

```bash
npm run build
```

- **Output**: `dist/` – a fully static site ready to be deployed to any static‑host (GitHub Pages, Netlify, Vercel, Cloudflare Pages, etc.).  
- **Optimisations**:  
  - SCSS → minified CSS  
  - JavaScript → ES‑module bundle with tree‑shaking & minification  
  - Images → WebP conversion (via `vite-imagetools`)  

### Previewing the Production Build  

```bash
npm run preview
```

Serves the `dist/` folder locally on `http://localhost:4173` – useful for a final sanity check before deployment.

### Deploying  

Below are the most common static‑host deployment commands. Adjust the remote URL to your own repository.

#### GitHub Pages  

```bash
# Assuming the `gh-pages` branch is used for the site
npm run build
npx gh-pages -d dist
```

#### Netlify  

1. Connect the repo in the Netlify dashboard.  
2. Set **Build command**: `npm run build`  
3. Set **Publish directory**: `dist`  

#### Vercel  

```bash
vercel --prod
```

Vercel automatically detects the Vite project and runs `npm run build`.

---  

## API Documentation  

Touropedia’s front‑end is modularised into ES‑modules under `src/js/`. The public API is intentionally small – only the utilities that a contributor might need to extend the site.

### 1. `src/js/api.js` – Core API  

| Export | Type | Description |
|--------|------|-------------|
| `fetchDestinations()` | `async () => Destination[]` | Retrieves the list of travel destinations from `data/destinations.json`. Returns an array of **Destination** objects. |
| `searchDestinations(query: string)` | `(query) => Destination[]` | Filters the destination list by name, country or tags (case‑insensitive). |
| `addDestination(dest: Destination)` | `(dest) => void` | Adds a new destination to the in‑memory store and persists it to `data/destinations.json` (only works in dev mode with the local JSON server). |
| `on(event: string, handler: Function)` | `(event, handler) => void` | Simple pub/sub for UI components (`'destinations:updated'`, `'theme:changed'`, …). |
| `off(event: string, handler: Function)` | `(event, handler) => void` | Unsubscribe a previously registered handler. |

#### Destination type (JSDoc)

```js
/**
 * @typedef {Object} Destination
 * @property {string} id          - UUID v4
 * @property {string} name        - Human‑readable name (e.g. "Kyoto")
 * @property {string} country     - Country name
 * @property {string[]} tags      - Keywords (e.g. ["culture","food"])
 * @property {string} image       - Relative path to the hero image
 * @property {string} description - Markdown‑formatted description
 * @property {number} rating      - 0‑5 (half‑star increments)
 */
```

### 2. `src/js/theme.js` – Theme Manager  

| Export | Type | Description |
|--------|------|-------------|
| `initTheme()` | `() => void` | Detects system preference (`prefers-color-scheme`) and applies the appropriate CSS variables. |
| `setTheme(theme: 'light' | 'dark')` | `(theme) => void` | Forces a theme and stores the choice in `localStorage`. |
| `toggleTheme()` | `() => void` | Switches between light and dark mode. |
| `getCurrentTheme()` | `() => 'light' | 'dark'` | Returns the active theme. |

**CSS Variables** (defined in `src/scss/_variables.scss`):  

```scss
:root {
  --color-bg: #ffffff;
  --color-text: #222222;
  --color-primary: #0066ff;
  // …
}
[data-theme='dark'] {
  --color-bg: #111111;
  --color-text: #eeeeee;
  --color-primary: #3399ff;
}
```

### 3. `src/js/modal.js` – Generic Modal Utility  

| Export | Type | Description |
|--------|------|-------------|
| `openModal(content: HTMLElement | string, options?)` | `(content, options) => void` | Inserts `content` into the modal container and displays it. |
| `closeModal()` | `() => void` | Hides the modal and clears its inner HTML. |
| `onClose(callback: Function)` | `(cb) => void` | Register a callback that runs after the modal is closed. |

**Options**  

```ts
interface ModalOptions {
  /** If true, clicking outside the modal closes it */
  closeOnOverlay?: boolean;
  /** CSS class added to the modal wrapper for custom styling */
  className?: string;
}
```

### 4. `src/js/router.js` – Simple SPA Router  

| Export | Type | Description |
|--------|------|-------------|
| `initRouter(routes: Route[])` | `(routes) => void` | Registers routes and starts listening to `popstate`. |
| `navigate(path: string)` | `(path) => void` | Programmatically change the URL and render the associated view. |

**Route definition**

```ts
interface Route {
  /** URL pattern, e.g. '/destinations/:id' */
  path: string;
  /** Function that returns a Promise resolving to an HTMLElement */
  component: () => Promise<HTMLElement>;
}
```

---  

## Examples  

Below are practical snippets that demonstrate how to extend Touropedia without digging into the core codebase.

### 1. Adding a New Destination (dev mode)  

```js
import { addDestination, fetchDestinations } from './api.js