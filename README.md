# Blog.Sabourau.LT

[![Github Workflow](<https://github.com/limezest/eleventy-blog-template/workflows/Blog%20build%20(main)/badge.svg?branch=main>)](https://github.com/limezest/eleventy-blog-template/actions?query=workflow%3A%22Blog+build+%28main%29%22) [![Netlify Status](https://api.netlify.com/api/v1/badges/5d47157f-8911-4215-811e-ee3ec24cfe2f/deploy-status)](https://app.netlify.com/sites/eleventy-blog-template/deploys)

You can check out the live version at <https://blog.sabourau.lt>.

If you want to start your own on Netlify, there's a button for it: [![Deploy to Netlify](https://www.netlify.com/img/deploy/button.svg)](https://app.netlify.com/start/deploy?repository=https://github.com/limezest/eleventy-blog-template)

## Features

-   💯 on Lighthouse
-   🔆 and 🌛 mode
-   🎯 SEO and OpenGraph optimized
-   🌄 Responsive images optimization
-   👀 Accessible
-   🛠 JavaScript and CSS build optimization
-   👨‍💻 Prism-based syntax highlighting
-   📚 RSS (yup, still a thing), sitemap.xml, and JSON-LD
-   🔍 [Algolia Search](https://github.com/algolia/algoliasearch-netlify) enabled
-   and more

Opinionated setup with [Prettier](https://prettier.io/), [ESlint](https://eslint.org/), [markdownlint](https://github.com/DavidAnson/markdownlint) and others. UX build with [Nunjucks](https://mozilla.github.io/nunjucks/templating.html) and [TailwindCSS](https://tailwindcss.com/docs). JavaScript bundled with [Rollup](https://rollupjs.org/).

## Configuration

All blog configuration is handled via [`siteconfig.js`](./content/_data/siteconfig.js). Everything is inline documented.

## Deployment

All build processes rely on how `NODE_ENV` is set. For production builds, which then also means minified CSS and JS you've to set the value to `production`. I mention this explicitly as this is for some vendors not the default.

If you want to speed up your build times a bit you can add the generated images to your git repo. The `.gitignore` already contains a commented section for that.

## Local Development

### Before you install dependencies

This repo uses [Volta](https://volta.sh/). Get it, and it'll make your node life so much easier.

### Instructions

Clone this repository.

```zsh
git clone https://github.com/limezest/eleventy-blog-template
```

Change into the cloned directory.

```zsh
cd eleventy-blog-template
```

Install dependencies. Note, if you prefer `npm` over `yarn` make sure to first remove the `yarn.lock` file, and then run `npm install`.

```zsh
yarn install
```

Start the local development process.

```zsh
yarn dev
```

Open the page, usually on <http://localhost:8080>, and dig around!

Be sure to run Github Actions checks locally before pushing:

```zsh
yarn precommit
```

## Credits

Bootstrapped from <https://github.com/muenzpraeger/eleventy-chirpy-blog-template>
