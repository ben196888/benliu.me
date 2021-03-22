---
title: How my blog works - (1)
description: introduction to the architech on this website
date: '2021-03-10'
tags: [11ty, github, github-action, github-pages, gh-pages, hackmd]
---

It's the first post of my blog and I would like to explain how my blog works from the bottom to the top with github-eco-system.

I would start from the static website generator [11ty](https://www.11ty.dev/) with theme template Eleventy Duo. Then I would explain the deployment on github page with simple github action. Last, I would end up on using hackmd to add/update my blog posts.

This project, surrounds github eco-system, bases and only bases on github repository, gh-page, and github-actions.


## Use eleventy-duo template

First, use the [eleventy-duo template](https://github.com/yinkakun/eleventy-duo) to kick-off the repository. It's a minimal, clean and neat blog template. You can simply click the `Use this template` button on the top right of eleventy-duo github repo.

Cool, you can install the node dependencies and start the dev server after you cloned your project. 

```shell
yarn && yarn dev
```

## Setup github action


This section introduces you the github action and ship the website to github-pages. Eleventy duo is using netlify to integration, deployment and hosting the website as recommended. You can skip this section if you would like to use netlify to deploy and hosting. 

### Setup github page publish scripts

I recommend to deploy your static page to github pages by `gh-pages` script. It makes your life easier keep the same scripts on CI and local, deploy static page in one line, and simply tweak the advanced options. Adding two lines of scripts in the `package.json` then we are ready to move on.

Note: You can replace this step with a pure [github page deployment action](https://github.com/peaceiris/actions-gh-pages). Don't forget to build the static page before publishing it.

```shell
yarn add -D gh-pages
```

```json
// package.json
// adding two scripts
"scripts": {
    "predeploy": "yarn build",
    "deploy": "gh-pages -d public"
}
```

### Setup github action deploy workflow

"As a tech blogger, I like to let the workflow take over most of my manual work so that I can more focus on writing new posts." Yeah, this is exactly what I want when I setup this blog on gh-pages. Therefore, I draw an flow chart to review my idea. I reckon it's decent enough for now. You should setup the auto deployment when you have your own blog.

```mermaid
graph LR
    P[Push] -->|Install dependency| B[Build]
    B --> C{master branch commit}
    C -->|YES| D[Deploy to GH-Pages]
    C -->|NO| E(Do Nothing)
```

After the diagram, converting the workflow diagram to be a read workflow.

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches-ignore:
      - "gh-pages"

env:
  node_version: "14.x"

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Setup Node.js
        uses: actions/setup-node@v1
        with:
          node-version: ${{ env.node_version }}
      - uses: c-hive/gha-yarn-cache@v1
      - name: Install JS dependencies
        run: yarn install
      - name: build
        run: yarn build
      - name: publish
        if: github.ref == 'refs/heads/master'
        run: |
          git remote set-url origin https://git:${GITHUB_TOKEN}@github.com/${GITHUB_REPOSITORY}.git
          yarn deploy --user "github-actions-bot <support+actions@github.com>"
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

```

Alright, everything on the infrastructure are set. The github actions a.k.a. workflow takes care of the build and publish the static pages to github pages.

I would explain how do I add/update the posts without forestry in the next article.

Thanks for reading.