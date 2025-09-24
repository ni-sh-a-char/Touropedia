# Touropedia  
**A responsive travel website built with HTML, SCSS, CSS and JavaScript**  

---  

## Table of Contents  

| # | Section |
|---|---------|
| 1 | [About the Project](#about-the-project) |
| 2 | [Features](#features) |
| 3 | [Prerequisites](#prerequisites) |
| 4 | [Installation](#installation) |
| 5 | [Development Workflow](#development-workflow) |
| 6 | [Build & Deploy](#build--deploy) |
| 7 | [Usage (Running Locally)](#usage-running-locally) |
| 8 | [Project Structure](#project-structure) |
| 9 | [API Documentation](#api-documentation) |
|10 | [Customization (SCSS Variables)](#customization-scss-variables) |
|11 | [Examples](#examples) |
|12 | [Contributing](#contributing) |
|13 | [License](#license) |

---  

## About the Project  

Touropedia is a **responsive, SEO‑friendly travel portal** that showcases destinations, itineraries, travel tips, and user‑generated reviews. The UI is built with **semantic HTML5**, styled with **SCSS** (compiled to CSS), and powered by **vanilla JavaScript** (ES6 modules).  

The site works out‑of‑the‑box on desktop, tablet, and mobile devices, and it can be extended with a simple JSON‑based data source or hooked up to any REST API.

---  

## Features  

| ✅ | Feature |
|---|---------|
| ✅ | Mobile‑first responsive layout (Flexbox + CSS Grid) |
| ✅ | SCSS architecture with variables, mixins, and BEM naming |
| ✅ | Lazy‑loaded images & IntersectionObserver for performance |
| ✅ | Search & filter of destinations (client‑side) |
| ✅ | Interactive map modal (Leaflet.js – optional) |
| ✅ | Dark‑mode toggle (CSS custom properties) |
| ✅ | Build pipeline with **npm**, **sass**, **esbuild**, and **live‑server** |
| ✅ | Easy to theme – just edit `_variables.scss` |
| ✅ | Fully documented JavaScript API (`src/js/api.js`) |

---  

## Prerequisites  

| Tool | Minimum Version |
|------|-----------------|
| **Git** | 2.20+ |
| **Node.js** | 18.x (LTS) |
| **npm** | 9.x (comes with Node) |
| **A modern browser** | Chrome, Firefox, Edge, Safari (latest) |

> **Note** – The project does **not** require a backend to run locally; all data can be stored in JSON files under `data/`. If you want to connect to a remote API, just replace the stub functions in `src/js/api.js`.

---  

## Installation  

```bash
# 1️⃣ Clone the repository
git clone https://github.com/your‑username/Touropedia.git
cd Touropedia

# 2️⃣ Install Node dependencies (dev tools only)
npm ci   # or `npm install` if you prefer

# 3️⃣ Install the SCSS compiler (sass is a dev dependency)
#    No extra step needed – `npm run dev` will handle it.
```

> **Tip** – If you only need the compiled assets (e.g., for a quick demo), you can skip the npm step and use the pre‑built `dist/` folder that ships with the repo.

---  

## Development Workflow  

| Command | Description |
|---------|-------------|
| `npm run dev` | Starts a **live‑server** (http://localhost:3000) + watches SCSS & JS files. Hot‑reloading enabled. |
| `npm run build` | Compiles SCSS → CSS, bundles JS with **esbuild**, and copies static assets to `dist/`. |
| `npm run lint` | Runs **stylelint** (SCSS) and **eslint** (JS) in strict mode. |
| `npm run format` | Auto‑formats code with **Prettier**. |
| `npm run preview` | Serves the production build from `dist/` (useful for final testing). |

All scripts are defined in `package.json`:

```json
{
  "scripts": {
    "dev": "npm-run-all --parallel watch:scss watch:js serve",
    "watch:scss": "sass --watch src/scss:dist/css --no-source-map",
    "watch:js": "esbuild src/js/index.js --bundle --outfile=dist/js/bundle.js --servedir=dist --watch",
    "serve": "live-server dist --port=3000 --open=./index.html",
    "build": "npm-run-all clean compile:scss compile:js copy:assets",
    "compile:scss": "sass src/scss:dist/css --style=compressed",
    "compile:js": "esbuild src/js/index.js --bundle --minify --outfile=dist/js/bundle.min.js",
    "copy:assets": "cpy \"src/assets/**/*\" dist/assets",
    "clean": "rimraf dist",
    "lint": "npm-run-all lint:scss lint:js",
    "lint:scss": "stylelint \"src/scss/**/*.scss\"",
    "lint:js": "eslint \"src/js/**/*.js\"",
    "format": "prettier --write \"src/**/*.{js,scss,html,md}\""
  }
}
```

---  

## Build & Deploy  

1. **Build** the production assets  

   ```bash
   npm run build
   ```

   The output lives in the `dist/` folder:

   ```
   dist/
   ├─ css/
   │   └─ main.min.css
   ├─ js/
   │   └─ bundle.min.js
   ├─ assets/
   │   └─ (images, fonts, etc.)
   └─ index.html
   ```

2. **Deploy**  

   - **Static hosting** (GitHub Pages, Netlify, Vercel, Cloudflare Pages, etc.) – just point the host to the `dist/` directory.  
   - **Custom server** – serve `dist/` with any static file server (nginx, Apache, Express, etc.).  

   Example **nginx** config snippet:

   ```nginx
   server {
       listen 80;
       server_name touropedia.example.com;

       root /var/www/touropedia/dist;
       index index.html;

       location / {
           try_files $uri $uri/ =404;
       }

       # Optional: enable gzip compression
       gzip on;
       gzip_types text/css application/javascript;
   }
   ```

---  

## Usage (Running Locally)  

```bash
npm run dev
```

- Open **http://localhost:3000** in your browser.  
- Any change to `src/scss/**/*.scss` or `src/js/**/*.js` triggers an automatic rebuild and live‑reload.  

### Quick “no‑npm” demo  

If you just want to see the site without installing anything:

```bash
# From the repository root
open dist/index.html   # macOS
# or double‑click the file on Windows/Linux
```

---  

## Project Structure  

```
Touropedia/
├─ .github/                # CI / issue templates
├─ src/
│   ├─ assets/             # Original images, fonts, icons
│   ├─ data/               # JSON data files (destinations, reviews, etc.)
│   ├─ js/
│   │   ├─ api.js          # Public JS API (fetch, filter, etc.)
│   │   ├─ ui.js           # UI helpers (modals, toggles)
│   │   └─ index.js        # Entry point – imports everything
│   ├─ scss/
│   │   ├─ base/           # Reset, typography, utilities
│   │   ├─ components/     # Buttons, cards, nav, etc.
│   │   ├─ layout/         # Grid, header, footer
│   │   ├─ themes/         # Light / dark theme variables
│   │   └─ main.scss      