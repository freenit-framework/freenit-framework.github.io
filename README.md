# Freenit documentation

This repository is prepared for publishing on GitHub Pages with MkDocs.

## Local development

Install the MkDocs dependencies and run:

```sh
mkdocs serve
```

To validate the generated site locally:

```sh
mkdocs build --strict
```

## GitHub Pages deployment

This repository is named `freenit-framework.github.io`, so it behaves like a GitHub user/organization Pages repository.

The MkDocs deployment guide explains that user/organization Pages deployments with `mkdocs gh-deploy` are intended to publish into a dedicated Pages repository/branch separate from the source repository. Because this repository stores the MkDocs source on `master`, deploying the built site back into that same branch with `mkdocs gh-deploy` would be the wrong workflow here.

Instead, publishing is handled by GitHub Actions in [`.github/workflows/pages.yml`](/home/meka/repos/freenit-framework.github.io/.github/workflows/pages.yml). On each push to `master`, the workflow:

1. installs MkDocs and the required plugins,
2. checks out and builds the `freenit-framework/designer` repository,
3. builds the site with `mkdocs build --strict`,
4. copies the designer static build into `site/design/`,
5. deploys the generated `site/` output to GitHub Pages.

## One-time GitHub configuration

In the GitHub repository settings, set Pages to use `GitHub Actions` as the source.

## SEO improvements

The site now includes:

1. stable canonical URLs derived from `site_url`,
2. a global meta description with page-level overrides,
3. Open Graph and Twitter card metadata,
4. structured data for the site,
5. a `robots.txt` file for crawling guidance.

Something
