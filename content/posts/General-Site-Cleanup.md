---
title: General Site Cleanup
date: 2023-07-02T06:00:00.000Z
draft: true
---

After [Creating the site](https://janetterounds.com/posts/creating-this-site/) and [Deploying the site](https://janetterounds.com/posts/deploying-this-site/) there were still a number of things I wanted to tweak, all minor. However, I had to move on to \<other project> for the sake of my brain. This is a big wrap-up post discussing all of the minor tweaks I made to get this site working how I want. Today we are going to cover:

* Using branches and pull requests (and other good git hygiene) within TinaCMS
* Swapping out and customizing themes in Hugo
* Updating the TinaCMS graphql schema
* Adding analytics
* Possibly other things TBD

## Using Branches

This was quite simple. I added the following code to Line 8 of my `tina/config.ts` file:

```javascript
cmsCallback: cms => {
  cms.flags.set("branch-switcher", true);
  return cms;
},
```

I merged my PR and now this post is being edited within a branch on Github!

## Updating the TinaCMS graphql schema

I wanted to add tags to my posts, and have a list of tags on the main page. To do this, I added the following to the config in my `tina/config.ts`

```
{
  type: "string",
  name: "tags",
  label: "Tags",
  list: true
},
```

And then in my config.toml, under my other menu options I added this:

```
[[menu.main]]
  identifier = "tags"
  name = "Tags"
  url = "/tags/"
  weight = 3
```
