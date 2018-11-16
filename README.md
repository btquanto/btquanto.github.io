# Jekyll Mon Cahier

Jekyll Mon Cahier is a simple, clean, and elegant Jekyll theme, based on the [Mon Cahier wordpress theme](https://wordpress.org/themes/mon-cahier/) by [Christine Rondeau](https://wordpress.org/themes/author/crondeau/).

![Jekyll Mon Cahier](/public/images/jekyll-mon-cahier.png)

# Content

* [Usage](#usage)
* [Options](#options)
    * [Posts](#posts)
    * [Pages](#pages)
    * [Comments](#comments)
    * [Recommended posts](#recommended-posts)
    * [Categories](#categories)
    * [Tags](#tags)
    * [Keywords](#keywords)
    * [Description](#description)
    * [Sidebar](#sidebar)
    * [Permalink](#permalink)
    * [Styles](#styles)
* [Author](#author)
* [License](#license)

# Usage

Jekyll Mon Cahier provides a full Jekyll setup. There are some usage options:

## Using github pages

* Fork this repo to your github account
* Rename the repo to `<username>.github.io`. Your site will be available at `<username>.github.io`
* If you want a custom domain, create a CNAME record and point it to `<username>.github.io`, then add your custom domain in the `CNAME` file.

## Using Jekyll server

* Install ruby

    ```
    curl -L https://get.rvm.io | bash -s stable --ruby=2.0.0
    ```

* Install Jekyll server

    ```
    gem install jekyll jekyll-pagination jekyll-mermaid
    ```

* Serve the site

    ```
    jekyll serve
    ```

* Open [localhost:4000](http://localhost:4000), and there it is

## Using docker

* Install docker by following instructions on [Docker's website](https://docker.io/). Make sure all necessary gems are installed as well.
* Use docker to serve your site.
    I am using [btquanto/docker-jekyll](https://hub.docker.com/r/btquanto/docker-jekyll/) docker image.

    ```
    docker run -d --name jekyll \
        -v `pwd`:/src \
        -p 4000:4000 \
        btquanto/docker-jekyll jekyll serve -H 0.0.0.0 --drafts
    ```

    The [grahamc/jekyll](https://hub.docker.com/r/grahamc/jekyll/) works as well, but the image size is much bigger (816.4MB in comparison to 226.5MB).

    ```
    docker run -d --name jekyll \
        -v `pwd`:/src \
        -p 4000:4000 \
        grahamc/jekyll serve -H 0.0.0.0 --drafts
    ```

    I prepared a `docker-compose.yml` file, so you can also install and use `docker-compose`. It by default uses the [btquanto/docker-jekyll](https://hub.docker.com/r/btquanto/docker-jekyll/) image. Follow instructions on [docker-compose release page](https://github.com/docker/compose/releases) to install the latest version. Do not use `apt-get` to install docker-compose.

    ```
    docker-compose up -d
    ```

* Some  commands to use with the created container:
    * Start the container:

        ```
        docker start jekyll
        ```

    * Stop the container:

        ```
        docker stop jekyll
        ```

    * Restart the container

        ```
        docker restart jekyll
        ```

# Options

## Posts

Place your posts inside `_posts` folder. A basic post only needs `title` specified in the front matter.

``` yaml
---
title: Blogging is fun
---
```

The default layout for posts in **Jekyll Mon Cahier** is `post`. You can customize it or add your own in `_layouts` folder.

There are other parameters for furthur customization. For example:

``` yaml
---
layout: post
title: Welcome to Jekyll
is_recommended: true
category: Hello World
tags: ["welcome", "hello"]
keywords: jekyll, welcome to jekyll, blog, hello world, the it fox, getting started
description: "Welcome to Jekyll! This is your first post. Edit or delete it, then start blogging!"
comments: false
---
```

Read more on [Jekyll's official documentation](https://jekyllrb.com/docs/posts/)

## Pages

Place your pages inside your site's root folder. Similar to a post, a basic page only needs `title` in its yaml front matter.

``` yaml
---
title: About
---
```

The default layout for pages in **Jekyll Mon Cahier** is `page`. You can customize it or add your own in `_layouts` folder.

There are other parameters for furthur customization. For example:

``` yaml
---
layout: page
title: About
keywords: about me, Jekyll Mon Cahier, Mon Cahier, Jekyll theme
description: "Jekyll Mon Cahier is a nice, simple, yet feature-rich Jekyll theme..."
comments: false
---
```

Read more on [Jekyll's official documentation](https://jekyllrb.com/docs/pages/)

## Comments

Comments are enabled by default for posts, and disabled by default for pages. Rendering comments can be enabled, or disabled, by adding `comments: true` or `comments: false` into the yaml front matter of the post or page.

``` yaml
comments: true
```

I am using [Disqus](https://disqus.com) to facilitate comments in this theme, thus you need to register a Disqus site in order to be able to moderate the comments on your site, then follow these instructions:

* Edit `_config.yaml`.
    * Change the field `disqus_identifier` to the value of your site's identifier.
    * (Optional) Change the field `url` to the value of your site's static url.
* Restart Jekyll (or the docker's container)

That's it. Your posts are ready to be commented.

## Recommended posts

Recommended posts will appear on the side bar. Add `is_recommended: true` into the post's front matter to mark it as recommended.

``` yaml
is_recommended: true
```

## Categories

**Posts** can be categorized into categories. Each post should only be in 1 category. The list of all categories is located on the side bar. Clicking on a category would lead to a the list of all posts belonging to that category. On hosts that do not support custom plugins, like **Github Pages**, the list of all posts belonging to a category will look differently.

``` yaml
category: Hello World
```

A post may belong to multiple categories, though I would prefer having only one category.

``` yaml
categories: ["Welcome", "Hello World"]
```

Although Jekyll would actually treat this as a single `"Welcome/Hello World"` category, I find it mismatched with the param `categories`, and `category: ["Welcome", "Hello World"]` looks weird. Thus, I decided to implement this as multiple categories.

## Tags

Posts can be tagged. A tag cloud that lists all tags is located on the sidebar. A post can have multiple tags. Tags behave in a manner that is very similar to category.

``` yaml
tags: ["welcome", "hello"]
```

Tags, along with keywords and description, are important for search engine optimization

## Keywords

You can add keywords meta data to your **posts**/**pages**, using the parameter `keywords` in your front matter. Keywords are strings, separated by commas.

``` yaml
keywords: jekyll, welcome to jekyll, blog, hello world, the it fox, getting started
```

Keywords, along with tags and description, are important for search engine optimization

## Description

**Posts** and **pages** can have a short description. Descriptions, along with keywords and tags, are important for search engine optimization. It can also be used when editing your Jekyll template.

``` yaml
description: "Welcome to Jekyll! This is your first post. Edit or delete it, then start blogging!"
```

## Sidebar

Edit `_include/sidebar.html` to add more stuffs into the sidebar. A sidebar section looks something like this:

``` html
<aside class="widget">
  <h1 class="widget-title"><a href="/recent/">Recent Posts</a></h1>
  <ul>
    {% for post in site.posts limit:site.recent_posts_limit %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
    {% endfor %}
  </ul>
</aside>
```

Jekyll's templates are written with Liquid. If you know enough, feel free to edit as you want.

## Permalink

The `permalink` setting in `_config.yml` is set to `/:year/:month/:day/:title:output_ext`, which is different from Jekyll's default setting. The default setting `/:category/:year/:month/:day/:title.html` will cause Disqus comments loses when changing the post's category.

You may change `permalink` with care. The current setting will generate urls ending with `.html`, which some do not like. However, it ensures the page working well under many HTTP server without any further configuration.

## Style

Pages and posts can have a style setting. Styles enabled different css rules for some elements in your blog post. Style setting is meant for advanced users, who may have different css rules for some blog posts, by writing their own css.

``` yaml
style: technical
```

The default style is set to `casual`, which adhere to the default visual look of the original wordpress theme **Mon Cahier**. There is also a `technical` style, whose only differences are the `font-family` and `font-size` of the `<quotation/>` tag.

# Author

* [https://github.com/btquanto](https://github.com/btquanto)
* [https://theitfox.com](https://theitfox.com)
* [https://fb.me/theitfox](https://fb.me/theitfox)

# License

[![License: GPL v2](https://img.shields.io/badge/License-GPL%20v2-blue.svg)](https://img.shields.io/badge/License-GPL%20v2-blue.svg)
