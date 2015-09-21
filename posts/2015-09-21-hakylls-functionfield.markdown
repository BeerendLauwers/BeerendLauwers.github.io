---
title: Hakyll's functionField
---

## Introduction

Hakyll's `functionField` function is not documented anywhere, but it is pretty neat.
Let's say you have this in a Hakyll template somewhere:

```
$translate("greetings")$
```

That will get parsed as a function call to a `Context a` that expects one or more `String` arguments.
We can define such a `Context a` with `functionField`:

```Haskell
functionField :: String -> ([String] -> Item a -> Compiler String) -> Context a
```

## Example

Here's a very simple example I used:

```Haskell
postCtx :: Context String
postCtx = functionField "testing" (\args _ -> error $ show args) <> defaultContext
```

Putting the snippet below in a template and running it:

```
$testing("hello","test")$
```

Result:
```bash
[ERROR] ["hello","test"]
```

Even better is that we can also pass other contexts:

```
$testing("hello","test",title)$
```

Result:
```bash
[ERROR] ["hello","test","Hakyll's functionField"]
```

In fact, we can pass in functions as well:

```
$testing("hello","test",testing("hi"))$
```

Result:
```bash
[ERROR] ["hi"]
```

Unfortunately, that's all we can do, because of the recursive call to `applyExpr` in the template processing module:

```Haskell
    applyExpr :: TemplateExpr -> Compiler ContextField

    applyExpr (Ident (TemplateKey k)) = context' k [] x

    applyExpr (Call (TemplateKey k) args) = do
        args' <- mapM (\e -> applyExpr e >>= getString e) args
        context' k args' x

    applyExpr (StringLiteral s) = return (StringField s)
```

## Uses

`functionField` allows us to introduce a custom processing function into our templates.
For example, I want to relativize a URL in some JavaScript code, and the `relativizeURLs` function only focuses on specific tag attributes in HTML code.

With `functionField`, I'm able to expose that functionality everywhere:

```Haskell
relativizeUrl :: Context a
relativizeUrl = functionField "relativizeUrl" $ \args item ->
    case args of
        [k] -> do   route <- getRoute $ itemIdentifier item
                    return $ case route of
                        Nothing -> k
                        Just r -> rel k (toSiteRoot r)
        _   -> fail "relativizeUrl only needs a single argument"
     where
        isRel x = "/" `L.isPrefixOf` x && not ("//" `L.isPrefixOf` x)
        rel x root = if isRel x then root ++ x else x
        
-- Add the functionality to our default context
postCtx :: Context String
postCtx = relativizeUrl <> defaultContext
```

Calling `$relativizeUrl("/a/b/c/d")$` then results in a relativized URL in the compiled document.