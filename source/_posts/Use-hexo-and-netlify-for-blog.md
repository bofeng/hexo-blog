---
title: Use hexo and netlify for blog
typora-root-url: ../../source
date: 2021-02-08 22:39:53
tags: hexo
---

I was using [Gridea](https://gridea.dev/) for a long time, but everytime when I try to setup in my new laptop, I have trouble syncing it, and don't know what happened under the hood and hard to fix it.

So switch to hexo, I like it so far. It is clean and have a clear struture, with the command line,  you know what it does, which files saved to which folder, etc. And it is very easy to setup and integrate with [Netlify](https://www.netlify.com/), all free.

You don't need another tutorial for how to set it up and host it with Netlify, [this article](https://www.netlify.com/blog/2015/10/26/a-step-by-step-guide-hexo-on-netlify/) already did a good job.



## Themes

There  are over [300 themes](https://hexo.io/themes/) ! Currently I am using the [cactus](https://github.com/probberechts/hexo-theme-cactus) one, clean. When you put the cactus folder under the `themes folder`, don't use `git clone`, just directly download the cactus zip, then unzip it and put it under the `themes` folder.

Config file sample, in top level's yaml:

```yaml
url: https://bofeng.netlify.app
root: /
permalink: /:year/:month/:day/:title/
##
theme: cactus-customized
theme_config:
  colorscheme: white
```

theme level yaml:

```yaml
nav:
  home: /
  Tags: /tags/
  articles: /archives/

# comment out all social medias
# social_links:
#   twitter: /

tags_overview: true

posts_overview:
  show_all_posts: true
  post_count: 20
  sort_updated: false
  
logo:
  enabled: true
  width: 50
  height: 50
  url: /images/logo.png
  gravatar: false
  grayout: false
  
rss: atom.xml
```



## Search

Another good thing, it is easy to integrate the search!

1, In your hexo blog top folder, run:

```bash
$ npm install hexo-generator-search --save
```

2, Create new the `search` folder under `source`:

```bash
$ hexo new page search
```

3, cd to the `source/search` folder, change `index.md`'s content to :

```
title: Search
type: search
---
```

4, Open your theme's `_config.yml` file, in the nav item, add search:

```yaml
nav:
  search: /search/
```

5, And in top level yaml (optional, b/c this is the default setting):

```yaml
search:
  path: search.xml
  field: post
  content: true
```



## Hexo Command

1, Create new post: `hexo new post "post name"`

2, Create new page: `hexo new page "open source"`, this will create a `open-source` folder under the `source` folder, and you can further edit the `index.md` inside.

3, To run hexo local server and preview your post: `hexo server`

4, To generate all static files for your site, use `hexo generate` or `hexo g`



## Reference

1, [Hexo Official Website](https://hexo.io/)

2, [A Step-by-Step Guide: Hexo on Netlify](https://www.netlify.com/blog/2015/10/26/a-step-by-step-guide-hexo-on-netlify/)