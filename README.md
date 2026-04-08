# personal-website

Source repository for [ken.horovatin.net](https://ken.horovatin.net/).

This site was originally created with **R Blogdown in RStudio** and uses **Hugo** with the legacy **hugo-academic** theme. The source content lives in this repository, and the generated static site is published to [`khorovatin/khorovatin.github.io`](https://github.com/khorovatin/khorovatin.github.io).

## Repositories

- `khorovatin/personal-website` — source repo
- `khorovatin/khorovatin.github.io` — published output repo served by GitHub Pages

## Workflow

- Edit content locally in **RStudio**
- Preview locally with **Blogdown**
- Commit and push to `master`
- GitHub Actions builds the site with **Hugo 0.68.3**
- GitHub Actions deploys the generated `public/` output to `khorovatin.github.io`

## Local development

Open the project in RStudio and install the required R packages:

```r
install.packages(c("blogdown", "servr"))
blogdown::install_hugo(version = "0.68.3")
blogdown::find_hugo()
blogdown::hugo_version()
```

Preview the site locally:

```r
blogdown::serve_site()
```

## Documentation

- [UPDATING.md](./UPDATING.md) — full maintenance and editing guide
- [DEPLOYING.md](./DEPLOYING.md) — short deployment checklist

## Notes

This repository still contains the historical `public/` folder from the older manual publishing workflow. Once GitHub Actions deployment is fully validated, that folder should no longer need to be updated manually.