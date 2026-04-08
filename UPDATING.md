# Updating ken.horovatin.net

This document describes the complete workflow for editing content and publishing the site at [ken.horovatin.net](https://ken.horovatin.net).

---

## Architecture Overview

| Repo | Purpose |
|---|---|
| [`khorovatin/personal-website`](https://github.com/khorovatin/personal-website) | **Source repo** — Hugo project with Markdown content, config, and theme |
| [`khorovatin/khorovatin.github.io`](https://github.com/khorovatin/khorovatin.github.io) | **Published output repo** — pre-built HTML pushed here, served by GitHub Pages at the custom domain |

The source repo builds the site with [Hugo](https://gohugo.io/) and the theme is located in `themes/hugo-academic/`. A GitHub Actions workflow (see below) automatically builds the site and pushes the output to the `.github.io` repo on every push to `master`.

---

## Prerequisites (Local Development)

Install these tools on your Mac:

```bash
# Install Hugo (extended version required for SCSS)
brew install hugo

# Verify
hugo version
# Should output: hugo v0.xxx.x+extended ...
```

> **Note:** The site uses Hugo Extended (for SCSS compilation). Always install the `extended` variant. On macOS via Homebrew this is the default.

---

## Day-to-Day: Editing Content

All editable content lives in the `content/` directory. Files are written in Markdown with YAML front matter.

### 1. Clone the source repo (first time only)

```bash
git clone https://github.com/khorovatin/personal-website.git
cd personal-website
```

### 2. Start the local preview server

```bash
hugo server -D
```

Open [http://localhost:1313](http://localhost:1313) in your browser. The page live-reloads whenever you save a file.

### 3. Edit content

Common content files:

| What to edit | File path |
|---|---|
| About / biography | `content/authors/admin/_index.md` |
| Skills | `content/home/skills.md` |
| Experience (work history) | `content/home/experience.md` |
| Projects | `content/project/` (one `.md` file per project) |
| Publications | `content/publication/` |
| Site-wide config (title, links) | `config.toml` + `config/_default/` |

### 4. Commit and push

```bash
git add .
git commit -m "Update skills section"
git push origin master
```

After pushing, the GitHub Actions workflow (see below) automatically builds the site and deploys it to `khorovatin.github.io`. **You do not need to manually build or push to the `.github.io` repo.**

---

## GitHub Actions: Automated Build & Deploy

A workflow file at `.github/workflows/deploy.yml` handles the build and deploy automatically. Here is what it does:

1. Triggers on every push to `master` in this repo
2. Checks out the source repo
3. Installs Hugo Extended
4. Runs `hugo --minify` to build the static site into the `public/` directory
5. Force-pushes the contents of `public/` to the `master` branch of `khorovatin/khorovatin.github.io`

### Setting up the workflow (one-time setup)

#### Step 1: Create a Personal Access Token (PAT)

The Actions workflow needs permission to push to the `.github.io` repo.

1. Go to [github.com/settings/tokens](https://github.com/settings/tokens)
2. Click **Generate new token (classic)**
3. Give it a descriptive name: `personal-website-deploy`
4. Set expiration to **No expiration** (or rotate annually)
5. Select scope: **`repo`** (full control of private repositories)
6. Click **Generate token** and copy the value immediately

#### Step 2: Add the token as a repository secret

1. Go to [github.com/khorovatin/personal-website/settings/secrets/actions](https://github.com/khorovatin/personal-website/settings/secrets/actions)
2. Click **New repository secret**
3. Name: `DEPLOY_TOKEN`
4. Value: paste the PAT you generated
5. Click **Add secret**

#### Step 3: Create the workflow file

Create `.github/workflows/deploy.yml` in this repo with the following content:

```yaml
name: Build and Deploy

on:
  push:
    branches:
      - master

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout source
        uses: actions/checkout@v4
        with:
          submodules: false
          fetch-depth: 0

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v3
        with:
          hugo-version: "latest"
          extended: true

      - name: Build site
        run: hugo --minify

      - name: Deploy to khorovatin.github.io
        uses: peaceiris/actions-gh-pages@v4
        with:
          personal_token: ${{ secrets.DEPLOY_TOKEN }}
          external_repository: khorovatin/khorovatin.github.io
          publish_branch: master
          publish_dir: ./public
          cname: ken.horovatin.net
```

> **Important:** The `cname` line ensures the `CNAME` file (needed for your custom domain) is preserved in the output repo on every deploy.

#### Step 4: Commit the workflow and push

```bash
git add .github/workflows/deploy.yml
git commit -m "Add GitHub Actions deploy workflow"
git push origin master
```

After this push, watch the Actions tab at [github.com/khorovatin/personal-website/actions](https://github.com/khorovatin/personal-website/actions) to confirm the first deploy succeeds.

---

## Manual Build & Deploy (if CI is unavailable)

If you need to build and publish manually without GitHub Actions:

```bash
# 1. Build the site
hugo --minify

# 2. Go into the built output
cd public

# 3. Push to the published repo (first time: init the remote)
git init
git remote add origin https://github.com/khorovatin/khorovatin.github.io.git
git add .
git commit -m "Publish site $(date '+%Y-%m-%d')"
git push --force origin master
```

> After a manual deploy, make sure the `CNAME` file exists in the root of `khorovatin.github.io` containing `ken.horovatin.net` — otherwise GitHub Pages will drop the custom domain.

---

## Updating the Theme

The theme is vendored in `themes/hugo-academic/`. It was last updated circa 2019 (hugo-academic v4).

### Check the current theme version

```bash
cat themes/hugo-academic/theme.toml | grep version
```

### Upgrading to a newer theme version

The `hugo-academic` theme has since been renamed and restructured as [HugoBlox](https://hugoblox.com/). A direct upgrade requires care because the config format changed significantly in v5+.

**Recommended approach for a major upgrade:**

1. Back up your `content/` directory — your actual content is portable
2. Create a fresh Hugo + HugoBlox project:
   ```bash
   git clone https://github.com/HugoBlox/theme-academic-cv.git new-site
   cd new-site
   hugo server
   ```
3. Copy your content files from the old `content/` into the new structure, adapting front matter as needed
4. Test locally, then replace this repo's contents

**Minor patch updates (staying on v4 theme):** You can manually copy updated files from the [hugo-academic v4 release](https://github.com/gcushen/hugo-academic/releases) into `themes/hugo-academic/`.

---

## Troubleshooting

| Problem | Solution |
|---|---|
| `hugo: command not found` | Run `brew install hugo` |
| Site builds but CSS is broken | Make sure you installed Hugo **Extended** (`hugo version` should include `+extended`) |
| GitHub Actions deploy fails with 403 | The `DEPLOY_TOKEN` secret may be missing or expired — regenerate it (see Setup above) |
| Custom domain disappears after deploy | Make sure the `cname: ken.horovatin.net` line is in the workflow file |
| Changes visible locally but not live | Check the Actions tab for build errors; allow 1-2 minutes for GitHub Pages CDN propagation |

---

## Repository Reference

- **Source repo:** [github.com/khorovatin/personal-website](https://github.com/khorovatin/personal-website)
- **Published repo:** [github.com/khorovatin/khorovatin.github.io](https://github.com/khorovatin/khorovatin.github.io)
- **Live site:** [ken.horovatin.net](https://ken.horovatin.net)
- **Hugo docs:** [gohugo.io/documentation](https://gohugo.io/documentation/)
- **HugoBlox (modern Academic theme):** [hugoblox.com](https://hugoblox.com)
