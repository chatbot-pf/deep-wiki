---
description: Generate a GitHub Actions workflow to deploy the wiki site (VitePress, Nextra, …) to GitHub Pages, parameterized from the chosen adapter's manifest
---

# Deep Wiki: Deploy to GitHub Pages

Generate a `.github/workflows/deploy-wiki.yml` GitHub Actions workflow that builds and deploys the wiki to GitHub Pages. The workflow is **parameterized from the chosen adapter's manifest** (in `wiki-site-core`) — no tool-specific value is hardcoded.

## Step 1: Check Prerequisites & Resolve Tool

1. **Verify wiki exists**: Check that `wiki/` and `wiki/package.json` exist.
   - If not, tell the user: _"Run `/deep-wiki:build` first to scaffold the site."_
2. **Resolve the target tool** (same order as `build`): a `--tool <name>` argument, else the **default `vitepress`**. Load the adapter at `skills/wiki-site-core/references/adapters/<tool>.md` and read its **manifest**:
   - `build_cmd` — the build command (`npm run build` for vitepress, `next build` for nextra)
   - `output_dir` — artifact directory relative to `wiki/` (`.vitepress/dist` for vitepress, `out` for nextra)
   - `node_version` — Node major (20 for vitepress, 22 for nextra)
   - `base_path` — how to set the project-site base path (`kind: config-edit` / `next-config` / `env`)
   - `extra_files` — files to emit before upload (e.g. `.nojekyll`)
3. **Check for existing deployment workflows**:
   ```bash
   ls .github/workflows/deploy-wiki.yml 2>/dev/null
   grep -rl "deploy-pages\|pages-artifact\|github-pages" .github/workflows/ 2>/dev/null
   ```
   - If `deploy-wiki.yml` exists → **STOP**. _"A deployment workflow already exists at `.github/workflows/deploy-wiki.yml`. No changes needed."_
   - If a DIFFERENT pages workflow exists → **ASK** whether to skip or create alongside it.
   - If none → proceed.

## Step 2: Generate Workflow File

Create `.github/workflows/deploy-wiki.yml`, substituting `{node_version}`, `{build_cmd}`, `{output_dir}` from the manifest, and adding an "Emit extra files" step only when `extra_files` is non-empty:

```yaml
name: Deploy Wiki to GitHub Pages

on:
  push:
    branches: [main]
    paths:
      - 'wiki/**'
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: {node_version}        # from manifest
          cache: npm
          cache-dependency-path: wiki/package-lock.json

      - name: Setup Pages
        uses: actions/configure-pages@v5

      - name: Install dependencies
        run: npm ci
        working-directory: wiki

      - name: Build
        run: {build_cmd}                       # from manifest
        working-directory: wiki

      # Only when manifest extra_files is non-empty — e.g. nextra needs .nojekyll:
      - name: Emit extra files
        run: touch {output_dir}/.nojekyll
        working-directory: wiki

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: wiki/{output_dir}              # from manifest

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

**Default invariant (STOP):** with no `--tool` (vitepress), the manifest yields `node_version: 20`, `build_cmd: npm run build`, `output_dir: .vitepress/dist`, and empty `extra_files` — so the generated workflow is byte-equivalent to the pre-refactor VitePress workflow. A changed default would break existing users' Pages deploys.

### Workflow Details

| Setting | Source | Why |
|---------|--------|-----|
| **Node version** | manifest `node_version` | VitePress 1.x → 20; Nextra v4 → 22 |
| **Build command** | manifest `build_cmd` | `npm run build` (vitepress) / `next build` (nextra) |
| **Artifact path** | `wiki/` + manifest `output_dir` | `.vitepress/dist` (vitepress) / `out` (nextra) |
| **Extra files** | manifest `extra_files` | e.g. `.nojekyll` so Next/Astro `_next` assets are served |
| **Trigger / concurrency / permissions** | fixed | Same across tools |

## Step 3: Set the Base Path (from manifest `base_path`)

GitHub Pages serves a project site from `/<repo-name>/`. Detect the case:

```bash
REPO_NAME=$(basename $(git remote get-url origin) .git)
OWNER=$(git remote get-url origin | sed -E 's|.*[:/]([^/]+)/[^/]+\.git|\\1|')
```

If the repo is NOT `{owner}.github.io`, inject the base path per the manifest's `base_path.kind`:

- **`config-edit`** (VitePress): set `base: '/{repo-name}/'` in `.vitepress/config.mts`.
- **`next-config`** (Nextra): set `basePath: '/{repo-name}'` and `assetPrefix: '/{repo-name}/'` in `next.config.mjs`.
- **`env`**: export the manifest's `base_path.key` env var (e.g. `NUXT_APP_BASE_URL=/{repo-name}/`) in the build step.

## Step 4: Generate package-lock.json

If `wiki/package-lock.json` is missing (required for `npm ci`):

```bash
cd wiki && npm install
```

Remind the user to commit it.

## Step 5: Commit and Report

Report the created workflow, any config/`next.config` base-path edit, the resolved tool, and the user actions: commit the workflow + lockfile, and enable **Settings → Pages → Source → GitHub Actions**. The site goes live at `https://{owner}.github.io/{repo-name}/` (project site) or `https://{owner}.github.io/` (user/org site).

## Troubleshooting

| Issue | Solution |
|-------|----------|
| 404 on deployed site | Verify the base path matches the repo name (per `base_path.kind`) |
| Build fails on `npm ci` | Ensure `wiki/package-lock.json` is committed |
| Pages not enabled | Settings → Pages → Source → "GitHub Actions" |
| Next/Astro assets 404 | Ensure `.nojekyll` is emitted (manifest `extra_files`) |
| Mermaid diagrams missing | Confirm the adapter's Mermaid wiring built (vitepress: `vitepress-plugin-mermaid`; nextra: `@theguild/remark-mermaid`) |

$ARGUMENTS
