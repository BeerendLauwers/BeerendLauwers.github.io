---
title: Authoring presentations with Docker, reveal.js and Pandoc
---

I've always liked [reveal.js](https://github.com/hakimel/reveal.js/)' style of HTML5 presentations, and I know you can write them in MarkDown as well. I don't want to install NodeJS and Python and all the other dependencies to get a local development environment going, so I'll try and use Docker.

## Getting a reveal.js testing server up and running with Docker (failed)

First off, [I downloaded this docker image](https://registry.hub.docker.com/u/danidemi/docker-reveal.js/):

```bash
docker pull docker-reveal.js:latest
Error: image library/docker-reveal.js:latest not found
```

Good start.

```bash
$ git clone https://github.com/danidemi/docker-reveal.js.git
$ cd docker-reveal.js
$ docker build -t "danidemi/docker-reveal.js:latest" .
```

That works. Took AGES, though.

Next up, the tutorial says to

> Run a Docker container with this command.

```bash
docker run -d -v //c/Users/blauwers/AppData/Roaming/hakyll/reveal:/slides/ -p 8000:8000 "danidemi/docker-reveal.js:latest"
```

Then, I tried to get the `pandoc` install in the container to compile my MarkDown file:

```bash
(docker exec -i $dockerid /bin/sh -c "pandoc -t revealjs -s -o /slides/index.html") < index.md
```

Doesn't work. The container keeps dying. Let's inspect with `-i`:
 
```bash
docker run -i -v //c/Users/blauwers/AppData/Roaming/hakyll/reveal:/slides/ -p 8000:8000 "danidemi/docker-reveal.js:latest"
```

> (a whole bunch of retries to find `index.html`)

Ah, NodeJS keeps trying to look for `index.html`, crashing due to running out of stack size.

Let's add index.html and try again.

```bash
touch index.html
```

Ok, now it doesn't crash anymore (this was apparently in the tutorial, woops).

Unfortunately, the `docker exec` command still doesn't work, it's trying to use my Git's `bin/sh` in Program Files.

## Docker + Windows install of `pandoc`

Ok, screw it, let's just install [`pandoc` for Windows](http://pandoc.org/installing.html).

Now, let's go to the folder and run it there:

```bash
cd C:\Users\blauwers\AppData\Roaming\hakyll\reveal
C:\Users\blauwers\AppData\Roaming\hakyll\reveal>pandoc -t revealjs -s -o index.html index.md
```

Ok, now to refresh the web page of the docker instance:


<div class="image small">
![First successful compilation](/images/presentation/first.jpg)
</div>

Seems to work!

## Windows install of `pandoc` + copy of the Git repo of revealJS

Wow, what a hassle. Can't I just use the static version of revealJS? 

[Release overview](https://github.com/hakimel/reveal.js/releases) -> zip -> put `index.md` in it -> compile!

Just a white screen.

Hmm, seems like pandoc-compiled presentations search for reveal.js in a subfolder called `reveal.js`.

Let's plonk the `index.md` file just next to the `reveal.js` folder.

Ok, now I'm getting my text.
But the presentation functionality isn't present, nor is the CSS styling.
Two more changes are required:

* Change `reveal.js/js/reveal.js` to `reveal.js/js/reveal.min.js`.
* Same for `reveal.js/css/reveal.css`, to `reveal.js/css/reveal.min.css`.

*(Of course, actually minifying it would be a good idea if we'd actually publish this presentation.)*

Great, works on `localhost` now without a docker image!

## Switching themes (not working)

Let's try another theme:

```bash
pandoc -t revealjs -s -V theme=moon -o index.html index.md
```

Hmm, can't switch themes. Weird.

It's in the HTML, in any case:

```
theme: 'moon', // available themes are in /css/theme
```

I'll leave it for the moment.

## Swapping out `pandoc`'s syntax highlighting for `highlight.js`

The Haskell code is really unreadable: 

<div class="image small">
![Unreadable Haskell code!](/images/presentation/haskell-bad.png)
</div>

Let's disable it:

```bash
pandoc -t revealjs -s --no-highlight -o index.html index.md
```

How to get `highlight.js` functionality in? [Let's google around.](http://www.artembutusov.com/pandoc-markdown-syntax-highlighting-with-highlight-js/)

> You can use snippet below, insert it in the top of markdown file:

```HTML
<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.2/css/bootstrap.min.css">
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/8.4/styles/monokai_sublime.min.css">
<script src="https://code.jquery.com/jquery-2.1.3.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/8.4/highlight.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/8.4/languages/javascript.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/8.4/languages/php.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/8.4/languages/sql.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/8.4/languages/xml.min.js"></script>

<style>body { padding: 20px } pre { padding: 0 }</style>

<script>
$(function() {
    $("pre > code").each(function(i, block) {
        var codeClass = $(this).parent().attr("class");
        if (codeClass == null || codeClass === "") {
            $(this).addClass("hljs");
        } else {
            var map = {
                js: "javascript"
            };
            if (map[codeClass]) {
                codeClass = map[codeClass];
            }
            $(this).addClass(codeClass);
            hljs.highlightBlock(this);
        }
    });
});
</script>
```

Nice.

<div class="image small">
![Readable Haskell code!](/images/presentation/haskell-good.png)
</div>

That's a lot better!