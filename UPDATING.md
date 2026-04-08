# Updating ken.horovatin.net

This site was originally created with **R Blogdown in RStudio** and uses **Hugo** as the static site generator. The source repo is [`khorovatin/personal-website`](https://github.com/khorovatin/personal-website), and the built site is published from [`khorovatin/khorovatin.github.io`](https://github.com/khorovatin/khorovatin.github.io).

Recommended workflow:

- Use **RStudio + Blogdown** for editing and local preview
- Use **Git + GitHub** for version control
- Use **GitHub Actions** to build and deploy automatically after each push

For a shorter publishing-only checklist, see [DEPLOYING.md](./DEPLOYING.md).

---

## Repository layout

| Repo | Purpose |
|---|---|
| `khorovatin/personal-website` | Source repo: content, config, theme, and project files |
| `khorovatin/khorovatin.github.io` | Published output repo: generated HTML, CSS, JS, and assets served by GitHub Pages |

Within `personal-website`, the key paths are:

| Path | Purpose |
|---|---|
| `content/` | Markdown content for pages, projects, publications, and home sections |
| `config.toml` | Main Hugo site config |
| `config/_default/` | Additional Hugo config files |
| `themes/hugo-academic/` | Vendored Academic theme used by the site |
| `public/` | Built output from the older manual workflow |
| `personal-website.Rproj` | RStudio project file |

---

## Local setup in RStudio

### Prerequisites

Install these tools on your Mac:

- **R**
- **RStudio**
- R packages: **blogdown** and **servr**
- **Hugo**, preferably installed via `blogdown::install_hugo()`

### Recommended: install Hugo from RStudio with Blogdown

For this site, prefer installing Hugo from within RStudio using Blogdown. This keeps the Hugo version used by Blogdown more predictable, which is especially helpful for older Blogdown/Hugo Academic sites.

Run this in the R console:

```r
install.packages(c("blogdown", "servr"))

blogdown::install_hugo()
blogdown::find_hugo()
blogdown::hugo_version()
```

If you need a specific Hugo version later, Blogdown also supports installing a chosen version.

### Optional: install Hugo with Homebrew

If you also use Hugo outside RStudio, you can install it system-wide with Homebrew:

```bash
brew install hugo
hugo version
```

However, Blogdown documentation notes that on macOS, Homebrew-based Hugo installs can be less stable for long-lived Blogdown sites because package upgrades may move Hugo to a newer version unexpectedly.

### Open the project

1. Clone the repository if you have not already:

```bash
git clone https://github.com/khorovatin/personal-website.git
cd personal-website
```

2. Open `personal-website.Rproj` in **RStudio**.

---

## Editing content

Most day-to-day edits happen in `content/`.

| What to edit | File path |
|---|---|
| About / profile | `content/authors/admin/_index.md` |
| Skills section | `content/home/skills.md` |
| Experience section | `content/home/experience.md` |
| Projects | `content/project/` |
| Publications | `content/publication/` |
| Site title / base URL / global settings | `config.toml` |

You can also update settings in `config/_default/` if needed.

---

## Previewing locally with Blogdown

From the R console in RStudio, run:

```r
blogdown::serve_site()
```

This starts a local preview server and automatically reloads changes as you edit files.

Useful alternatives:

```r
blogdown::build_site()
blogdown::stop_server()
blogdown::find_hugo()
blogdown::hugo_version()
```

Notes:

- `blogdown::serve_site()` is the easiest day-to-day workflow in RStudio
- Under the hood, Blogdown calls Hugo to render the site
- If something fails locally, first check `blogdown::find_hugo()` and `blogdown::hugo_version()`

---

## Publishing changes

Once the deploy workflow is set up, publishing should be simple:

1. Make your edits in RStudio
2. Preview with `blogdown::serve_site()`
3. Commit your changes:

```bash
git add .
git commit -m "Update site content"
git push origin master
```

4. GitHub Actions builds the site with Hugo
5. GitHub Actions pushes the generated output to `khorovatin/khorovatin.github.io`
6. GitHub Pages serves the updated site at `https://ken.horovatin.net/`

After this is working, you should no longer need to manually build and push the `public/` folder yourself.

---

## GitHub Actions deploy setup

Create `.github/workflows/deploy.yml` in `khorovatin/personal-website` with this content:

```yaml
name: Build and Deploy

on:
  push:
    branches:
      - master
  workflow_dispatch:

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout source
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v3
        with:
          hugo-version: 'latest'
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

### Required GitHub secret

The workflow needs a Personal Access Token so it can push to the output repo.

1. Go to `https://github.com/settings/tokens`
2. Generate a **classic** token
3. Name it something like `personal-website-deploy`
4. Give it the `repo` scope
5. Copy the token
6. In `personal-website`, go to **Settings > Secrets and variables > Actions**
7. Create a new repository secret named `DEPLOY_TOKEN`
8. Paste the token value and save

---

## Manual fallback workflow

If GitHub Actions is unavailable, you can still build manually.

### Option 1: Build from RStudio / R

```r
blogdown::build_site()
```

### Option 2: Build from the terminal

```bash
hugo --minify
```

Then publish the generated `public/` directory to `khorovatin.github.io` manually.

This should be considered a fallback only.

---

## Updating the theme

The site currently uses the vendored `hugo-academic` theme in `themes/hugo-academic/`. This theme is old and may require a larger migration if you want to move to the modern HugoBlox successor.

### Minor changes

For small tweaks, you can continue editing the current theme files directly.

### Major upgrade path

For a larger modernization:

1. Back up your content in `content/`
2. Create a fresh site using the modern Academic / HugoBlox starter
3. Migrate content into the new structure
4. Test locally in RStudio or with Hugo
5. Replace the old site once the new one is working

Because the theme structure changed significantly over time, a fresh migration is usually easier than trying to patch the old theme in place.

---

## Troubleshooting

| Problem | What to check |
|---|---|
| `blogdown::serve_site()` fails | Confirm `blogdown` is installed and run `blogdown::find_hugo()` |
| Wrong Hugo version | Run `blogdown::hugo_version()` |
| `hugo` command not found | Install Hugo with `blogdown::install_hugo()` or Homebrew |
| CSS / SCSS problems | Confirm Hugo is the Extended version |
| Deploy fails in GitHub Actions | Check the workflow log and confirm `DEPLOY_TOKEN` exists |
| Site updates locally but not live | Wait a minute, then check the Actions run and the output repo |
| Custom domain disappears | Confirm the workflow still sets `cname: ken.horovatin.net` |
| Homebrew updated Hugo unexpectedly | Reinstall Hugo with `blogdown::install_hugo()` and verify the version |

---

## Quick summary

- Edit and preview locally in **RStudio with Blogdown**
- Install Hugo with **`blogdown::install_hugo()`** unless you have a specific reason to use Homebrew
- Push changes to `personal-website`
- Let **GitHub Actions** build with Hugo and publish to `khorovatin.github.io`
- Keep the old manual `public/` workflow only as a fallback