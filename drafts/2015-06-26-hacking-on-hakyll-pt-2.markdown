---
title: Hacking on Hakyll - Part 2
---

`test´

## Motivation

[Part 1](2015-06-25-hacking-on-hakyll-pt-1.html) shows how easy it is to quickly add some simple functionality to Hakyll.

I want to do it a little more structured this time and understand what the types and functions are doing.

## Start

Let's focus solely on the pure function we perform on the file contents:

```Haskell
readTuples' :: String -> String
readTuples' xs = 
  let res = map (\x -> (read x)::(Int,Int)) (lines xs)
  in  concatMap (show . fst) res
```

Let's just have it return the list of `(Int,Int)`'s:

```Haskell
readTuples' :: String -> [(Int,Int)]
readTuples' xs = map (\x -> (read x)::(Int,Int)) (lines xs)
```

```
No instance for (Writable [(Int,Int)])
``` 

Hmm. What's this? [Hakyll docs say](https://hackage.haskell.org/package/hakyll-4.7.1.0/docs/Hakyll-Core-Writable.html):

```Haskell
-- | Describes an item that can be saved to the disk
class Writable a where
    -- | Save an item to the given filepath
    write :: FilePath -> Item a -> IO ()
```

Here's an example for `String`:

```Haskell
instance Writable [Char] where
    write p = writeFile p . itemBody
```

Ok, what is `itemBody`? [Hakyll docs say](https://hackage.haskell.org/package/hakyll-4.7.1.0/docs/Hakyll-Core-Item.html):

```Haskell
data Item a = Item
    { itemIdentifier :: Identifier
    , itemBody       :: a
    } deriving (Show, Typeable)
```

Ok, so that's part of the `Item` record.

## Writing a `Writable` instance

We're not going to include an instance for [(Int,Int)]. Instead