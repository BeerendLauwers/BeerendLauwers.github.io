---
title: Getting Hakyll up and running on Windows
---

Phew. That took a lot longer than expected.

## Installing GHC on Windows

Most people use the [Haskell Platform](https://www.haskell.org/platform/) to get GHC working properly on Windows. 
I knew that the HP is currently in a state of flux and that building libraries like `network` tends to fail.

Enter [MinGHC](https://github.com/fpco/minghc): a minimal installer for getting GHC, Cabal and (optionally) MSYS.
I already had MSYS installed because of [msysgit](https://msysgit.github.io/), so I was a bit fearful it would tear a hole into my Gitting capabilities.
Fortunately, MinGHC installed without a hitch, and I could compile packages with C-library dependencies. Great!

## Setting up Stackage and putting it all in a sandbox

### Stackage

I like [Stackage](http://www.stackage.org/). It's a stable package repository. 
Combining it with Cabal sandboxes, you're usually well protected against cabal hell.

To make Stackage the default global repository, do this:

1. Download the `.config` file from [Stackage](http://www.stackage.org/). 
I originally used version [lts-1.15](http://www.stackage.org/lts-1.15/cabal.config).
This becomes a problem later on because it doesn't include `hakyll`, so you'll want a more recent version. 
I used [lts-2.14](http://www.stackage.org/lts-2.13/cabal.config).

2. Then, (on Windows 7) go to the directory `Users/<username>/AppData/Roaming/cabal` and open the `config` file.
Go to the bottom of the file and paste the entire contents of the Stackage LTS config file there.

3. Because I like extra safety, I also uncommented the `remote-repo` line:

```
-- To only use tested packages, uncomment the following line:
remote-repo: stackage-lts-2.13:http://www.stackage.org/lts-2.13
```

### Cabal sandboxes

Ok. Now to install everything in a cabal sandbox. Go to the folder where you want to put the sandbox in and do:

```
> cabal sandbox init
> cabal install hakyll
```

Because I was originally using Stackage LTS 1.15, it didn't have Hakyll. 
I then had the very bad idea of allowing the Hackage repository, but it would solely be for this cabal sandbox.

### Exposing the Hackage repository to a single sandbox

Go into your initiated cabal sandbox directory and put a `cabal.config` file in it with the following contents:

```
remote-repo: hackage.haskell.org:http://hackage.haskell.org/packages/archive
remote-repo-cache: <absolute-path-to-your-cabal-sandbox-directory>\.cabal-sandbox\packages
```

We can then access it with the `--config-file` flag:

```
> cabal --config-file=cabal.config update
```

I tried installing Hakyll this way, which failed spectacularly. 
I then updated my global cabal `.config` file to LTS 2.13 and downloaded and installed the `hakyll` package in a new sandbox.

## Generating a Hakyll site and customizing it

### hakyll-init

I just followed the instructions on the [Hakyll website](http://jaspervdj.be/hakyll/tutorials/01-installation.html) here:

```
> hakyll-init beerendlauwers
'hakyll-init' is not recognized as an internal or external command,
operable program or batch file.
```

Oh. Right. The binary was installed in the sandbox:

```
> ".cabal-sandbox/bin/hakyll-init" beerendlauwers
Creating beerendlauwers\templates\post.html
Creating beerendlauwers\templates\post-list.html
Creating beerendlauwers\templates\default.html
Creating beerendlauwers\templates\archive.html
Creating beerendlauwers\site.hs
Creating beerendlauwers\posts\2012-12-07-tu-quoque.markdown
Creating beerendlauwers\posts\2012-11-28-carpe-diem.markdown
Creating beerendlauwers\posts\2012-10-07-rosa-rosa-rosam.markdown
Creating beerendlauwers\posts\2012-08-12-spqr.markdown
Creating beerendlauwers\index.html
Creating beerendlauwers\images\haskell-logo.png
Creating beerendlauwers\css\default.css
Creating beerendlauwers\contact.markdown
Creating beerendlauwers\about.rst
Creating beerendlauwers\test.cabal
```

### Customizing the site

I took over the great majority of the layout of [Chris Done's excellent website](http://chrisdone.com/).
He uses an old version of Hakyll, so the Haskell code was slightly different. 
Still, it was easy to mentally map how Chris' `Main.hs` correlated with the `site.hs` file.

I wanted nice font-based icons for the social media links on my homepage, and [Font Awesome](http://fortawesome.github.io/Font-Awesome/) delivers in spades.
I copied over the `css` and `fonts` folders into my website root directory, but the `fonts` folder wasn't being copied over during compilation.
A quick look in `site.hs` showed I needed to add a few lines:

```haskell
    match "images/*" $ do
        route   idRoute
        compile copyFileCompiler
        
    match "fonts/*" $ do            -- Added
        route   idRoute             -- these
        compile copyFileCompiler    -- lines

    match "css/*" $ do
        route   idRoute
        compile compressCssCompiler
```

After that, the site was ready for writing posts!

##Compiling this very post

Yes, we are most certainly not out of the woods yet. Running `site build` caused this output, dying on the `'` in the sentence `Combining it with Cabal sandboxes, you're usually well protected against cabal hell.`:

```
Initialising...
  Creating store...
  Creating provider...
  Running rules...
Checking for out-of-date items
Compiling
  updated posts/2015-06-06-getting-hakyll-running-on-windows.markdown
site: _site\posts/2015-06-06-getting-hakyll-running-on-windows.html: 
commitBuffer: invalid argument (invalid character)
```

[Apparently](https://github.com/jaspervdj/hakyll/issues/253), the solution is to change the active code page by doing `chcp 65001` in the command line before doing a `site build`:

```
> chcp 65001
Active code page: 65001

> site build
Initialising...
  Creating store...
  Creating provider...
  Running rules...
Checking for out-of-date items
Compiling
  updated posts/2015-06-06-getting-hakyll-running-on-windows.markdown
  updated archive.html
  updated index.html
Success
```

##Throwing it all on Github to host on Github pages

I had already configured a Github pages repository, so all I needed to do was push it to the repository.

There's some really fancy deployment options available, such as automatic deployment with [Travis CI](http://begriffs.com/posts/2014-08-12-create-static-site-with-hakyll-github.html).

I'll keep it simple for the moment:

