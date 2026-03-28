# yangyao.github.io

This repository contains the source for my personal blog at:

- https://blog.phpman.top/

The site is built with Hexo and deployed through GitHub Pages.

## Local development

Install dependencies:

```bash
npm install
```

Start the local server:

```bash
npm run start
```

Build the static site:

```bash
npm run build
```

## Deployment

The site is deployed automatically from the `hexo` branch via GitHub Actions and GitHub Pages.

## Structure

- `source/_posts/`: blog posts
- `source/about/`: about page
- `themes/even/`: current theme
- `.github/workflows/pages.yml`: Pages workflow
