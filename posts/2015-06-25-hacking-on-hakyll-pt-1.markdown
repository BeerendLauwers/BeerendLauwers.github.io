---
title: Hacking on Hakyll - Part 1
---

## Motivation

I wanted to see how hard it was to add some new simple functionality to Hakyll: 
read a single file containing a list of `(Int,Int)`'s, one on each line.
Then, take the first `Int` and of each and lump them together.
(The idea is to use them to populate a data type later on.)

## Start

First, I looked at [Hakyll's documentation on Hackage](https://hackage.haskell.org/package/hakyll-4.7.1.0).
`Hakyll.Web.CompressCss` was very useful to understand how I could lift a pure function to a function that works in `Compiler`:

(code here)

## Writing the `readTuples` function 

So, what I first needed to do was to create the `readTuples` function:

```Haskell
readTuples :: Compiler (Item String)
readTuples = fmap readTuples' <$> getResourceString

readTuples' :: String -> String
readTuples' xs = 
  let res = map (\x -> (read x)::(Int,Int)) (lines xs)
  in  concatMap (show . fst) res
```

(I just put this in `site.hs`, by the way.)
 
`readTuples'` is simple: take the file contents (`xs`) and apply `lines` to it to get a list of `String`s.
Then, it `read`s each `String`, casting it to an `(Int,Int)`.[^1]
Finally, the first element of each tuple is taken and turned into a `String`. All those are then concatenated together.

`readTuples` then applies the aforementioned function to `getResourceString`, which provides a `String` version of the file's contents.

## Using it in the site compiler

We'll want to use `match`, because we take an existing file and transform it.
So, let's add this line to `main`:

```Haskell
match "testing/*" $ do
```

Then, we need to say where the processed file is going to end up.
This is what routes are for. 
For the moment, I'm just going to modify the file's contents when building the site, but not its location or name.
That's what `idRoute` does: just use the same directory and file name as that of the input file.

```Haskell
match "testing/*" $ do
    route idRoute
```

Then, we need to say what we actually want to *do* with the file's contents by using `compile`:

```Haskell
compile :: Compiler (Item a) -> Rules()
```

`readTuples` slots in here nicely:

```Haskell
readTuples :: Compiler (Item String)
```

So, we end up with:

```Haskell
match "testing/*" $ do
    route idRoute
    compile $ readTuples
```

That's it! Recompile `site.hs` and run it:

```bash
ghc --make -fforce-recomp site.hs
site build
```

## Example input

My test file is `testing/abcd.txt`[^2]:

```
(1,2)
(4,5)
(6,8)
```

After compilation, `_site/testing/abcd.txt` contains:

```
146
```

## Conclusion

It's not really so hard! 
Just define a pure function that does a transformation, lift it to `Compiler` and pass that to `compile`.

[^1]:   `read x` will cause an error if parsing fails, so use something safer if you actually want to use this in a program.

[^2]:   `read` doesn't seem to care if the returns are CRLF or LF:

        readTuples' "(1,3)\n(4,6)" = "14"
        readTuples' "(1,3)\r\n(4,6) = "14"