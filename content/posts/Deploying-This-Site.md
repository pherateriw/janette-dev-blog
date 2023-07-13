---
title: Deploying This Site
date: 2023-06-18T06:00:00.000Z
draft: false
tags:
  - 'development, blog'
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

I'll cover steps 3-5 in detail here (the first two are covered better elsewhere, specifically in the links I provided).

### Setting up Secrets

TinaCMS requires 3 environment variables, and 2 of them are secret: TINA\_CLIENT\_ID and TINA\_TOKEN. You set these values when you connect TinaCMS to the cloud.

To set the secrets for your action:

1. Go to the settings page for your repo.
2. In the sidebar menu, select Secrets and Variables and click on Actions. \
   ![A screenshot of Github Settings menu with 'Secrets and Variables' sub-menu opened and the 'Actions' option highlighted.](</uploads/Screenshot 2023-06-18 at 2.15.36 PM.png>)
3. Hit the "New Repository Secret" button.
4. Name your secret. When you reference the secret in your action, you'll use `secrets.<NAME>`. The UPPERCASE convention is just that, a convention, but it helps to visually set values apart. Also, keep in mind how you will use the secret. It helps me to keep names mostly consistent. ![A screenshot of the Actions secrets/New secret page in Github](</uploads/Screenshot 2023-06-18 at 2.16.29 PM.png>)
5. Add the actual secret value.

### Creating The Action

Go to the Action Tab for your repo. If you haven't set up a previous Github Action, Github will have so many action templates for you to work from, or you can set up one without a template. Here is my action:

```yaml
name: Deploy Hugo site to Pages
on: 
  push:
    branches: ["main"] # 1
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest # 2
    env:
      HUGO_VERSION: 0.112.7
    steps:
      - name: Install Hugo CLI # 3
        run: |
          wget -O ${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb \
          && sudo dpkg -i ${{ runner.temp }}/hugo.deb
      - name: Install Dart Sass Embedded
        run: sudo snap install dart-sass-embedded
      - name: Checkout # 4
        uses: actions/checkout@v3
        with:
          submodules: recursive
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v3
      - name: Install Node.js dependencies # 5
        run: "[[ -f package-lock.json || -f npm-shrinkwrap.json ]] && npm ci || true"
      - name: Build with Hugo # 6
        env:
          TINA_CLIENT_ID: ${{ secrets.TINA_CLIENT_ID }} # 7
          TINA_TOKEN: ${{ secrets.TINA_TOKEN }}
          TINA_BRANCH: main
          HUGO_ENVIRONMENT: production
          HUGO_ENV: production
        run: |
          npm run build
      - name: Upload artifact # 8
        uses: actions/upload-pages-artifact@v1
        with:
          path: ./static # 9
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages # 10
        id: deployment
        uses: actions/deploy-pages@v2
```

What this does:

1. Sets the action to occur whenever I push to the main branch.
2. Sets up a container to run the action in.
3. Installs Hugo CLI.
4. Checks out the code.
5. Installs Node dependencies.
6. Generate the static pages.
7. Uses the secrets set up earlier.
8. Uploads the generated static pages.
9. Uses the configured generated pages location from a [previous blog post](https://janetterounds.com/posts/creating-this-site/).
10. Actually deploys the site.

### Testing the Deployment

This is easy to do. Just create a PR to the main branch, and merge it. Observe the action on the Actions tab for your repo. If your action succeeds, there will be a lovely green checkmark next to it.

![A screenshot of a successful Github Action with a green checkmark and the text "TinaCMS content update by Janette Rounds". Under that, there is more text "Deploy Hugo site to Pages #85: Commit e314b22 pushed by tina-cloud-app \[bot\]"](</uploads/Screenshot 2023-06-18 at 3.00.37 PM.png>)

If your action does not succeed, you can then click on the failed deployment and check out the logs.

## Struggles

* I originally had planned to use Namecheap for shared hosting as well. I went through some struggles before I finally gave up on Namecheap shared hosting including:
  * Using cPanel.yaml to deploy directly to Namecheap. However, this required I add a yaml file with my cPanel username checked into source control in the root directory. My cPanel username is not a super huge secret, but I still didn't want it floating about the internet. (I have since changed my cPanel username.)
  * Using Github integration with Namecheap to checkout the updated code on Namecheap shared hosting and then build it, and move the static pages into the correct folder for the public site. There were a lot of problems with this plan: I needed Hugo to be installed on the Namecheap shared host, the version of Node was incompatible with many of the packages that TinaCMS requires, and worst of all, there were still manual steps in the process that I would have had to automate with shell scripts or something.
  * Using the Github Action to generate the static pages and then use FTP to upload the generated pages to Namecheap shared hosting. This wasn't working originally, but at that point I hadn't figured out how to configure Hugo to generate everything in the `static` directory rather than the `public` directory. However, I've since figured it out, so I suppose this might work if I need to move away from Github Pages.

## Wrapping Up

This site is pretty simple, all things considered, so for right now, generating and deploying the static pages is all I really need. However, there are a LOT of ways to use Github Actions to automate things. For example, you could use it to run unit tests, deploy to a development or testing environment prior to full deployment, notify folks of changes, generate a changelog, and many more things. I hope this was helpful to you, or at least that I convinced you that your deployments should be automated. Thanks for reading!
