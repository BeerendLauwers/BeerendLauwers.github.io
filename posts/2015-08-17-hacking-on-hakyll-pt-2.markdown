---
title: Hacking on Hakyll - Part 2
---

## Motivation

I'm working a Hakyll site that uses tags. 
Adding tags has been explained many times in other blogs, and that's not what this post is about.
I wanted to take the tags and another metadata element of a post, and output that in a single file all by itself.

To do so, I needed to inject the data into fields, so Hakyll could use it in its templates.

## Start

I first looked at how tag pages are generated:

```Haskell
    -- As explained at http://javran.github.io/posts/2014-03-01-add-tags-to-your-hakyll-blog.html
    tags <- buildTags "content/*/*" (fromCapture "tags/*")

    tagsRules tags $ \tag pattern -> do
        let title = "Content tagged with " ++ tag

        route idRoute
        compile $ do
            alltags <- recentFirst =<< loadAll pattern
            let ctx = constField "title" title <>
                        listField "alltags" (addTags tags postCtx) (return alltags) <>
                        defaultContext
            makeItem ""
                >>= loadAndApplyTemplate "templates/tags.html" ctx
                >>= loadAndApplyTemplate "templates/default.html" ctx
                >>= relativizeUrls
```

The interesting bits here start at `alltags <- recentFirst =<< loadAll pattern`.
The tags are loaded using `loadAll pattern`, which matches Hakyll's version of a file path.
Then they're filtered and bound to `alltags`.
Then, the tags are injected into a `listField` called `"alltags"`, which is added to the context.

In the `tags.html` template, that field is iterated over:

```
<ul>
    $for(alltags)$
        <li>
            <a href="$url$">$title$</a> - $date$
        </li>
    $endfor$
</ul>
```

The type of `listField` is: `listField :: String -> Context a -> Compiler [Item a] -> Context b`.

So, I have two questions: 

1. How can I manually construct a `listField` without a file path, just from data?
2. How can I ensure each element of the `listField` itself also has fields (like `$url` and `$title$` in the template above)?

## Type safari

### `Context a`

Ok, so I know what a `String` is, but what is a `Context a`?

```Haskell
--------------------------------------------------------------------------------
-- | The 'Context' monoid. Please note that the order in which you
-- compose the items is important. For example in
--
-- > field "A" f1 <> field "A" f2
--
-- the first context will overwrite the second. This is especially
-- important when something is being composed with
-- 'metadataField' (or 'defaultContext'). If you want your context to be
-- overwritten by the metadata fields, compose it from the right:
--
-- @
-- 'metadataField' \<\> field \"date\" fDate
-- @
--
newtype Context a = Context
    { unContext :: String -> [String] -> Item a -> Compiler ContextField
    }
```

The example is very useful: `field "A" f1 <> field "A" f2`.
So it's a description of a Hakyll field, and we can chain them together with `<>`.

### `field`

Ok, what is `field`?

```Haskell
field
    :: String                      -- ^ Key
    -> (Item a -> Compiler String) -- ^ Function that constructs a value based
                                   -- on the item
    -> Context a
field key value = field' key (fmap StringField . value)
```

Ok, so given a key (that we use in the template inbetween dollar signs) and a function that gives us a `Compiler String`, we can produce a field!

### Making a `listField`

So I can fill in the first two arguments of `listField` already: 
`listField :: String -> Context a -> Compiler [Item a] -> Context b`.

How do I get a `Compiler [Item a]`?
Browsing the Hakyll documentation, I found [http://jaspervdj.be/hakyll/reference/Hakyll-Web-Template.html](this):

> Another concrete example one may consider is the following. Given the context
> 
> ```Haskell
> listField "things" (field "thing" (return . itemBody))
>   (sequence [makeItem "fruits", makeItem "vegetables"])
> ```
>
> and a template
> 
> ```
> I like
> $for(things)$
>   fresh $thing$$sep$, and 
> $endfor$
> ```
>
> the resulting page would look like
> 
> ```HTML
> <p>
> I like
>  fresh fruits, and 
>  fresh vegetables
> </p>
> ```

Great, so with `(sequence [makeItem "fruits", makeItem "vegetables"])`, we get a `Compiler [Item a]`!

### `makeItem`

The type of `makeItem` is `a -> Compiler (Item a)`, which is what we need!

However, I would like to have a list of tuples, not a list of `String`s.
Something like `[("a",["b","c"]), ("d",["e","f","g"])]`.

So, we would like to pass the first part of the tuple to one field, and the second part to another (list)field.

But how can inspect the `a` in `Item a`?

### `itemBody`

```Haskell
data Item a = Item
    { itemIdentifier :: Identifier
    , itemBody       :: a
    } deriving (Show, Typeable)
```

Ok, so we can just apply `itemBody` to an `Item a` to get back the `a`.

## Completing the puzzle

We have some data (a list of 2-tuples) and want to pass each part of the tuple to its own field.
Let's write out something like that:

```Haskell
listField "categories" 
    (
        field "parent" (return . fst . itemBody) <>
        listField "children" 
                  (field "child" (return . itemBody)) 
                  (sequence . map makeItem . snd . itemBody)
    ) 
    (sequence [makeItem ("p1",["c1","c2"]), makeItem ("p2",["p3","p4"])])
```

Let's break it down:

```Haskell
listField "categories" (...) (sequence [makeItem ..., makeItem ...])
```

Here, we take our original data and put it in a list of Hakyll fields.
Focusing on the `(...)`:

```Haskell
(
    field "parent" (return . fst . itemBody) <>
    listField "children" 
              (field "child" (return . itemBody)) 
              (sequence . map makeItem . snd . itemBody)
)
```

From the `itemBody` (which is a single 2-tuple from the list), we create two fields: `"parent"` and `"children"`.

```Haskell
field "parent" (return . fst . itemBody)
```

If the tuple was `("p1",["c1","c2"])`, `"parent"` would have `"p1"` as its value (= its `itemBody`).

```Haskell
listField "children" 
          (field "child" (return . itemBody)) 
          (sequence . map makeItem . snd . itemBody)
```

For `"children"`, we have to make another `listField`. 
We know how this is done: after an identifier, we need to give it a `Context a` and a `Compiler [Item a]`.
Unfortunately, the type checker stops us:

```bash
   Couldn't match expected type Compiler [Item String]
               with actual type Item (a1, [a0]) -> Compiler [Item a0]
   In the third argument of listField, namely
     (sequence . map makeItem . snd . itemBody)
   In the second argument of (<>), namely
     listField
        "children"
        (field "child" (return . itemBody))
        (sequence . map makeItem . snd . itemBody)
   In the second argument of listField, namely
     (field "parent" (return . fst . itemBody)
       <>
         listField
           "children"
           (field "child" (return . itemBody))
           (sequence . map makeItem . snd . itemBody))
```

And, indeed, `listField` doesn't have access to an `Item a` like `field` does:

```Haskell
listField :: String -> Context a -> Compiler [Item a] -> Context b
field     :: String -> (Item a -> Compiler String)    -> Context a
```

Luckily, Hakyll provides us with `listFieldWith`:

```Haskell
listFieldWith
    :: String 
    -> Context a 
    -> (Item b -> Compiler [Item a]) 
    -> Context b
```

Replacing `listField` with `listFieldWith` resolves the issue, and it compiles!

Let's use this code to generate a new page:

```Haskell
create ["test.html"] $ do
  route idRoute
  compile $ do
      let ctx =
              listField "parents" 
                        (
                         field "parent" (return . fst . itemBody) <>
                         listFieldWith "children" 
                                       (field "child" (return . itemBody)) 
                                       (sequence . map makeItem . snd . itemBody)
                        ) 
                        (sequence [makeItem ("p1",["c1","c2"]), 
                                   makeItem ("p2",["p3","p4"])]) <>
              defaultContext
      makeItem ""
          >>= loadAndApplyTemplate "templates/testing.html" ctx
          >>= loadAndApplyTemplate "templates/default.html" defaultContext
          >>= relativizeUrls
```

Here's `testing.html`:

```
 $for(parents)$
 $parent$: 
   $for(children)$
    $child$
   $endfor$
 $endfor$
```

And here's the output:

```HTML
 p1: 
   
    c1
   
    c2
   
 
 p2: 
   
    p3
   
    p4
```

## Conclusion

This one was more complex to find out, but shows quite clearly how Hakyll fields are built and composed.
Now we can generate static pages from arbitrary data!