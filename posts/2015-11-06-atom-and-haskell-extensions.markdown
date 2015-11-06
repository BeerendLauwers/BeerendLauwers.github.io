---
title: Installing Atom + Haskell extensions: an experiment
---

I wanted to see how easy it would be to install the recommended Haskell plugins on Atom, using Stack.

<div class="note">
**This is not a tutorial or how-to**. Your best bet for getting a working `ghc-mod` with Stack on Windows is:

* `stack` >= 0.1.4.0
* `ghc` >= 7.10
* `ghc-mod` >= 5.4 ([see mailing list announcement](https://mail.haskell.org/pipermail/haskell-cafe/2015-September/121412.html))
* Atom >= 1.0

</div>

## ide-haskell

Googling for "Atom Haskell" gives me this plugin: [ide-haskell](https://atom.io/packages/ide-haskell)

From the page:

> To install:
> 
> `apm install language-haskell haskell-ghc-mod ide-haskell-cabal ide-haskell autocomplete-haskell`
> 

### Attempt via bash shell

In git bash:

```bash
$ ./apm install language-haskell haskell-ghc-mod ide-haskell-cabal ide-haskell autocomplete-haskell
Installing language-haskell to C:\Users\blauwers\.atom\packages failed

language-haskell@1.4.12 postinstall C:\Users\blauwers\AppData\Local\Temp\apm-install-dir-115106-7516-4i9g01\node_modules\language-haskell
coffee src/haskell.coffee

npm WARN addRemoteGit Error: Command failed:
npm WARN addRemoteGit     at ChildProcess.exithandler (child_process.js:658:15)
npm WARN addRemoteGit     at ChildProcess.emit (events.js:98:17)
npm WARN addRemoteGit     at maybeClose (child_process.js:766:16)
npm WARN addRemoteGit     at Socket.<anonymous> (child_process.js:979:11)
npm WARN addRemoteGit     at Socket.emit (events.js:95:17)
npm WARN addRemoteGit     at Pipe.close (net.js:466:12)
npm WARN addRemoteGit  git://github.com/michaelficarra/cscodegen.git#73fd7202ac086c26f18c9d56f025b18b3c6f5383 resetting remote C:\Users\blauwers\.atom\.apm\_git-remotes\git-github-com-michaelficarra-cscodegen-git-79ca3b9e because of error: { [Error: Command failed: ] killed: false, code: 1, signal: null }
npm ERR! git clone --template=C:\Users\blauwers\.atom\.apm\_git-remotes\_templates --mirror git://github.com/michaelficarra/cscodegen.git C:\Users\blauwers\.atom\.apm\_git-remotes\git-github-com-michaelficarra-cscodegen-git-79ca3b9e: Cloning into bare repository 'C:\Users\blauwers\.atom\.apm\_git-remotes\git-github-com-michaelficarra-cscodegen-git-79ca3b9e'...
npm ERR! git clone --template=C:\Users\blauwers\.atom\.apm\_git-remotes\_templates --mirror git://github.com/michaelficarra/cscodegen.git C:\Users\blauwers\.atom\.apm\_git-remotes\git-github-com-michaelficarra-cscodegen-git-79ca3b9e: fatal: unable to connect to github.com:
npm ERR! git clone --template=C:\Users\blauwers\.atom\.apm\_git-remotes\_templates --mirror git://github.com/michaelficarra/cscodegen.git C:\Users\blauwers\.atom\.apm\_git-remotes\git-github-com-michaelficarra-cscodegen-git-79ca3b9e: github.com[0: 192.30.252.130]: errno=No error
npm WARN optional dep failed, continuing cscodegen@git://github.com/michaelficarra/cscodegen.git#73fd7202ac086c26f18c9d56f025b18b3c6f5383
'coffee' is not recognized as an internal or external command,
operable program or batch file.
npm ERR! Windows_NT 6.1.7601
npm ERR! argv "C:\\Users\\blauwers\\AppData\\Local\\atom\\app-1.1.0\\resources\\app\\apm\\bin\\node.exe" "C:\\Users\\blauwers\\AppData\\Local\\atom\\app-1.1.0\\resources\\app\\apm\\node_modules\\npm\\bin\\npm-cli.js" "--globalconfig" "C:\\Users\\blauwers\\.atom\\.apm\\.apmrc" "--userconfig" "C:\\Users\\blauwers\\.atom\\.apmrc" "install" "C:\\Users\\blauwers\\AppData\\Local\\Temp\\d-115106-7516-1e496pn\\package.tgz" "--target=0.30.7" "--arch=ia32" "--msvs_version=2012"
npm ERR! node v0.10.40
npm ERR! npm  v2.13.3
npm ERR! code ELIFECYCLE

npm ERR! language-haskell@1.4.12 postinstall: `coffee src/haskell.coffee`
npm ERR! Exit status 1
npm ERR!
npm ERR! Failed at the language-haskell@1.4.12 postinstall script 'coffee src/haskell.coffee'.
npm ERR! This is most likely a problem with the language-haskell package,
npm ERR! not with npm itself.
npm ERR! Tell the author that this fails on your system:
npm ERR!     coffee src/haskell.coffee
npm ERR! You can get their info via:
npm ERR!     npm owner ls language-haskell
npm ERR! There is likely additional logging output above.
```

Woo.
Okay, let's try via Atom itself.

### Via Atom GUI

#### language-haskell

![language-haskell](/images/atom/language-haskell.png)

Ok, that installed properly.

#### haskell-ghc-mod

![haskell-ghc-mod](/images/atom/ghc-mod-error.png)

Oh dear.
Googling gives me this: [Github issue: install failed](https://github.com/atom-haskell/haskell-ghc-mod/issues/43)

> Did you install it?

Ah.

[haskell-ghc-mod](https://atom.io/packages/haskell-ghc-mod) says:

> You need to have `ghc-mod`, `ghc-modi` (part of `Ghc-Mod`) and `hlint` executables installed on your system. 
> `ghc-mod` needs to be able to find `hlint` (eiter add `hlint` directory to PATH, or install both in the same cabal sandbox).

Okay, let's build it via Stack:

Windows shell:

```bash
C:\Users\blauwers\AppData\Local\atom\bin>stack install ghc-mod
Run from outside a project, using implicit global config
Using resolver: lts-2.21 from global config file: C:\Users\blauwers\AppData\Roam
ing\stack\global\stack.yaml
Caching build plan
Continuing despite missing tool: msys2
Setting codepage to UTF-8 (65001) to ensure correct output from GHC
Progress: 0/54
...
Completed all 54 actions.
Installation path C:\Users\blauwers\AppData\Roaming\local\bin\ not found in PATH environment variable
Copying from C:\Users\blauwers\AppData\Roaming\stack\snapshots\i386-windows\lts-2.21\7.8.4\bin\ghc-mod.exe 
    to C:\Users\blauwers\AppData\Roaming\local\bin\ghc-mod.exe
Copying from C:\Users\blauwers\AppData\Roaming\stack\snapshots\i386-windows\lts-2.21\7.8.4\bin\ghc-modi.exe 
    to C:\Users\blauwers\AppData\Roaming\local\bin\ghc-modi.exe

Copied executables to C:\Users\blauwers\AppData\Roaming\local\bin\:
- ghc-modi.exe
- ghc-mod.exe

C:\Users\blauwers\AppData\Local\atom\bin>
```

Okay, I have to add that path to my PATH environment variable. Let's test:

```bash
C:\Users\blauwers>ghc-mod --version
ghc-mod version 5.2.1.2 compiled by GHC 7.8.4

C:\Users\blauwers>ghc-modi
fdf
OK
NG BUG: user interrupt

C:\Users\blauwers>
```

Looks good!
Atom still gives me those error messages, though, so let's restart.

Okay, I get the green after a restart:

![haskell-ghc-mod runs!](/images/atom/ghc-mod-fixed.png)

#### ide-haskell-cabal

Installed without a hitch.

#### ide-haskell

Same.

#### autocomplete-haskell

Same! I'm surprised.

## Trying it out

I loaded up the code branch of the repository that generates this website in Atom as a project.
After selecting `site.hs`, I get... Not much. 
A message that tells me the editor isn't responding.

Let's see if there are any known issues:

> There can be some problems with `ghc-modi` upstream, most notably, it does not work on paths with whitespace. 

Ah. Disabling it in the settings and restarting the plugin does seem to allow `site.hs` to load:

![haskell-ghc-mod user error (ghc not found)](/images/atom/ghc-mod-errors.png)

Hmm.

`stack path` tells me:

```bash
ghc-paths: C:\Users\blauwers\AppData\Local\Programs\stack\i386-windows
```

None of the `bin` folders of the GHC's there are in my path, so more path pollution is in order!

Ok, having added GHC to my path and restarting Atom, I get these messages:

![ghc-mod-cabal-errors](/images/atom/ghc-mod-cabal-errors.png)

Here's the error message:

> launching operating system process `cabal configure` failed: cabal: readProcessWithExitCode: does not exist (No such file or directory)

Googling around gives me these results:

* [ghc-mod issue: using ghc-mod with stack](https://github.com/kazu-yamamoto/ghc-mod/issues/498)
* [haskell-ghc-mod issue: haskell-ghc-mod doesn't support local GHC using Stack](https://github.com/atom-haskell/haskell-ghc-mod/issues/44)

## Conclusion

I think the best course of action is to nuke it all and start with a more recent version of `ghc-mod`, seeing as Stack support was only introduced in 5.4.
I'll revisit this another time.
