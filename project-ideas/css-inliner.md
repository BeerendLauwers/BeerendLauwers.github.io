---
title: CSS inliner
tags: Simple, AST, Command-Line, Web
---

## Why

HTML-based e-mails require that all CSS is inlined to prevent it from clashing with the mail client's user interface.
I couldn't find a CSS inlining tool written in Haskell yet, and it seems like a nice little project with practical uses.

What it would do is take an HTML file and search for CSS that has been applied to it, which is then inlined using the `style` attribute.

## What you will learn / Rough implementation guide

It might be in your best interest to build on existing parsers for HTML and CSS:

### HTML

* [`tagsoup`](https://hackage.haskell.org/package/tagsoup) is able to parse HTML / XML and scrape data out of it.
* [`pandoc`](https://hackage.haskell.org/package/pandoc-1.9.3/docs/Text-Pandoc-Readers-HTML.html#v:readHtml) has a HTML parser that uses `tagsoup` internally.
  Using Pandoc might make it easy to create a traversal over the data structure.
  See [this page for examples on operating on the Pandoc data structure](http://pandoc.org/scripting.html). 
* [`html-conduit`](https://hackage.haskell.org/package/html-conduit) parses HTML in a more structured manner than `tagsoup`.
* [`hxt`](https://hackage.haskell.org/package/hxt) is the most comprehensive of all libraries here. Old, but trusted.

### CSS

* [`handsomesoup`](https://hackage.haskell.org/package/HandsomeSoup-0.4.2) is built atop `hxt` and aims at supporting CSS2 parsing.
* [`css-text`](https://hackage.haskell.org/package/css-text) is a lightweight library that is able to handle CSS media queries. 
  Outputs a simple tree data structure filled with `Text`.


### Useful resources

* [Working With HTML In Haskell](http://adit.io/posts/2012-04-14-working_with_HTML_in_haskell.html)

The way you track down what CSS is applicable is down to you.
[`handsomesoup`](https://hackage.haskell.org/package/HandsomeSoup-0.4.2) allows extracting elements based on a CSS selector, perhaps see how it's done?