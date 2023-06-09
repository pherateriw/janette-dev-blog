---
title: Creating This Site
date: 2023-06-07T06:00:00.000Z
draft: true
---

This site has been in active development for .... has it really only been a week?? It has seemed much longer. Not gonna lie, I had some struggles. I'll talk about those in the Struggles section. But first, the things that actually worked for me!

## How it was made

You can find the source code for this site [on github](https://github.com/pherateriw/janette-dev-blog "ongithub"). I used [Tina CMS](https://tina.io/ "tina") with [Hugo](https://gohugo.io/documentation/) and the [m10c theme](https://github.com/vaga/hugo-theme-m10c). I'll talk more about hosting and deployment in a follow-up post.

First, [set up a repo](https://docs.github.com/en/github-ae@latest/get-started/quickstart/create-a-repo "setuprepo") on Github (or Git platform of choice). Also make sure you have Node.js and npm [installed](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm "nodesetup").
Finally, set up an IDE of your choice (I use [vscode](https://code.visualstudio.com/download))

### Setting up TinaCMS and Hugo

Now that you have a repo, node, and an IDE, it's time to set up TinaCMS.

1. Run `npx create-tina-app@latest`
2. Follow the prompts (note that a lot of TinaCMS docs use yarn as the package manager, so maybe consider using yarn instead). ![Screenshot of CLI which reads "Which package manager would you like to use? NPM. What is your project named? janette-demo. What starter code would you like to use? Hugo Starter"](</uploads/Screenshot 2023-06-07 at 10.39.38 PM.png> "TinaCLI")
3. **Important!** In the package json, add `--destination ./static` to the build command after `hugo`. (I will discuss why in the Struggles section). ![In the package json, add --destination ./static to the build command after hugo](</uploads/Screenshot 2023-06-07 at 10.51.47 PM.png>)
4. The setup instructions include a line about how this assumes you have Hugo running on your machine. I ran `brew install hugo` (windows set up instructions [here](https://gohugo.io/installation/windows/)). However, hugo may or may not be running on the server you deploy your site to. So I'd also recommend adding [hugo-bin-extended](https://www.npmjs.com/package/hugo-bin-extended) to your package.json file with `npm i hugo-bin-extended`.
5. Run `npm run dev` and go to localhost:1313. ![Screen shot of a website with the title "My New Hugo Site" and some Lorem Ipsum](</uploads/Screenshot 2023-06-08 at 6.18.33 PM.png> "Hugo-starter generated site") 
6. \*\*Also Important! \*\*Configuring Hugo to use relative paths:
7. Run `npm run build`

### Configuring Themes

## Struggles

* Hugo's default destination for generated static pages is the ./public folder
* The
