# Deploy React app to GitHub Pages

This guide explains how to set up and deploy a Create React App (CRA) project to GitHub Pages, including a GitHub Actions workflow similar to the one used in this repository.

The steps assume you already have a repository on GitHub and a CRA app in a subfolder (this repo uses `newice/`). Adjust paths if your app is at the repo root.

## Pre-requisites

- Node.js and npm installed locally
- A GitHub repository where you can push code
- Basic git knowledge

## 1) Project setup (Create React App)

If you don't yet have a CRA app, create one inside the repo:

```bash
cd <repo-root>
npx create-react-app newice --template typescript
cd newice
```

## 2) Add Tailwind CSS (optional, but used in NewIce)

Install Tailwind and PostCSS tooling, then configure: 

```bash
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

Update `tailwind.config.js` to scan your sources:

```js
module.exports = {
  content: ['./public/index.html', './src/**/*.{js,jsx,ts,tsx}'],
  theme: { extend: {} },
  plugins: []
}
```

In `src/index.css` add:

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

Import that CSS from `src/index.js`/`src/index.tsx`:

```js
import './index.css'
```

CRA v5 manages PostCSS internally; avoid adding a conflicting `postcss.config.js` unless you know what you're doing.

## 3) Configure `package.json` for GitHub Pages

- Set the `homepage` field to your GitHub Pages URL. Example:

```json
"homepage": "https://<your-github-username>.github.io/<repo-name>"
```

- Install `gh-pages` as a dev dependency and add deploy scripts:
# Deploy a Create React App (CRA) to GitHub Pages

This generic guide shows how to deploy a Create React App project to GitHub Pages and how to automate deployment using GitHub Actions. Replace `your-app` with your app folder (or `.` if the app is in the repository root).

## Prerequisites

- Node.js and npm installed locally
- A GitHub repository
- Basic git knowledge

## 1) Create or locate your CRA app

If you need a new app inside the repo:

```bash
cd <repo-root>
npx create-react-app your-app --template typescript
cd your-app
```

If your app lives at the repository root, skip the `your-app` folder steps and run commands from the repo root.

## 2) (Optional) Add Tailwind CSS

To add Tailwind:

```bash
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

Configure `tailwind.config.js` to scan your files:

```js
module.exports = {
  content: ['./public/index.html', './src/**/*.{js,jsx,ts,tsx}'],
  theme: { extend: {} },
  plugins: []
}
```

Add the Tailwind directives to `src/index.css`:

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

Import that file from `src/index.js`/`src/index.tsx`:

```js
import './index.css'
```

Note: CRA v5 manages PostCSS internally. Avoid adding a conflicting `postcss.config.js` unless required.

## 3) Configure `package.json` for GitHub Pages

1. Set the `homepage` field to your GitHub Pages URL:

```json
"homepage": "https://<your-github-username>.github.io/<repo-name>"
```

2. Install `gh-pages` and add deploy scripts:

```bash
npm install -D gh-pages
```

Add these scripts to `package.json`:

```json
"scripts": {
  "predeploy": "npm run build",
  "deploy": "gh-pages -d build"
}
```

Running `npm run deploy` will build and publish the `build/` directory to the `gh-pages` branch.

## 4) Test a local deploy

```bash
# from the app folder
npm run build
npm run deploy
```

You should see `Published` output from the `gh-pages` tool and find the site at the `homepage` URL.

## 5) Automate with GitHub Actions

Create a workflow file at `.github/workflows/deploy.yml` to build and publish on push to `main`. Example workflow (replace `your-app` with your folder or `.`):

```yaml
name: Deploy Webpage

on:
  push:
    branches: [ main ]
  workflow_dispatch:

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pages: write
      id-token: write

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: '22'

      - name: Install dependencies
        run: npm install
        working-directory: ./your-app

      - name: Build
        run: npm run build
        working-directory: ./your-app

      - name: Deploy to gh-pages
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          npx gh-pages -d build -b gh-pages --repo "https://x-access-token:${{ secrets.GITHUB_TOKEN }}@github.com/${{ github.repository }}.git"
        working-directory: ./your-app
```

Notes:

- This workflow uses `npm install`. `npm ci` is preferred for reproducible installs but requires a committed, up-to-date `package-lock.json`.
- `GITHUB_TOKEN` is automatically available inside GitHub Actions and allows pushing to the repo.

## 6) PUBLIC_URL and `homepage` pitfalls

- CRA uses the `homepage` field to prefix asset paths. If your site is served under `/repo-name/`, the `homepage` must match that path or assets will 404.
- If you see missing CSS/JavaScript in production, inspect `build/index.html` for incorrect paths and check `package.json` `homepage`.

## 7) Fixing `npm ci` lockfile errors

If CI fails with an error like:

```
npm ci can only install packages when your package.json and package-lock.json or npm-shrinkwrap.json are in sync.
```

Update the lockfile locally (from the app folder):

```bash
npm install
# or update only the lockfile
npm install --package-lock-only
```

Commit the updated `package-lock.json` and push.

## 8) Troubleshooting & tips

- Clear browser cache if changes don't appear after deploy.
- Ensure the `homepage` path uses the same casing as your repository name.
- For non-CRA setups, the process is similar: build to a `build`/`dist` folder and publish that directory to `gh-pages`.

## 9) Alternative: Use `peaceiris/actions-gh-pages`

Instead of `npx gh-pages`, you can use the `peaceiris/actions-gh-pages` action to publish the build directory. Example step (run after the build):

```yaml
- name: Deploy
  uses: peaceiris/actions-gh-pages@v3
  with:
    github_token: ${{ secrets.GITHUB_TOKEN }}
    publish_dir: ./your-app/build
    publish_branch: gh-pages
```

This action expects `publish_dir` to exist (build step must run first).

---

If you want, I can add a tailored workflow to your repo and open a PR, or update your `package.json` `homepage` and commit a refreshed `package-lock.json`.
