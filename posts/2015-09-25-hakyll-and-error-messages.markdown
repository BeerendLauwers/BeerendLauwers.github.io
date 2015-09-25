---
title: Hakyll and error messages
---

## The problem

During site compilation, Hakyll may stop with an error message like the following:

```bash
[ERROR] Missing field $translationFor$ in context for item content/fr/index.html
```

However, the *actual* reason may be something different entirely.

## The solution

To find out, run your build command again with the `-v` flag:

```bash
[DEBUG] Hakyll.Core.Compiler.Internal: Alternative failed: 
    translation set en not found.
[DEBUG] Hakyll.Core.Compiler.Internal: Alternative failed: 
    Missing field $translationFor$ in context for item content/fr/index.html
[DEBUG] Hakyll.Core.Compiler.Internal: Adding dependency: 
    IdentifierDependency content/fr/index.html
[DEBUG] Hakyll.Core.Compiler.Internal: Alternative failed: 
    Missing field $translationFor$ in context for item content/fr/index.html
[ERROR] Missing field $translationFor$ in context for item content/fr/index.html
```

(In my case, I deliberately tried to get an error to test a [`functionField`](/posts/2015-09-21-hakylls-functionfield.html).)

## The internals

The reason for this is that `Compiler` has an `Alternative` instance that will check for:

```Haskell
instance Alternative Compiler where
    empty   = compilerThrow []
    x <|> y = compilerCatch x $ \es -> do
        logger <- compilerLogger <$> compilerAsk
        forM_ es $ \e -> compilerUnsafeIO $ Logger.debug logger $
            "Hakyll.Core.Compiler.Internal: Alternative failed: " ++ e
        y
```

If a `Context` encounters a failure (wrong amount of arguments, a lookup gave back `Nothing`, anything that would merit failing with a call to `fail`), the error message will be logged to the debug log.
Then, it tries the next option (`y`).

The `Monoid` instance for `Compiler` uses `<|>` here:

```Haskell
instance Monoid (Context a) where
    mempty                          = missingField
    mappend (Context f) (Context g) = Context $ \k a i -> f k a i <|> g k a i
```

In other words, `Context`s are tried one after the other, until there is a success.
`missingField` is a `Context` wrapper around failure:

```Haskell
missingField :: Context a
missingField = Context $ \k _ i -> fail $
    "Missing field $" ++ k ++ "$ in context for item " ++
    show (itemIdentifier i)
```

The error message looks familiar, no?

It's also the last element in the definition of `defaultContext`:

```Haskell
defaultContext :: Context String
defaultContext =
    bodyField     "body"     `mappend`
    metadataField            `mappend`
    urlField      "url"      `mappend`
    pathField     "path"     `mappend`
    titleField    "title"    `mappend`
    missingField
```

Hence, its error message will be the last one in the debug log.
Why are the others not shown? 
This is discussed in [this Github issue](https://github.com/jaspervdj/hakyll/issues/126):

> **jaspervdj**
>
> Some discussion. 
> The problem is that the user composes a lot of different compilers using `Context`s `mappend`, which is `<|>` for `Compiler`. 
> If we print the errors for each alternative, this would cause a huge amount of output -- there's no real way to figure out which output matters and which not.
> 
> Would it be sensible to always print the last error, but print all errors when running with `-v`?

So, in short: if an error occurs, always re-run with `-v` to get to the real source of the problem.