---
title: Writing a Servant backend for AnnotatorJS
---

## Getting `servant`

First, I followed the `servant` tutorial at [http://haskell-servant.github.io/tutorial/server.html](http://haskell-servant.github.io/tutorial/server.html).

But I couldn't go to `servant-examples` and do `cabal install --only-dependencies` :(

`cabal install servant` in root, then.

## Writing out an API

```Haskell
module ServantAnnotator where

import Servant
```

```
ghci ServantAnnotator.hs
```

```Haskell
ServantAnnotator.hs:3:8:
    Could not find module `Servant'
    Locations searched:
      Servant.hs
      Servant.lhs
Failed, modules loaded: none.
```

Hmm. `ghci` by itself doesn't seem to respect sandboxes. Something else, then:

```
cabal repl ServantAnnotator.hs
Prelude>
```

Oh, that didn't load anything.

```
:l ServantAnnotator.hs
```

Ok, that works. Let's add the [storage API description](http://docs.annotatorjs.org/en/latest/modules/storage.html).

```Haskell
type AnnotationAPI = "api" :> Get '[JSON] Text
                :<|> "api" :> "annotations" :> Capture "id" Identifier :> Get '[JSON] Annotation
                :<|> "api" :> "annotations" :> Post '[JSON] Annotation
                :<|> "api" :> "annotations" :> Capture "id" Identifier :> Put '[JSON] Annotation
                :<|> "api" :> "annotations" :> Capture "id" Identifier :> Delete
                :<|> "api" :> "search" :> QueryParam "text" Text :> QueryParam "limit" Int :> Get '[JSON] SearchResponse
```
                
*Nifty*.

For the moment, I'm just going to store the JSON as-is as some `Text`.
It's a little unclear what an `Annotation`'s schema is, so I'll have to strongly-type that later.

```Haskell
type Annotation = Text
type Identifier = Text

data SearchResponse = SearchResponse 
                        { total :: Int
                        , rows  :: [Annotation]
                        } deriving (Eq, Show, Generic)

instance ToJSON SearchResponse
```

## Adding dummy data

Now let's add some dummy data:

```Haskell
dummyAnnotations :: [Annotation]
dummyAnnotations = 
   [ "Test A"
   , "Tesb B"
   ]
```
   
Ok, now let's show that in GHCI:

```Haskell
*ServantAnnotator> dummyAnnotations
Loading package array-0.5.0.0 ... linking ... done.
Loading package deepseq-1.3.0.2 ... linking ... done.
Loading package bytestring-0.10.4.0 ... linking ... done.
Loading package Win32-2.3.0.2 ... linking ... done.
Loading package transformers-0.3.0.0 ... linking ... done.
Loading package old-locale-1.0.0.6 ... linking ... done.
Loading package time-1.4.2 ... linking ... done.
Loading package pretty-1.1.1.1 ... linking ... done.
Loading package filepath-1.3.0.2 ... linking ... done.
Loading package directory-1.2.1.0 ... linking ... done.
Loading package containers-0.5.5.1 ... linking ... done.
Loading package text-1.2.0.6 ... linking ... done.
Loading package hashable-1.2.3.2 ... linking ... done.
Loading package scientific-0.3.3.8 ... linking ... done.
Loading package attoparsec-0.12.1.6 ... linking ... done.
Loading package dlist-0.7.1.1 ... linking ... done.
Loading package mtl-2.1.3.1 ... linking ... done.
Loading package syb-0.4.4 ... linking ... done.
Loading package template-haskell ... linking ... done.
Loading package unordered-containers-0.2.5.1 ... linking ... done.
Loading package primitive-0.6 ... linking ... done.
Loading package vector-0.10.12.3 ... linking ... done.
Loading package aeson-0.8.0.2 ... linking ... done.
Loading package random-1.1 ... linking ... done.
Loading package transformers-compat-0.4.0.3 ... linking ... done.
Loading package MonadRandom-0.3.0.2 ... linking ... done.
Loading package stm-2.4.4 ... linking ... done.
Loading package StateVar-1.1.0.0 ... linking ... done.
Loading package nats-1 ... linking ... done.
Loading package semigroups-0.16.2.2 ... linking ... done.
Loading package void-0.7 ... linking ... done.
Loading package contravariant-1.3.1.1 ... linking ... done.
Loading package tagged-0.7.3 ... linking ... done.
Loading package distributive-0.4.4 ... linking ... done.
Loading package comonad-4.2.6 ... linking ... done.
Loading package semigroupoids-4.3 ... linking ... done.
Loading package bifunctors-4.2.1 ... linking ... done.
Loading package exceptions-0.8.0.2 ... linking ... done.
Loading package prelude-extras-0.4 ... linking ... done.
Loading package profunctors-4.4.1 ... linking ... done.
Loading package free-4.11 ... linking ... done.
Loading package transformers-base-0.4.4 ... linking ... done.
Loading package monad-control-1.0.0.4 ... linking ... done.
Loading package either-4.3.4.1 ... linking ... done.
Loading package blaze-builder-0.4.0.1 ... linking ... done.
Loading package case-insensitive-1.2.0.4 ... linking ... done.
Loading package http-types-0.8.6 ... linking ... done.
Loading package mmorph-1.0.4 ... linking ... done.
Loading package parsec-3.1.9 ... linking ... done.
Loading package network-uri-2.6.0.3 ... linking ... done.
Loading package safe-0.3.9 ... linking ... done.
Loading package double-conversion-2.0.1.0 ... 
<interactive>: stdc++: The specified module could not be found.
can't load .so/.DLL for: stdc++.dll (addDLL: could not load DLL)
```

Oh, goodie. Let's google. Results:

 * [minghc Github issue](https://github.com/fpco/minghc/issues/5)
 * [haskell-game sdl2 Github issue](https://github.com/haskell-game/sdl2/issues/41)
 * [GHC's Trac](https://ghc.haskell.org/trac/ghc/ticket/3242)
 
So it's a Windows-only bug that's existed for over four years. *Awesome*.

More googling:

 * [Reddit thread](http://www.reddit.com/r/haskell/comments/k2idm/eclipsefp_210_released_with_hoogle_and_hlint/), CTRL+F "double-conversion"
 * [EclipseFP FAQ](http://eclipsefp.github.io/faq.html)
 
The latter says:

> Q: I'm on Windows and/or the error is more like `Loading package double-conversion-0.2.0.0 ... can't load .so/.DLL for: stdc++ (addDLL: could not load DLL) ghc.exe: stdc++: The specified module could not be found.`.
> 
> A: Try the workaround outlined here:
> ```
> cabal install blaze-textual --user -fnative --reinstall
> cabal install aeson --user --reinstall
> ```
> Then, restart Eclipse to force scion-browser to be rebuilt.
                
Ok, let's reinstall `aeson`:

```
cabal install aeson --reinstall
```

Didn't work.

Let's just add this DLL to the `bin` folder. Where can I find it? 

> [You can get these files from /mingwXX/bin directory, where XX is 32 or 64](http://sourceforge.net/p/mingw-w64/mailman/message/31883429/)

Ok, install mingw and copy it over. mingw provides `libstdc++-6.dll`.  
I'll copy that over to `C:\Program Files\MinGHC-7.8.4\msys-1.0.1\bin`.

Doesn't work, same error. Git `bin` (`C:\Program Files (x86)\Git\bin`) ?  
Nope. Rename to `stdc++.dll` ? No dice.

Ok, let's dockerize this and get it to run there, I guess. But first; let's finish up the code, we can still get it to typecheck.

## Writing the server-side functions

```Haskell
server :: Server AnnotationAPI
server = return "API information" -- Would this be automatically generated? No mention of it in the tutorial :(
    :<|> return readAnnotation
    :<|> return createAnnotation
    :<|> return updateAnnotation
    :<|> return deleteAnnotation
    :<|> return searchAnnotation
```

Easy enough. Let's define these functions:

```Haskell
readAnnotation :: Identifier -> EitherT ServantErr IO Annotation
readAnnotation id = undefined
createAnnotation = undefined
updateAnnotation = undefined
deleteAnnotation = undefined
searchAnnotation = undefined
```

```Haskell
ServantAnnotator.hs:45:33:
    Not in scope: type constructor or class `EitherT'
    Perhaps you meant `Either' (imported from Prelude)
Failed, modules loaded: none.
```

Huh. Let's add it:

```Haskell
import Control.Monad.Trans.Either
```

```Haskell
Prelude> :r
ServantAnnotator.hs:38:10:
    Couldn't match type `EitherT ServantErr IO Text'
                  with `Identifier -> EitherT ServantErr IO Annotation'
    Expected type: Server AnnotationAPI
      Actual type: EitherT ServantErr IO Text
                   :<|> ((Text -> Identifier -> EitherT ServantErr IO Annotation)
                         :<|> (EitherT ServantErr IO Text
                               :<|> ((Text -> t0 -> t1)
                                     :<|> ((Text -> t2 -> t3)
                                           :<|> (Maybe Text -> Maybe Int -> t4 -> t5)))))
    In the expression:
      return "API information"
      :<|>
        return readAnnotation
        :<|>
          return createAnnotation
          :<|>
            return updateAnnotation
            :<|> return deleteAnnotation :<|> return searchAnnotation
    In an equation for `server':
        server
          = return "API information"
            :<|>
              return readAnnotation
              :<|>
                return createAnnotation
                :<|>
                  return updateAnnotation
                  :<|> return deleteAnnotation :<|> return searchAnnotation
Failed, modules loaded: none.
```

Hmm. `readAnnotation` expects an identifier, doesn't it? I guess I'll just remove that...

```Haskell
readAnnotation :: EitherT ServantErr IO Annotation
readAnnotation = undefined
```

That typechecks. Next! Hmm, this keeps on giving typechecker errors.
There has to be an easier way to figure out what it expects.
This example pushes the `return` inside the function:

> From [http://haskell-servant.github.io/tutorial/server.html](http://haskell-servant.github.io/tutorial/server.html)
> ```Haskell
> server :: Server API
> server = position
>     :<|> hello
>     :<|> marketing
> 
>   where position :: Int -> Int -> EitherT ServantErr IO Position
>         position x y = return (Position x y)
> 
>         hello :: Maybe String -> EitherT ServantErr IO HelloMessage
>         hello mname = return . HelloMessage $ case mname of
>           Nothing -> "Hello, anonymous coward"
>           Just n  -> "Hello, " ++ n
> 
>         marketing :: ClientInfo -> EitherT ServantErr IO Email
>         marketing clientinfo = return (emailForClient clientinfo)
> ```

Let's do it like that:

```Haskell
server :: Server AnnotationAPI
server = return "API information"
    :<|> readAnnotation
    :<|> createAnnotation
    :<|> updateAnnotation
    :<|> deleteAnnotation
    :<|> searchAnnotation
    
readAnnotation :: Identifier -> EitherT ServantErr IO Annotation
readAnnotation id = return undefined
createAnnotation = return undefined
updateAnnotation = return undefined
deleteAnnotation = return undefined
searchAnnotation = return undefined
```

Great, that typechecks. Let's inspect what it expects for the rest:

```Haskell
*ServantAnnotator> :t createAnnotation
createAnnotation :: EitherT ServantErr IO a
*ServantAnnotator> :t updateAnnotation
updateAnnotation :: Text -> a
*ServantAnnotator> :t deleteAnnotation
deleteAnnotation :: Text -> a
*ServantAnnotator> :t searchAnnotation
searchAnnotation :: Maybe Text -> a
```

Looks good! Let's use this pattern.

```Haskell
readAnnotation :: Identifier -> EitherT ServantErr IO Annotation
readAnnotation id = return undefined

createAnnotation :: EitherT ServantErr IO Annotation
createAnnotation = return undefined

updateAnnotation :: Identifier -> EitherT ServantErr IO Annotation
updateAnnotation = return undefined

deleteAnnotation :: Identifier -> EitherT ServantErr IO Annotation
deleteAnnotation = return undefined

searchAnnotation = return undefined
```

```Haskell
ServantAnnotator.hs:38:10:
    Couldn't match type `ServerT Delete (EitherT ServantErr IO)'
                   with `EitherT ServantErr IO Annotation'
    Expected type: Server AnnotationAPI
      Actual type: EitherT ServantErr IO Text
                   :<|> ((Identifier -> EitherT ServantErr IO Annotation)
                         :<|> (EitherT ServantErr IO Annotation
                               :<|> ((Identifier -> EitherT ServantErr IO Annotation)
                                     :<|> ((Identifier -> EitherT ServantErr IOAnnotation)
                                           :<|> (Maybe Text
                                                 -> Maybe Int
                                                 -> EitherT ServantErr IO SearchResponse)))))
    In the expression:
      return "API information"
      :<|>
        readAnnotation
        :<|>
          createAnnotation
          :<|> updateAnnotation :<|> deleteAnnotation :<|> searchAnnotation
    In an equation for `server':
        server
          = return "API information"
            :<|>
              readAnnotation
              :<|>
                createAnnotation
                :<|> updateAnnotation :<|> deleteAnnotation :<|> searchAnnotation
```

Hmm. I think it might be best to find an example. [Google gets me this](https://github.com/krdlab/examples/blob/master/haskell-servant-webapp/src/Main.hs). Looks good.

Oh, I didn't include a `ReqBody` in my API, let's do that:

```Haskell
type AnnotationAPI = "api" :> Get '[JSON] Text
                :<|> "api" :> "annotations" :> Capture "id" Identifier :> Get '[JSON] Annotation
                :<|> "api" :> "annotations" :> ReqBody Annotation :> Post '[JSON] Annotation
                :<|> "api" :> "annotations" :> Capture "id" Identifier :> ReqBody Annotation :> Put '[JSON] Annotation
                :<|> "api" :> "annotations" :> Capture "id" Identifier :> Delete
                :<|> "api" :> "search" :> QueryParam "text" Text :> QueryParam "limit" Int :> Get '[JSON] SearchResponse
```

```Haskell
Prelude> :r
[1 of 1] Compiling ServantAnnotator ( ServantAnnotator.hs, interpreted )

ServantAnnotator.hs:25:56:
    The first argument of `ReqBody' should have kind `[*]',
      but `Annotation' has kind `*'
    In the type `("api" :> Get '[JSON] Text)
                 :<|>
                 (("api"
                   :>
                   ("annotations"
                    :> (Capture "id" Identifier :> Get '[JSON] Annotation)))
                  :<|>
                  (("api"
                    :>
                    ("annotations" :> (ReqBody Annotation :> Post '[JSON] Annotation)))
                   :<|>
                   (("api"
                     :>
                     ("annotations"
                      :>
                      (Capture "id" Identifier
                       :> (ReqBody Annotation :> Put '[JSON] Annotation))))
                    :<|>
                    (("api" :> ("annotations" :> (Capture "id" Identifier :> Delete)))
                     :<|>
                     ("api"
                      :>
                      ("search"
                       :>
                       (QueryParam "text" Text
                        :> (QueryParam "limit" Int :> Get '[JSON] SearchResponse))))))))'
    In the type declaration for `AnnotationAPI'
```

Hmm. That example is probably for an old version. [Let's see what Hackage has to say.](https://hackage.haskell.org/package/servant-0.4.0/docs/Servant-API-ReqBody.html)

> Example
> ```Haskell
> >>> -- POST /books
> >>> type MyApi = "books" :> ReqBody '[JSON] Book :> Post '[JSON] Book
> ```

OK, we just need to add `'[JSON]` to our `ReqBody`'s as well.
Great, that compiles.

Now to rewrite the handlers:

```Haskell
server :: Server AnnotationAPI
server = apiDetails
    :<|> readAnnotation
    :<|> createAnnotation
    :<|> updateAnnotation
    :<|> deleteAnnotation
    :<|> searchAnnotation
    
apiDetails :: EitherT ServantErr IO Text
apiDetails = return undefined
    
readAnnotation :: Identifier -> EitherT ServantErr IO Annotation
readAnnotation id = return undefined

createAnnotation :: Annotation -> EitherT ServantErr IO Annotation
createAnnotation ann = return undefined

updateAnnotation :: Identifier -> Annotation -> EitherT ServantErr IO Annotation
updateAnnotation id ann = return undefined

deleteAnnotation :: Identifier -> EitherT ServantErr IO ()
deleteAnnotation id = return undefined

searchAnnotation :: Maybe Text -> Maybe Int -> EitherT ServantErr IO SearchResponse
searchAnnotation = return undefined
```

That works save for the `apiDetails`:

```Haskell
ServantAnnotator.hs:38:10:
    Couldn't match type `EitherT ServantErr IO Text'
                  with `IO Annotation'
    Expected type: Server AnnotationAPI
      Actual type: EitherT ServantErr IO Text
                   :<|> ((Identifier -> IO Annotation)
                         :<|> ((Annotation -> IO Annotation)
                               :<|> ((Identifier -> Annotation -> IO Annotation)

                                     :<|> ((Identifier -> IO ())
                                           :<|> (Maybe Text -> Maybe Int -> IO SearchResponse)))))
    In the expression:
      apiDetails
      :<|>
        readAnnotation
        :<|>
          createAnnotation
          :<|> updateAnnotation :<|> deleteAnnotation :<|> searchAnnotation
    In an equation for `server':
        server
          = apiDetails
            :<|>
              readAnnotation
              :<|>
                createAnnotation
                :<|> updateAnnotation :<|> deleteAnnotation :<|> searchAnnotation
```

*Man*, it would be nice if we could generate functions stubs from the API type. 
Just a naive function name, the type and a `return undefined` as the function body.

Ok, [reviewing the tutorial](http://haskell-servant.github.io/tutorial/server.html) tell me I can just do this:

``` Haskell
type UserAPI = "users" :> Get '[JSON] [User]
          :<|> "albert" :> Get '[JSON] User
          :<|> "isaac" :> Get '[JSON] User
          
isaac :: User
isaac = User "Isaac Newton" 372 "isaac@newton.co.uk" (fromGregorian 1683 3 1)

albert :: User
albert = User "Albert Einstein" 136 "ae@mc2.org" (fromGregorian 1905 12 1)

users :: [User]
users = [isaac, albert]

server :: Server UserAPI
server = return users
    :<|> return albert
    :<|> return isaac
``` 

Let's copy that pattern:

```Haskell
server :: Server AnnotationAPI
server = return apiDetails
    :<|> readAnnotation
    :<|> createAnnotation
    :<|> updateAnnotation
    :<|> deleteAnnotation
    :<|> searchAnnotation
    
apiDetails :: Text
apiDetails = "API data"
```

Great, that typechecks. The `Delete` endpoint is complaining now, though:

```Haskell
ServantAnnotator.hs:38:10:
    Couldn't match type `ServerT Delete (EitherT ServantErr IO)'
                  with `EitherT ServantErr IO ()'
    Expected type: Server AnnotationAPI
      Actual type: EitherT ServantErr IO Text
                   :<|> ((Identifier -> EitherT ServantErr IO Annotation)
                         :<|> ((Annotation -> EitherT ServantErr IO Annotation)
                               :<|> ((Identifier
                                      -> Annotation -> EitherT ServantErr IO Annotation)
                                     :<|> ((Identifier -> EitherT ServantErr IO())
                                           :<|> (Maybe Text
                                                 -> Maybe Int
                                                 -> EitherT ServantErr IO SearchResponse)))))
    In the expression:
      return apiDetails
      :<|>
        readAnnotation
        :<|>
          createAnnotation
          :<|> updateAnnotation :<|> deleteAnnotation :<|> searchAnnotation
    In an equation for `server':
        server
          = return apiDetails
            :<|>
              readAnnotation
              :<|>
                createAnnotation
                :<|> updateAnnotation :<|> deleteAnnotation :<|> searchAnnotation
```

Ok, let's change the type:

```Haskell
deleteAnnotation :: Identifier -> ServerT Delete (EitherT ServantErr IO)
```

```Haskell
ServantAnnotator.hs:58:23:
    Couldn't match expected type `ServerT
                                    Delete (EitherT ServantErr IO)'
                with actual type `m0 a0'
    The type variables `m0', `a0' are ambiguous
    In the expression: return undefined
    In an equation for `deleteAnnotation':
        deleteAnnotation id = return undefined
Failed, modules loaded: none.
```

Dammit. Let's google for more example pertaining to `Delete` queries.
[Here we go](https://github.com/haskell-servant/servant-server/blob/master/example/greet.hs).
Let's load this up into a file (ripped out the serving parts):

```Haskell
{-# LANGUAGE DataKinds #-}
{-# LANGUAGE PolyKinds #-}
{-# LANGUAGE TypeFamilies #-}
{-# LANGUAGE DeriveGeneric #-}
{-# LANGUAGE TypeOperators #-}
{-# LANGUAGE OverloadedStrings #-}

import Data.Aeson
import Data.Monoid
import Data.Proxy
import Data.Text
import GHC.Generics
import Network.Wai
import Network.Wai.Handler.Warp

import Servant

-- * Example

-- | A greet message data type
newtype Greet = Greet { _msg :: Text }
  deriving (Generic, Show)

instance FromJSON Greet
instance ToJSON Greet

-- API specification
type TestApi =
       -- GET /hello/:name?capital={true, false}  returns a Greet as JSON
       "hello" :> Capture "name" Text :> QueryParam "capital" Bool :> Get '[JSON] Greet

       -- POST /greet with a Greet as JSON in the request body,
       --             returns a Greet as JSON
  :<|> "greet" :> ReqBody '[JSON] Greet :> Post '[JSON] Greet

       -- DELETE /greet/:greetid
  :<|> "greet" :> Capture "greetid" Text :> Delete

-- Server-side handlers.
--
-- There's one handler per endpoint, which, just like in the type
-- that represents the API, are glued together using :<|>.
--
-- Each handler runs in the 'EitherT (Int, String) IO' monad.
server :: Server TestApi
server = helloH :<|> postGreetH :<|> deleteGreetH

  where helloH name Nothing = helloH name (Just False)
        helloH name (Just False) = return . Greet $ "Hello, " <> name
        helloH name (Just True) = return . Greet . toUpper $ "Hello, " <> name

        postGreetH greet = return greet

        deleteGreetH _ = return ()
```

Nope, same issue:

```Haskell
ServantAnnotator.hs:46:10:
    Couldn't match type `ServerT
                           Delete (Control.Monad.Trans.Either.EitherT ServantErr IO)'
                  with `m0 ()'
    The type variable `m0' is ambiguous
    Expected type: Server TestApi
      Actual type: (Text
                    -> Maybe Bool
                    -> Control.Monad.Trans.Either.EitherT ServantErr IO Greet)
                   :<|> ((Greet
                          -> Control.Monad.Trans.Either.EitherT ServantErr IO Greet)
                         :<|> (Text -> m0 ()))
    In the expression: helloH :<|> postGreetH :<|> deleteGreetH
    In an equation for `server':
        server
          = helloH :<|> postGreetH :<|> deleteGreetH
          where
              helloH name Nothing = helloH name (Just False)
              helloH name (Just False) = return . Greet $ "Hello, " <> name
              helloH name (Just True)
                = return . Greet . toUpper $ "Hello, " <> name
              postGreetH greet = return greet
              deleteGreetH _ = return ()
Failed, modules loaded: none.
```

Am I using a bad version of GHC or something?
Let's check what version of LTS Haskell I'm at.
`2.13`. That uses `servant-0.2.2`.
Let's switch to `2.14` and reinstall:

```Haskell
D:\Persoonlijk\Haskell\servant>cabal update
Warning: C:\Users\blauwers\AppData\Roaming\cabal\config: Unrecognized field
constraints on line 174
Downloading the latest package list from stackage-lts-2.14

D:\Persoonlijk\Haskell\servant>
D:\Persoonlijk\Haskell\servant>
D:\Persoonlijk\Haskell\servant>cabal install servant --reinstall -j2
Warning: C:\Users\blauwers\AppData\Roaming\cabal\config: Unrecognized field
constraints on line 174
Resolving dependencies...
In order, the following will be installed:
servant-0.4.1 (reinstall)
servant-server-0.4.1 (reinstall)
Warning: Note that reinstalls are always dangerous. Continuing anyway...
Notice: installing into a sandbox located at
D:\Persoonlijk\Haskell\servant\.cabal-sandbox
Configuring servant-0.4.1...
Building servant-0.4.1...
Installed servant-0.4.1
Configuring servant-server-0.4.1...
Building servant-server-0.4.1...
Installed servant-server-0.4.1
```

Ok. Let's retry.
Nope.


## Dockerizing it


