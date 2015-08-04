---
title: Building Hakyll with Stack
---

[Stack](https://github.com/commercialhaskell/stack) is *really* useful to get single packages up and running.
Here's a short tutorial on how to go from absolutely nothing to a `site.exe` file that can build your site contents.

## Get `stack`

First, I downloaded and installed `stack`, which was at [0.1.2.0](https://github.com/commercialhaskell/stack/releases/tag/v0.1.2.0) at the time of writing (for Windows).

## Make a new project

Then, I went to a folder in the command line and typed `stack new`:

```bash
NOTE: Currently stack new functionality is very rudimentary
There are plans to make this feature more useful in the future
For more information, see: https://github.com/commercialhaskell/stack/issues/137

For now, we'll just be generating a basic project structure in your current directory

Writing: LICENSE
Writing: Setup.hs
Writing: app\Main.hs
Writing: new-template.cabal
Writing: src\Lib.hs
Writing: test\Spec.hs
Writing default config file to: D:\Experiments\test\stack.yaml
Basing on cabal files:
- D:\Experiments\test\new-template.cabal

Checking against build plan lts-2.13
Selected resolver: lts-2.13
Wrote project config to: D:\Experiments\test\stack.yaml
```

## Downloading Hakyll

Then, we want to install [Hakyll](http://jaspervdj.be/hakyll/) with `stack install hakyll`:

```bash
While constructing the BuildPlan the following exceptions were encountered:

--  While attempting to add dependency,
    Could not find package Win32-notify in known packages

--  Failure when adding dependencies:
      Win32-notify: needed (>=0.3), latest is 0.3.0.1, but not present in build plan
    needed for package: fsnotify-0.1.0.3

--  Failure when adding dependencies:
      fsnotify: needed (>=0.1 && <0.2), latest is 0.2.1, but couldn't resolve its dependencies
    needed for package: hakyll-4.6.9.0

Recommended action: try adding the following to your extra-deps in D:\Experiments\test\stack.yaml
- Win32-notify-0.3.0.1

You may also want to try the 'stack solver' command
```

The most important part is down below:

> Recommended action: try adding the following to your extra-deps in D:\Experiments\test\stack.yaml
> 
> \- Win32-notify-0.3.0.1

Let's do that:

```yaml
flags: {}
packages:
- '.'
extra-deps:
- Win32-notify-0.3.0.1
resolver: lts-2.13
```

Let's do `stack install hakyll` again:

```bash
D:\Experiments\test>stack install hakyll
Win32-notify-0.3.0.1: configure
Win32-notify-0.3.0.1: build
Win32-notify-0.3.0.1: install
fsnotify-0.1.0.3: configure
fsnotify-0.1.0.3: build
fsnotify-0.1.0.3: install
hakyll-4.6.9.0: configure
hakyll-4.6.9.0: build
hakyll-4.6.9.0: install
Completed all 3 actions.
Installation path C:\Users\blauwers\AppData\Roaming\local\bin\ not found in PATH
 environment variable
Copying from D:\Experiments\test\.stack-work\install\x86_64-windows\lts-2.13\7.8
.4\bin\hakyll-init.exe to C:\Users\blauwers\AppData\Roaming\local\bin\hakyll-ini
t.exe

Installed executables to C:\Users\blauwers\AppData\Roaming\local\bin\:
- hakyll-init.exe
```

***Note:** You will probably have a far longer list than this, because stack caches its packages.*
*It took around 30 minutes on my computer to get to that last line the first time.*

## Bootstrapping Hakyll

Copy `hakyll-init.exe` from that folder into your base folder, like so:

<div class="image small">
![Location for `hakyll-init.exe`](/images/stack/hakyll.png)
</div>

Now, run it:

```bash
D:\Experiments\test>hakyll-init.exe .
Creating .\templates\post.html
Creating .\templates\post-list.html
Creating .\templates\default.html
Creating .\templates\archive.html
Creating .\site.hs
Creating .\posts\2012-12-07-tu-quoque.markdown
Creating .\posts\2012-11-28-carpe-diem.markdown
Creating .\posts\2012-10-07-rosa-rosa-rosam.markdown
Creating .\posts\2012-08-12-spqr.markdown
Creating .\index.html
Creating .\images\haskell-logo.png
Creating .\css\default.css
Creating .\contact.markdown
Creating .\about.rst
Creating .\test.cabal
```

## Merging the `.cabal` files

Time for some small bookkeeping.
Open up `test.cabal` and copy over this block into `new-template.cabal`:

```
executable site
  main-is:          site.hs
  build-depends:    base == 4.*
                  , hakyll == 4.6.*
  ghc-options:      -threaded
  default-language: Haskell2010
```

You might also want to remove the stack boilerplate build targets (`library`, `executable new-template-exe`, `test-suite new-template-test`).

Rename the package name as well:

```
name:                test
```

Rename `new-template.cabal` to `test.cabal` (remove the original one) and we're done.

## Building Hakyll

With `stack build`:

```bash
D:\Experiments\test>stack build
test-0.1.0.0: configure
Configuring test-0.1.0.0...
test-0.1.0.0: build
Building test-0.1.0.0...
Preprocessing executable 'site' for test-0.1.0.0...
[1 of 1] Compiling Main             ( site.hs, .stack-work\dist\x86_64-windows\Cabal-1.18.1.5\build\site\site-tmp\Main.o )
Linking .stack-work\dist\x86_64-windows\Cabal-1.18.1.5\build\site\site.exe ...
test-0.1.0.0: install
Installing executable(s) in
D:\Experiments\test\.stack-work\install\x86_64-windows\lts-2.13\7.8.4\bin
```

Copy over the `site.exe` file into your base directory, just like you did with `hakyll-init.exe`.

## Using Hakyll

Now, you can use `site.exe` to generate your site:

```bash
D:\Experiments\test>site.exe build
Initialising...
  Creating store...
  Creating provider...
  Running rules...
Checking for out-of-date items
Compiling
  updated templates/default.html
  updated about.rst
  updated templates/post.html
  updated posts/2012-08-12-spqr.markdown
  updated posts/2012-10-07-rosa-rosa-rosam.markdown
  updated posts/2012-11-28-carpe-diem.markdown
  updated posts/2012-12-07-tu-quoque.markdown
  updated templates/archive.html
  updated templates/post-list.html
  updated archive.html
  updated contact.markdown
  updated css/default.css
  updated images/haskell-logo.png
  updated index.html
Success
```

### Addendum: Full `.cabal` and `.yaml` files

`.cabal`:

```
name:                test
version:             0.1.0.0
synopsis:            Initial project template from stack
description:         Please see README.md
homepage:            http://github.com/name/project
license:             BSD3
license-file:        LICENSE
author:              Your name here
maintainer:          your.address@example.com
-- copyright:           
category:            Web
build-type:          Simple
-- extra-source-files:  
cabal-version:       >=1.10

executable site
  main-is:          site.hs
  build-depends:    base == 4.*
                  , hakyll == 4.6.*
  ghc-options:      -threaded
  default-language: Haskell2010

source-repository head
  type:     git
  location: https://github.com/name/project
```

`.yaml`:

```yaml
flags: {}
packages:
- '.'
extra-deps:
- Win32-notify-0.3.0.1
resolver: lts-2.13
```