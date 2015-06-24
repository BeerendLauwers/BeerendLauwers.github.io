---
title: Dockerizing Hakyll
---

It would be nice if I could have Docker container that, given a Git repository with Hakyll site code, pulled it and built it.

Then, other Docker containers could start from that point and push the built site to something (local file system, another Git repo, etc).

## The tools

I'm using [boot2docker](http://boot2docker.io/) version 1.6.2:

```
>boot2docker version
Boot2Docker-cli version: v1.6.2
Git commit: cb2c3bc
```

## Firing up a container

I started from the image [https://registry.hub.docker.com/u/ivotron/hakyll/](https://registry.hub.docker.com/u/ivotron/hakyll/) by doing:

```
docker pull ivotron/hakyll
```

Then,

```
docker run -d ivotron/hakyll git@github.com:beerendlauwers/beerendlauwers.github.io.git
Error response from daemon: Cannot start container x: 
[8] System error: exec: "git@github.com:beerendlauwers/beerendlauwers.github.io.git": 
stat git@github.com:beerendlauwers/beerendlauwers.github.io.git: no such file or directory
```

Ooookaaay.

Let's go inside the running process.

```
docker run -i -t -d ivotron/hakyll
9dec513a87dc1c50482d157e0e468cbf95cbe452d8e82b1e7a872049402286b9
```

 * `-i` keeps the STDIN open.
 * `-t` gives us a pseudo-TTY.
 * `-d` runs the container in the background and prints container ID.

Now let's login:

``` 
$ docker ps
CONTAINER ID    IMAGE                  COMMAND       CREATED         STATUS        
9dec513a87dc    ivotron/hakyll:latest  "/bin/bash"   26 minutes ago  Up 1 seconds
```

Now let's attach to it:

```
docker attach 9dec
root@9dec513a87dc:~/src#
```

```
root@9dec513a87dc:~/src# git clone git@github.com:beerendlauwers/beerendlauwers.github.io.git clone
(want to accept fingerprint) (YES)
Permission denied (publickey).
Fatal: Could not read from remote repository.
```

Fine.

```
root@9dec513a87dc:~/src# git clone https://github.com/beerendlauwers/beerendlauwers.github.io.git clone
Cloning into 'clone'...
```

Nice.

```
root@9dec513a87dc:~/src# cd clone
root@9dec513a87dc:~/src/clone# git checkout remotes/origin/code
root@9dec513a87dc:~/src/clone# ghc --make -fforce-recomp site.hs
[1 of 1] Compiling Main             ( site.hs, site.o )
Linking site ...
root@9dec513a87dc:~/src/clone# ./site build
```

## Copy over compiled site

Open another (boot2docker) shell.
Go to the physical machine folder you want to copy the site contents into.

```
$ docker cp 9dec513a87dc:/home/src/clone/_site .
FATA[0000] Error response from daemon: 
Could not find the file /home/src/clone/_site in container 9dec513a87dc.
```

Bah.

Google: [http://stackoverflow.com/questions/22907231/copying-files-from-host-to-docker-container](http://stackoverflow.com/questions/22907231/copying-files-from-host-to-docker-container)

Ok.

```
$ docker run -i -t -d -v /d/Experiments/test ivotron/hakyll
FATA[0000] Error response from daemon: 
cannot bind mount volume: d volume paths must be absolute.
```

Google: [http://superuser.com/questions/903193/mounting-a-directory-from-the-host-in-boot2docker-for-windows](http://superuser.com/questions/903193/mounting-a-directory-from-the-host-in-boot2docker-for-windows)

Ok.

```
$ docker run -i -t -d -v $APPDATA ivotron/hakyll
invalid value "C:\\Users\\blauwers\\AppData\\Roaming" for flag -v: 
\Users\blauwers\AppData\Roaming is not an absolute path
See 'd:\Program Files (x86)\Boot2Docker for Windows\docker.exe run --help'.
```

*Ugh*.

```
$ docker run -i -t -d -v //c/Users/blauwers/AppData/Roaming ivotron/hakyll
c64b7c2d64b0e4067c4cb47211082fd65cf505fd915fc73e2c01828fabdefba0
```

Ok, THIS works. Let's do it with the folder I really wanted:

```
$ docker run -i -t -d -v //d/Experiments/test:/artifacts ivotron/hakyll
94c06711f94a30bca343cc63e3bfdbbc1e54d83b86493fd78aed295fefce2c18
```

Attach to it and execute it all:

```
root@94c06711f94a:~/src# git clone https://github.com/beerendlauwers/beerendlauwers.github.io.git clone
root@94c06711f94a:~/src# cd clone
root@94c06711f94a:~/src/clone# git checkout remotes/origin/code
root@94c06711f94a:~/src/clone# ghc --make -fforce-recomp site.hs
root@94c06711f94a:~/src/clone# cd _site
root@94c06711f94a:~/src/clone/_site# cp . /artifacts -R
```

Zilch.

## Copy over compiled site with a new `Dockerfile`:

Ok, let's just make our own Dockerfile containing this:

```
from ivotron/hakyll

copy build /root/src/

workdir /root/src
entrypoint /root/src/build
```

```
$ docker build -t=hakylltest .
WARN[0000] SECURITY WARNING: You are building a Docker image from Windows agains
t a Linux Docker host. All files and directories added to build context will hav
e '-rwxr-xr-x' permissions. It is recommended to double check and reset permissi
ons for sensitive files and directories.
Sending build context to Docker daemon 3.072 kB
Sending build context to Docker daemon
Step 0 : FROM ivotron/hakyll
 ---> ea350b6c362a
Step 1 : COPY build /root/src/
 ---> Using cache
 ---> b990d59a8eab
Step 2 : WORKDIR /root/src
 ---> Using cache
 ---> 9515187aa728
Step 3 : ENTRYPOINT /root/src/build
 ---> Using cache
 ---> 1a2f38fe9045
Successfully built 1a2f38fe9045

$ docker run -it hakylltest https://github.com/beerendlauwers/beerendlauwers.github.io.git
: No such file or directory
```

Hmm.

Let's look at the `build` file.

```bash
#! /usr/bin/env bash

git clone $1 clone || exit 1
cd clone
ghc --make -fforce-recomp site.hs
./site build
```

That seems correct. [Google!](http://unix.stackexchange.com/questions/144718/sudo-unable-to-execute-script-sh-no-such-file-or-directory)

> *In my case, line ending was set wrong, to CR-LF for Windows, should be LF for Linux. Can take a while before you find that out.*

Ah, CRLF. [Let's fix it in Notepad++.](http://sqlblog.com/blogs/jamie_thomson/archive/2012/08/07/replacing-crlf-with-lf-using-notepad.aspx)

```
$ docker build -t=hakylltest .
blauwers@LBNL06449 /d/Experiments/test/hakyllwindows
$ docker run -it hakylltest https://github.com/beerendlauwers/beerendlauwers.github.io.git
fatal: repository 'clone' does not exist
```

Better. But it seems as if the repository isn't being passed to the shell script. [Google!](http://serverfault.com/questions/647779/docker-run-not-appending-arguments-to-image-entrypoint) 

> *It turns out the answer is to use the array form of ENTRYPOINT (and/or CMD) in order for appending from command line to work.*

Ok, so always wrap the `entrypoint`: 

```
entrypoint ["/root/src/build"]
```

```
$ docker run -it hakylltest "https://github.com/beerendlauwers/beerendlauwers.github.io.git"
Cloning into 'clone'...
remote: Counting objects: 142, done.
remote: Total 142 (delta 0), reused 0 (delta 0), pack-reused 142
Receiving objects: 100% (142/142), 498.32 KiB | 422.00 KiB/s, done.
Resolving deltas: 100% (50/50), done.
Checking connectivity... done.

<no location info>: can't find file: site.hs
/root/src/build: line 6: ./site: No such file or directory
```

A lot better! Don't have the correct branch, though. Let's add a command:

```bash
#! /usr/bin/env bash

git clone $1 clone || exit 1
cd clone
git checkout $2
ghc --make -fforce-recomp site.hs
./site build
```

```
$ docker build -t=hakylltest .
$ docker run -it hakylltest "https://github.com/beerendlauwers/beerendlauwers.g
ithub.io.git" "remotes/origin/code"
Cloning into 'clone'...
remote: Counting objects: 142, done.
remote: Total 142 (delta 0), reused 0 (delta 0), pack-reused 142
Receiving objects: 100% (142/142), 498.32 KiB | 396.00 KiB/s, done.
Resolving deltas: 100% (50/50), done.
Checking connectivity... done.
Note: checking out 'remotes/origin/code'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by performing another checkout.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -b with the checkout command again. Example:

  git checkout -b new_branch_name

HEAD is now at 8d08989... Google Analytics tracking
[1 of 1] Compiling Main             ( site.hs, site.o )
Linking site ...
Initialising...
  Creating store...
  Creating provider...
  Running rules...
Checking for out-of-date items
Compiling
  updated templates/default.html
  updated about.rst
  updated templates/post.html
  updated posts/2015-06-06-getting-hakyll-running-on-windows.markdown
  updated posts/2015-06-17-adventures-with-yesod-subsites.markdown
  updated templates/archive.html
  updated templates/post-list.html
  updated archive.html
  updated contact.markdown
  updated css/default.css
  updated css/font-awesome.css
  updated css/font-awesome.css.map
  updated css/font-awesome.min.css
  updated fonts/FontAwesome.otf
  updated fonts/fontawesome-webfont.eot
  updated fonts/fontawesome-webfont.svg
  updated fonts/fontawesome-webfont.ttf
  updated fonts/fontawesome-webfont.woff
  updated fonts/fontawesome-webfont.woff2
  updated images/haskell-logo.png
  updated index.html
Success
```

*Excellent.* Now, to do stuff with it.

## Copy over to host

Modify the build file as such

```bash
#! /usr/bin/env bash

git clone $1 clone || exit 1
cd clone
git checkout $2
ghc --make -fforce-recomp site.hs
./site build
echo Copying to host...
cd _site
cp . /artifacts -R
echo Copied to host.
```

Then run the container with a volume attached:

```bash
$ docker run -it -v //c/Users/blauwers/AppData/Roaming/test:/artifacts hakylltest 
 "https://github.com/beerendlauwers/beerendlauwers.github.io.git" "remotes/origin/code"
```
 
And, like magic, the files are there!

## Conclusion

Docker containers are a great way to modularly build up functionality.
It's awesome that we're able to take a basic Docker container that just builds a public repository and derive from it to build a container that does a custom deploy.
