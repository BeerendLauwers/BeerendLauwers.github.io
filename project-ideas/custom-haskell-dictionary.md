---
title: Chrome extension that queries a custom dictionary for Haskell terms
tags: Simple, Web, Chrome Extension
---

## Why

Learning Haskell can be very hard for both experienced programmers and novices because of a lot of new lingo. 
What's a functor? A monad? What do they mean by "mapping over" a data structure?

Wouldn't it be nice if, on Haskell-related websites, such terms were automatically tagged with a tooltip that appears when you hover over it (or click it)?
The tooltip itself could just refer to some newbie-friendly tutorial or wiki page.

## What you will learn

This one is pretty devoid of Haskell programming.
You'll learn to create a Chrome extension and will have to write some JavaScript (although you could use [TypeScript](http://www.typescriptlang.org/) or [PureScript](http://www.purescript.org/) instead.)

## Rough implementation guide

### Chrome extension

There's already an existing Chrome extension that does something similar: [Dictionary Lookup for Chrome](https://github.com/max99x/dict-lookup-chrome-ext).
Apart from the Chrome extension, that Github repo has an entire Wiki-scraping parser that is irrelevant.

It might also be worthwhile (and interesting!) to simply write your own Chrome extension.

#### Bonus features

* Bonus points if it also includes a whitelist of sites on which the extension should run.
* Extra bonus points if the user is able to permanently dismiss a tooltip with an "I got the concept, thanks" button.

### Data source

For development purposes, a simple hardcoded JSON array would suffice.
For the live dictionary, there are a lot of options, but my simplest idea would be to have a Github repository that hosts a JSON file.
This would allow people to submit push requests to update the JSON file with new or updated entries.

#### Bonus features

* Perhaps having several JSON files, each with a specific flavour (category theory, type theory, etc) would be nice to have as well.
  The extension user could then opt in to several dictionaries via a user interface.
