# Touropedia  
**A responsive travel website built with HTML, SCSS, CSS and JavaScript**  

---  

## Table of Contents  

| # | Section |
|---|---------|
| 1 | [Overview](#overview) |
| 2 | [Prerequisites](#prerequisites) |
| 3 | [Installation](#installation) |
| 4 | [Development workflow](#development-workflow) |
| 5 | [Usage](#usage) |
| 6 | [Build & Deploy](#build--deploy) |
| 7 | [API Documentation](#api-documentation) |
| 8 | [Examples](#examples) |
| 9 | [Configuration](#configuration) |
|10 | [Contributing](#contributing) |
|11 | [License](#license) |
|12 | [Acknowledgements](#acknowledgements) |

---  

## 1. Overview  

Touropedia is a **responsive, single‑page travel guide** that showcases destinations, itineraries, and travel tips. The project is deliberately lightweight:  

* **HTML5** – semantic markup for accessibility and SEO.  
* **SCSS** – modular, themable styling compiled to a single CSS bundle.  
* **Vanilla JavaScript** – no framework overhead; ES6+ modules handle data fetching, UI interactions, and routing.  

The site works on any modern browser and adapts gracefully from mobile phones to large‑desktop monitors.

---  

## 2. Prerequisites  

| Tool | Minimum version | Why it’s needed |
|------|----------------|-----------------|
| **Git** | 2.20+ | Clone the repository |
| **Node.js** | 18.x LTS | Run npm scripts, compile SCSS, start dev server |
| **npm** (or **pnpm** / **yarn**) | 9.x+ | Package manager |
| **A modern browser** | Chrome/Firefox/Edge/Safari  latest | Test the UI |
| **Optional – Docker** | 20.10+ | Run the site in an isolated container |

> **Tip:** If you only want to view the static site, you can skip Node and just open `dist/index.html` in a browser. All build steps are optional for a quick demo.

---  

## 3. Installation  

```bash
# 1️⃣ Clone the repo
git clone https://github.com/your‑username/Touropedia.git
cd Touropedia

# 2️⃣ Install dependencies
npm ci   # or `pnpm i` / `yarn install`

# 3️⃣ (Optional) Install the optional pre‑commit hook
npm run prepare   # sets up husky lint‑staged hooks
```

> **What’s installed?**  
* `sass` – compiles SCSS → CSS.  
* `esbuild` – bundles JavaScript modules.  
* `live-server` – lightweight dev server with hot‑reload.  
* `eslint` + `stylelint` – code quality enforcement.  

---  

## 4. Development workflow  

| Command | Description |
|---------|-------------|
| `npm run dev` | Starts a live‑reloading dev server (`http://localhost:3000`). SCSS and JS are compiled on‑the‑fly. |
| `npm run build` | Produces a production‑ready `dist/` folder (minified CSS/JS, hashed filenames). |
| `npm run lint` | Runs ESLint + Stylelint. |
| `npm test` | Runs the (currently minimal) Jest test suite. |
| `npm run format` | Formats code with Prettier. |

**Typical cycle**

```bash
npm run dev          # start dev server
# edit src/**/*.scss or src/**/*.js
# browser reloads automatically
# when ready for a release:
npm run build
```

---  

## 5. Usage  

### 5.1 Running locally (development)

```bash
npm run dev
```

*Open* `http://localhost:3000` in your browser.  
The site will automatically reload when you edit any file under `src/`.

### 5.2 Viewing the production build

```bash
npm run build
npx serve dist   # or any static file server
```

Open the URL printed by `serve` (usually `http://localhost:5000`).  

### 5.3 Customising the theme  

All colours, fonts, and breakpoints live in `src/scss/_variables.scss`.  
Edit the variables and re‑run `npm run dev` (or `npm run build` for production).

```scss
// Example: change primary colour
$color-primary: #0066ff;   // default blue
```

### 5.4 Adding a new destination page  

1. **Create a markdown file** in `src/content/destinations/` (e.g., `paris.md`).  
2. Use the front‑matter format:

   ```markdown
   ---
   title: "Paris, France"
   slug: "paris"
   cover: "/assets/images/paris/cover.jpg"
   tags: ["city", "culture", "food"]
   ---
   # Welcome to Paris
   ...content...
   ```

3. Run `npm run dev`. The JavaScript router automatically picks up the new slug and renders the page using the built‑in markdown parser.

---  

## 6. Build & Deploy  

### 6.1 Production build  

```bash
npm run build
```

The command creates a **self‑contained** `dist/` folder:

```
dist/
├─ index.html
├─ assets/
│  ├─ css/
│  │  └─ main.[hash].css
│  ├─ js/
│  │  └─ bundle.[hash].js
│  └─ images/
└─ manifest.json
```

### 6.2 Deploying to static hosts  

Touropedia is a **static site** – you can host it on any static‑file CDN (Netlify, Vercel, GitHub Pages, Cloudflare Pages, etc.).  

**Example: Deploy to GitHub Pages**

```bash
# 1️⃣ Build
npm run build

# 2️⃣ Push the `dist` folder to the `gh-pages` branch
git checkout --orphan gh-pages
git --work-tree dist add --all
git --work-tree dist commit -m "Deploy Touropedia"
git push origin gh-pages --force
git checkout main
```

> **Tip:** Add a GitHub Action (`.github/workflows/deploy.yml`) that runs `npm ci && npm run build && gh-pages -d dist` for automatic CI/CD.

### 6.3 Docker (optional)

```dockerfile
# Dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY . .
RUN npm ci && npm run build

FROM nginx:stable-alpine
COPY --from=builder /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

```bash
docker build -t touropedia .
docker run -p 8080:80 touropedia
# Open http://localhost:8080
```

---  

## 7. API Documentation  

Touropedia ships with a **tiny client‑side API** that abstracts data fetching from the public travel‑info endpoint `https://api.touropedia.dev`. All calls return **JSON** and are typed with JSDoc.

### 7.1 Base URL  

```
https://api.touropedia.dev/v1
```

### 7.2 Endpoints  

| Method | Path | Description | Query parameters | Example response |
|--------|------|-------------|------------------|------------------|
| `GET` | `/destinations` | List all destinations (paginated). | `page` (int, default 1) `limit` (int, default 20) | `{ "data": [{ "id": "paris", "name": "Paris", "country": "France", "cover": "..."}], "meta": { "page":1, "totalPages":5 } }` |
| `GET` | `/destinations/:slug` | Get detailed info for a single destination. | – | `{ "id":"paris","name":"Paris","description":"…","highlights":[…],"gallery":[…] }` |
| `GET` | `/search` | Full‑text search across destinations, articles, and tips. | `q` (string, required) `type` (`dest|article|tip`) | `{ "results": [{ "type":"dest","slug":"paris","title":"Paris"}] }` |
| `GET` | `/weather/:city` | Current