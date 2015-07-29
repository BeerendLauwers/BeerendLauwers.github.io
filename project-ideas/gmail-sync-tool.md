---
title: Desktop Outlook Calendar to Google Calendar Sync Tool
tags: Intermediate / Expert, Web, Command-Line, API, Parsing, FFI
---

## What

There are not a lot of syncing tools that can sync calendar items in my Outlook Calendar to my Gmail Calendar.
There are virtually no *free* syncing tools that can do that *reliably*, either.
I'm currently using [GO Contact Sync Mod](http://sourceforge.net/projects/googlesyncmod/).
For some reason, it offsets the times of all of my Outlook Calendar items by one hour, so it's only marginally useful to me.

There's a few extra features I would like it to have:

* Ability to sync my Outlook Calendar items at the right time.
* Ability to sync my Outlook Calendar items under a specific Google Calendar, not just the default one, so I can share it with others.
* Filter Outlook Calendar items and sync them in a specific Google Calendar (not that important).

I wondered how hard it would be to write a command-line tool like this in Haskell.

## General approach

### 100% Haskell

Writing it entirely in Haskell will probably be *very hard*. 
The first hurdle to pass is the parsing of `.pst` files, a gargantuan file format that saves all Outlook data in a proprietary binary format.
[A highly detailed technical documentation of the format is available here](https://msdn.microsoft.com/en-us/library/ff385210(v=office.12).aspx).
While, hypothetically, it would be possible to write a parser for these files, it would be a very large undertaking.
For those still interested, there are several open-source projects available in other languages for inspiration: [Java](https://github.com/rjohnsondev/java-libpst), [C](http://hg.five-ten-sg.com/libpst/).

### F\#

Writing it in [F#](http://www.tryfsharp.org/) might be a fun project to learn the language and allows you to use existing (C#) libraries to get data from a `.pst` file.

### Haskell + C FFI

Alternatively, one could interface with the [libpst](http://www.five-ten-sg.com/libpst/) C library via the C Foreign Function Interface and get data that way.
Several simple utilities are also available.
Especially [lspst](http://hg.five-ten-sg.com/libpst/file/c006b76da81d/src/lspst.c) is a very small example.
