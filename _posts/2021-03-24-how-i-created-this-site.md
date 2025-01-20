---
layout: archive
categories: 
    - Tools
author: Yue Yin
---

I created this page using GitHub page and [Minimals Mistakes](https://mmistakes.github.io/minimal-mistakes/docs/collections/), a [Jekyll](https://jekyllrb.com/) theme.

Here're the steps.

## Basic adoption

Install all dependencies. Follow the [quick-start guide](https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/). When this step is done, I have a good-looking homepage, but without navigation functionality.

## Navigation

### Jekyll documentation

Read [this](https://jekyllrb.com/docs/) to have some basic knowledge of how Jekyll works. Here are some important ones: 

- [Gem](https://jekyllrb.com/docs/ruby-101/) and basic [command line usages](https://jekyllrb.com/docs/step-by-step/01-setup/). Jekyll generates a static site, the output is in `_site`. 

- [How to customize the font?](https://github.com/mmistakes/minimal-mistakes/discussions/1219#discussioncomment-172827)

- [Liquid](https://jekyllrb.com/docs/step-by-step/02-liquid/). 

- [Front matter](https://jekyllrb.com/docs/step-by-step/03-front-matter/)

- [Layouts](https://jekyllrb.com/docs/step-by-step/04-layouts/). 
    - Stored in `_layouts` directory. To use layout, specify in the front matter of a html/markdown file.
        
- [Include](https://jekyllrb.com/docs/step-by-step/05-includes/) allows you to include content from another file stored in `_includes` folder. It's usually used to avoid repeating code. For example, you can create `_includes/navigation_bar.html`, and add to each page using {% raw %} `{% include navigation_bar.html  %}` {% endraw %}.

- [Data files](https://jekyllrb.com/docs/step-by-step/06-data-files/) serves to separate data from view. 

### Minimal Mistakes documentation

- [Add a post](https://mmistakes.github.io/minimal-mistakes/docs/posts/).

- [Add a page](https://mmistakes.github.io/minimal-mistakes/docs/pages/). 

- To create a `Post` session listing out all posts. 
    - Create an item in `navigation.yml`. This creats a new item in the navigation bar. However, the page doesn't exist yet.
    - Create a `posts.md` in `_pages`. An empty page is created.
    - Add the following to `posts.md`
    ```
    ---
    title: "Posts (by year)"
    permalink: /posts/
    layout: posts
    author_profile: true
    ---
    ```
    this applies the `posts` layout `_layouts/posts.html` to the page. It's looks good enough.

- To create a `Categories` session grouping posts by category.
    - Create an item in `navigation.yml`. This creats a new item in the navigation bar. However, the page doesn't exist yet.
    - Create a `categories.md` in `_pages`. An empty page is created.
    - Add the following to `categories.md`
    ```
    ---
    title: "Posts by category"
    permalink: /categories/
    layout: categories
    author_profile: true
    ---
    ```
    this applies the `posts` layout `_layouts/categories.html` to the page. It's looks good enough.

### Others

- How to show {% raw %} `{{` {% endraw %} in Jekyll? {% raw %} `{{` {% endraw %} and {% raw %} `}}` {% endraw %} is special syntax in Jekyll.
    - [https://www.tomordonez.com/curly-braces-markdown-jekyll/](https://www.tomordonez.com/curly-braces-markdown-jekyll/)
    - [https://docs.j7k6.org/escape-double-curly-brackets-jekyll/](https://docs.j7k6.org/escape-double-curly-brackets-jekyll/)

- You need different setups for building the site locally and GitHub page. See [this](https://yyin-dev.github.io/).

    - To run locally, run `bundle exec jekyll serve`. Note that this replaces the urls in `index.html` with your localhost `http://localhost:4000` which won't work when you push to GitHub. There are two workarounds ([reference](https://github.com/jekyll/jekyll-seo-tag/issues/406)):

        - Before pushing, run `bundle exec jekyll build` to build the production artifact.

        - When building locally, use `JEKYLL_ENV=production bundle exec jekyll serve`.