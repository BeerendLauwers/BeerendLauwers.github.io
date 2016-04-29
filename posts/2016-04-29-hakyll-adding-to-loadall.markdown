---
title: Hakyll: Adding a field to each item loaded by loadAll
---

## Introduction

Let's say we've loaded a bunch of posts with `loadAll`:

```Haskell
match "index.html" $ do
        route idRoute
        compile $ do
            myPosts <- loadAll "posts/*"
            ...
```

We can iterate over these posts in a template file if we expose them with `listField`:

```Haskell
match "index.html" $ do
        route idRoute
        compile $ do
            myPosts <- loadAll "posts/*"
            let indexContext =
                   listField "allPostsWithExtraField" defaultContext (return myPosts) <>
                   defaultContext
            ...
```

In a template, we can do:

```
$for(allPostsWithExtraField)$
    The title of one of the posts: $title$
$endfor$
```

Now, let's say we want to have access to another field inside that loop, perhaps one that is computed from some existing metadata of the item:

```
$for(allPostsWithExtraField)$
    The title of one of the posts: $title$
    And here's the URL without its file extension: $url-plain$
$endfor$
```

Obviously, we don't want to manually introduce a metadata field for this. But how do we "add" another field?

## Jasper to the rescue

I struggled with this, so I cheated and asked [Jasper Van der Jeugt](https://jaspervdj.be/), the creator of Hakyll, on the [Hakyll Google Group](https://groups.google.com/forum/#!topic/hakyll/7Kve_5Hd65I).

Jasper provided a complete working solution:

> First we define a function which does what you want: 
>
> ```Haskell
> import System.FilePath (dropExtension) 
> 
> urlPlainField :: Context a 
> urlPlainField = field "url-plain" $ \item -> do 
> mbFilePath <- getRoute (itemIdentifier item) 
> case mbFilePath of 
>  Nothing       -> return "???" 
>  Just filePath -> return $ toUrl $ dropExtension filePath 
> ```
>
> And then we add it to some `Context` that we define for the use in the list: 
>
> ```Haskell
> postCtx :: Context String 
> postCtx = 
>  urlPlainField `mappend` 
>  defaultContext 
> ```
>
> Now, the following should enable you to use the new `$url-plain$`: 
>
> ```Haskell
> let ctx = listField "how-do-i-posts" postCtx (return howDoIPosts)
> ```

In our example, this solution looks like the following:

```Haskell
match "index.html" $ do
        route idRoute
        compile $ do
            myPosts <- loadAll "posts/*"
            let postContext = urlPlainField <> defaultContext
            let indexContext =
                   listField "allPostsWithExtraField" postContext (return myPosts) <>
                   defaultContext
            ...
```

So, we provided `listField` with a context that includes `urlPlainField`.

## Why does this work?

Checking a proposed solution is always easier than finding it yourself, and I wanted to know *why* this is the solution.
So, into the source code we go!

In `Hakyll.Web.Template.Context`, we find:

```Haskell
--------------------------------------------------------------------------------
listField :: String -> Context a -> Compiler [Item a] -> Context b
listField key c xs = listFieldWith key c (const xs)


--------------------------------------------------------------------------------
listFieldWith
    :: String -> Context a -> (Item b -> Compiler [Item a]) -> Context b
listFieldWith key c f = field' key $ fmap (ListField c) . f
```

So, `listField` is a wrapper for `listFieldWith`.
`listFieldWith` does the following:

* Create a field with a key (our `"allPostsWithExtraField"`)
* That field is a `ListField`, hence it being used as the constructor a bit further in the function.
* `f` is first applied to some incoming `Item b`, and yields a `Compiler [Item a]`, which we will call `is`.
* Then, we `fmap` over that `Compiler [Item a]` and apply the `ListField` constructor to it. The constructor already has `c` applied to it (which is the context we passed! In our case, `postContext`).
* The resulting `ListField c is` value is then applied to `field' key`, producing a `Context`.

Ok, so now the context is being carried around somewhere.
But we haven't actually *done* anything with it yet.
So, we'll need to find out where a `ListField` is pattern matched and deconstructed.

That happens to be in `Hakyll.Web.Template`, in a `where` clause in the `applyTemplate'` function:

```Haskell
...

applyElem (For e b s) = applyExpr e >>= \cf -> case cf of
    StringField str  -> fail $
        "Hakyll.Web.Template.applyTemplateWith: expected ListField but " ++
        "got StringField for expr " ++ show e ++ ", namely " ++ str
    ListField c xs -> do
        sep <- maybe (return "") go s
        bs  <- mapM (applyTemplate' b c) xs
        return $ intercalate sep bs
        
...
```

We're interested in what is going to be done with `c`, which is our context.

As you can see, it's used in a call to `applyTemplate'` that is mapped over all the items (called `xs`) inside our `ListField`.

So, what happens, is:

* We load a bunch of posts with `myPosts <- loadAll "posts/*"`, which yields us an `[Item a]`.
* We put them in a `ListField` with the `listField` function.
* We also provide a context that will be available to each `Item a`.
* In `applyElem`, the template `b` (which is just everything between `$for$` and `$endfor$`, the **b**ody), our context `c` and an `Item a`, are all passed to `applyTemplate'`.
* So, that means the context we passed in is now available in that template.

And how can we then use something from the `Item` to generate some new value?
Let's take the definition of `urlPlainField` again for that:

```Haskell
urlPlainField :: Context a 
urlPlainField = field "url-plain" $ \item -> do 
mbFilePath <- getRoute (itemIdentifier item) 
 case mbFilePath of 
  Nothing       -> return "???" 
  Just filePath -> return $ toUrl $ dropExtension filePath
```

Each `Context` always gets access to the item that was passed to `applyTemplate'`.
So, we can just get the `Identifier` of the item with `itemIdentifier`.
And with an `Identifier`, we can access:

* Metadata with `getMetadata` .
* The filepath of the item (if any) with `toFilePath`.
* The version with `identifierVersion`.
* The route for the item with `getRoute`.
* Create a snapshot of your item with `saveSnapshot <snapshotName>`.
* Load a snapshot of your item with `loadSnapshot <snapshotName>`.


