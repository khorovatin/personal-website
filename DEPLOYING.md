# Deploying the Site

This site was created with **R Blogdown in RStudio**. Use **RStudio + Blogdown** for editing and previewing, then let **GitHub Actions** deploy the site after you push changes.

For fuller maintenance notes, see [UPDATING.md](./UPDATING.md).

## Normal workflow

1. Open `personal-website.Rproj` in **RStudio**
2. Preview locally with:

```r
blogdown::serve_site()
```

3. Commit and push your changes to `master`
4. GitHub Actions builds the site with Hugo
5. The workflow publishes the built site to `khorovatin.github.io`
6. GitHub Pages serves the updated site at `https://ken.horovatin.net/`

## One-time setup

Create `.github/workflows/deploy.yml` in this repo:

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

Then create a GitHub Personal Access Token with `repo` scope and save it in this repo as the `DEPLOY_TOKEN` Actions secret.

## Hugo installation note

For this site, prefer installing Hugo from RStudio with:

```r
blogdown::install_hugo()
```

This is usually safer for older Blogdown sites than relying on Homebrew upgrades.

## Notes

- You can keep using **RStudio + Blogdown** for editing and previewing
- Deployment does **not** need to run from RStudio
- The checked-in `public/` folder reflects the older manual workflow and should no longer need to be updated by hand once Actions is working