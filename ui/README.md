# Leadfreak Utils — Static UI

A single-file HTML/JS UI for the Leadfreak Utils FastAPI backend deployed on Railway. No build step, no framework — just open `index.html` and it works.

## What it covers

- **Job Search** (`/api/v1/job-search` + `/api/v1/job-search/csv`) — LinkedIn or ATS career sites, with filters and CSV download
- **Suggest Titles** (`/api/v1/suggest-titles`) — resume upload + form fields, Groq LLM
- **Estimate Salary** (`/api/v1/estimate-salary`) — batch up to 10 jobs
- **LinkedIn Comments** (`/api/v1/generate-comments`) — paste post, get classified comments

The advanced multi-source search (`/api/v1/advanced-job-search`) is intentionally not exposed — use Swagger docs at `/docs` for that.

## Local preview

```bash
cd ui
python3 -m http.server 8000
# open http://localhost:8000
```

## Deploy to GitHub Pages

### Option A — repo subfolder (simplest)

1. Push this `ui/` folder to a public GitHub repo.
2. Repo → **Settings → Pages**.
3. **Source:** "Deploy from a branch", **Branch:** `main`, **Folder:** `/ui`.
4. Save. Pages publishes at `https://<user>.github.io/<repo>/`.

### Option B — dedicated `gh-pages` branch

```bash
git checkout --orphan gh-pages
git rm -rf .
cp ui/index.html .
git add index.html
git commit -m "Publish UI"
git push -u origin gh-pages
```

Then set Pages source to the `gh-pages` branch, root folder.

### Option C — GitHub Actions (auto-deploy on push)

Drop this at `.github/workflows/pages.yml`:

```yaml
name: Deploy UI to Pages
on:
  push:
    branches: [main]
    paths: ['ui/**']
permissions:
  pages: write
  id-token: write
concurrency:
  group: pages
  cancel-in-progress: true
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - uses: actions/checkout@v4
      - uses: actions/upload-pages-artifact@v3
        with:
          path: ui
      - id: deployment
        uses: actions/deploy-pages@v4
```

Then in **Settings → Pages**, set source to **GitHub Actions**.

## Configuration

The header has an editable **API URL** field (default points at the Railway production URL). Anything users type is saved to `localStorage`, so each teammate only configures it once.

Optional API keys can be entered in the collapsible "API keys" section at the top — they're sent as headers (`X-RapidAPI-Key`, `X-Groq-Api-Key`, etc.). Leave blank to use the env vars configured on Railway.

## Notes on auth & exposure

- The Railway backend has `CORSMiddleware` set to `allow_origins=["*"]`, so any origin (including GitHub Pages) can call it.
- There's no auth layer in front — anyone who finds the Railway URL can hit the API. If that becomes a problem, add a simple shared-secret header check in `app/middleware/security.py`.
- API keys typed into the UI live only in the user's browser (localStorage). They're never sent anywhere except as request headers to the Railway backend.
