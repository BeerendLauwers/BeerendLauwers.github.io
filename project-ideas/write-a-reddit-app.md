---
title: Organize your saved posts on Reddit
tags: Simple, API, Reddit, UI, Web App
---

## What

Reddit only saves 500 posts or threads, silently dropping the tail of the list.
It would be nice if we could export these saves and do stuff with them, such as assigning them to a category or searching through them.

Recently, [a library for interfacing with Reddit's API](https://hackage.haskell.org/package/reddit) was made available, so that part's already done (although I don't know if it supports fetching one's saved posts).

## What you will learn / Rough plan

* You'll want to save the posts to a persistent store. 
Lots of options here: an SQL database, a NoSQL store, a Git repo, ...  
Might be best to do this first before you actually try to display it.
Others might be able to use the "save your posts to X" as a library!

* How to save the post? A simple MarkDown blob with some metadata at the top?
Or perhaps a more structured approach for fancy filtering and categorization?

* If you want to have a web interface for all this, you'll need a web framework.

* The UI generation is up to you as well.
Perhaps a small project such as this is ideal to test out multiple ways of programming a UI in Haskell!

* Why not share your categories / tags with others or browse their saved posts?
This might work very nicely with using a Git repository as your data store: just point at the repository and load it in.