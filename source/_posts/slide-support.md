title: hexo-theme-melody v1.5 supports iframe & slides
categories:
  - Web 
  - 开发
date: 2018-03-06 19:57:52
tags: hexo
layout: slides
slide:
  theme: night
---

## hexo-theme-melody <small>v1.5</small>
<!-- .slide: data-background="#49B1F5" -->

Supports iframe & slides. You can use a layout called `slides` to enabled the slides layout.

Also you can add a `iframe` front-matter with the `slides` layout in your `md` file to enable the iframe page.

===

## Steps
<!-- .slide: data-transition="concave" data-background="#C7916B" -->

### 1. Add a slides page

```bash
hexo new page slides
cd ./source/slides
```

===

### 2. Add the layout type
<!-- .slide: data-transition="fade" data-background="#00C4B6" -->

```bash
vim index.md
```

Add a type called `slides`：


```yaml
title: slides
date: 2018-03-06 20:24:48
type: slides
```

===

### 3. Modified the melody.yml
<!-- .slide: data-transition="convex" data-background="#1B9EF3" -->

Add slides default config:


```yaml
slide:
  separator: whatever you like
  separator_vertical: whatever you like
  charset: utf-8
  theme: black
  mouseWheel: false
  transition: slide
  transitionSpeed: default
  parallaxBackgroundImage: ''
  parallaxBackgroundSize: ''
  parallaxBackgroundHorizontal: null
  parallaxBackgroundVertical: null
```

> See reveal.js [config](https://github.com/hakimel/reveal.js#configuration)

===

### 4. Write a md file with slides layout
<!-- .slide: data-transition="zoom" data-background="#F47466" -->

In `_posts` folder, add a `md` file.

For example:

```
title: hexo-theme-melody v1.5 supports iframe & slides
date: 2018-03-06 19:57:52
layout: slides
---

// balalala...
```

Then you will get a post of slides type.

===

## Slides layout with iframe

If you want to add a website whatever you like within an iframe, try this:

In `_posts` folder, add a `md` file.

```
title: hexo-theme-melody v1.5 supports iframe & slides
date: 2018-03-06 19:57:52
layout: slides
iframe: https://the-url-whatever-you-like
---
```

Then you will get a post of iframe.

===

## Configurate single slides in md
<!-- .slide: data-transition="convex" data-background="#69C282" -->

The slides config in `meldoy.yml` can change whole slides page.

But if you set the config in the md file, it will effect the single page.

==

For example:

```
title: hexo-theme-melody v1.5 supports iframe & slides
date: 2018-03-06 19:57:52
layout: slides
slide:
  theme: white
  transition: zoom
---

// balalala...
```

===

# Enjoy!
<!-- .slide: data-background="#49B1F5" -->


