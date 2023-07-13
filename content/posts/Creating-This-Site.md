---
title: Creating This Site
date: 2023-06-08T06:00:00.000Z
draft: false
tags:
  - 'development, blog'
---

This site has been in active development for .... has it really only been a week?? It has seemed much longer. Not gonna lie, I had some struggles. I'll talk about those in the Struggles section. But first, the things that actually worked for me!

## How it was made

You can find the source code for this site [on github](https://github.com/pherateriw/janette-dev-blog "ongithub"). I used [Tina CMS](https://tina.io/ "tina") with [Hugo](https://gohugo.io/documentation/) and the [m10c theme](https://github.com/vaga/hugo-theme-m10c). I'll talk more about hosting and deployment in a follow-up post.

First, [set up a repo](https://docs.github.com/en/github-ae@latest/get-started/quickstart/create-a-repo "setuprepo") on Github (or Git platform of choice). Also make sure you have Node.js and npm [installed](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm "nodesetup").
Finally, set up an IDE of your choice (I use [vscode](https://code.visualstudio.com/download))

### Setting up TinaCMS and Hugo

Now that you have a repo, node, and an IDE, it's time to set up TinaCMS. I used the starter.

1. Run `npx create-tina-app@latest`
2. Follow the prompts (note that a lot of TinaCMS docs use yarn as the package manager, so maybe consider using yarn instead). ![Screenshot of CLI which reads "Which package manager would you like to use? NPM. What is your project named? janette-demo. What starter code would you like to use? Hugo Starter"](</uploads/Screenshot 2023-06-07 at 10.39.38 PM.png> "TinaCLI")
3. **Important!** In the package json, add `--destination ./static` to the build command after `hugo`. (I will discuss why in the Struggles section). ![In the package json, add --destination ./static to the build command after hugo](</uploads/Screenshot 2023-06-07 at 10.51.47 PM.png>)
4. **Also Important!** Configure Hugo to use relative paths by adding `relativeURLs = true` to the `config.toml` file, especially if you use `baseURL = '/'` (Again, see the Struggles section for why)
5. The setup instructions include a line about how this assumes you have Hugo running on your machine. I ran `brew install hugo` (windows set up instructions [here](https://gohugo.io/installation/windows/)). However, hugo may or may not be running on the server you deploy your site to. So I'd also recommend adding [hugo-bin-extended](https://www.npmjs.com/package/hugo-bin-extended) to your package.json file with `npm i hugo-bin-extended`.
6. Run `npm run dev` and go to localhost:1313 in your browser. ![Screen shot of a website with the title "My New Hugo Site" and some Lorem Ipsum](</uploads/Screenshot 2023-06-08 at 6.18.33 PM.png> "Hugo-starter generated site")
7. Create an account on TinaCMS cloud and set up a project, following [this doc](https://tina.io/docs/tina-cloud/). Once you have an account and a token generated for your project, you can then export the variables TINA\_CLIENT\_ID, TINA\_TOKEN, and TINA\_BRANCH with the values you created.
8. Run `npm run build`

#### Configuring Themes

This is an optional step, but you can find different themes [here](https://themes.gohugo.io/). I used the [m10c theme](https://github.com/vaga/hugo-theme-m10c). Setup is mostly the same as the README. Git will also ask you to run `git submodule add https://github.com/vaga/hugo-theme-m10c themes/m10c` before pushing your local branch to github.

## Struggles

* Hugo's default destination for generated static pages is the ./public folder. This meant that all the stuff for TinaCMS was not added to the output folder. Adding `--destination ./static` to the hugo build command fixes this.
* CSS/JS for the generated pages originally had incorrect paths in the html (e.g. `/<css/file/path>`). This meant that the site was deployed without any styling. I set `relativeURLs = true` , and now my site has styling.

## Wrapping Up

I hope this helped you set up your own TinaCMS and Hugo site. Next up, deploying sites on push to the main branch with Github Actions!
