---
title: Deploying This Site
date: 2023-06-18T06:00:00.000Z
draft: true
---

There are so many reasons a manual deployment can cause problems. Some manual deployment problems I've seen in the past have included:

* The people who started a software project moved on to other projects, and not many engineers still working on the project knew about the extra manual deployment steps. 6 months later, the manual deployment steps were "rediscovered", after several features and bugfixes had been completed but not deployed. 
* One particular manual deployment took 12 steps to complete. Combined with post-deployment verification, that basically took half of someone's day. Given the amount of effort that went into a single deployment, features and minor fixes had a tendency to "stack up" prior to deployment. That led to more frequent bugs because every deployment included a lot of changed code. This led to even more frequent half-day deployments because fixes had to be deployed. Then feature code tended to back up while fixes were deployed, and the feedback loop continued. 
* The internet went down for the whole region for several hours mid-deployment. The site was stuck in a partially broken, half-deployed state, while folks scrambled to find a resource outside the region who knew the deployment process. It is much harder to talk someone through a manual process than it is to talk someone through pushing a button. 

For my own personal development, I also knew that paid development work would always take priority over personal development. It would be far too easy for my own site deployment to just sit and never actually get deployed. All of this motivated me to set up automated deployments for this blog. 

## Tooling

Since this site and the source code for it are in a public Github repo, I settled on a couple of free resources that are available through Github for public repos. For hosting the site, I am using Github Pages (though I originally went with a different hosting provider first, more on that in the Struggles section). For deploying, I am using Github Actions. Professionally, I have used a number of paid services for deployment (CircleCI, Harness, TeamCity), but for ease of use and cost Github Actions was top of my list. I also bought the domain on Namecheap. Very broadly, the steps are:

1. Creating a repo (see more on the building of the site [here](https://janetterounds.com/posts/creating-this-site/))
2. Setting up Github Pages for the repo (The [Github Docs](https://docs.github.com/en/pages/getting-started-with-github-pages/creating-a-github-pages-site) on this are pretty good)
3. Create secrets for the repo
4. Create the Github Action. 
5. Test the deployment

I'll cover steps 3-5 in detail here. 

### Setting up Secrets

First of all 

### The Action

```yaml
name: Deploy Hugo site to Pages
on:
  # Runs on pushes targeting the default branch
  push:
    branches: ["main"]
  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

jobs:
  # Build job
  build:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.112.7
    steps:
      # Install Hugo and other dependencies
      - name: Install Hugo CLI
        run: |
          wget -O ${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb \
          && sudo dpkg -i ${{ runner.temp }}/hugo.deb
      - name: Install Dart Sass Embedded
        run: sudo snap install dart-sass-embedded
      # Get the latest Code
      - name: Checkout
        uses: actions/checkout@v3
        with:
          submodules: recursive
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v3
      # Get the Node dependencies
      - name: Install Node.js dependencies
        run: "[[ -f package-lock.json || -f npm-shrinkwrap.json ]] && npm ci || true"
      # Generate static pages
      - name: Build with Hugo
        env:
          TINA_CLIENT_ID: ${{ secrets.TINA_PUBLIC_CLIENT_ID }}
          TINA_TOKEN: ${{ secrets.TINA_TOKEN }}
          TINA_BRANCH: main
          HUGO_ENVIRONMENT: production
          HUGO_ENV: production
        run: |
          npm run build
      # Upload static pages
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v1
        with:
          path: ./static
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      # Deploy static pages
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v2
```

## Struggles

* Deploying to namecheap hosting fails

## Wrapping Up
