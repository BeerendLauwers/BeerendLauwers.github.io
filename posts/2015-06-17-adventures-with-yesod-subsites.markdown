---
title: An adventure with Yesod subsites
---

## Preface

This is a braindump during my development of a Yesod subsite a month or two ago.
It was surprisingly hard to find decent examples that actually worked, and many of the extremely obscure error messages were hard to solve.
[The source code can be found here.](https://github.com/beerendlauwers/haskell-datasource)
So, I hope that someone finds this useful.

## The braindump

### Setting things up

First off, ensure your IDE automatically changes tabs to spaces. 1 tab = 4 spaces.

Second, I googled "yesod subsite" and got this: [http://www.yesodweb.com/book/creating-a-subsite](http://www.yesodweb.com/book/creating-a-subsite).

Where do I place the files? No idea. Let's google a little more. Nope, no results. I guess I'll place the new subsite `HelloSub.hs` in my newly scaffolded site.

`HelloSub.Data` goes in `HelloSub/Data.`

```Haskell
HelloSub.hs:16:67: Not in scope: type constructor or class `IO'
```

Off to a good start.

```Haskell
import Prelude (IO)
```

```Haskell
HelloSub.hs:17:20: Not in scope: `$'
```

Nope. Let's try:

```Haskell
import Prelude (IO,($))
```

Ok, that compiles.

The example seems to overwrite the routes. 
I guess I'll copy-paste that into the route config file. 
Would be nice if there was a tutorial that started from the scaffolded site.

```Haskell
Application.hs:60:36: Warning:
    Fields of `App' not initialised: getHelloSub
    In the expression: App {..}
    In an equation for `mkFoundation':
        mkFoundation appConnPool = App {..}
    In the expression:
      do { appHttpManager <- newManager;
           appLogger <- newStdoutLoggerSet defaultBufSize >>= makeYesodLogger;
           appStatic <- (if appMutableStatic appSettings then
                             staticDevel
                         else
                             static)
                          (appStaticDir appSettings);
           let mkFoundation appConnPool = ...
               tempFoundation
                 = mkFoundation $ error "connPool forced in tempFoundation"
               ....;
           .... }
```

Man, I hope that doesn't break things.

```
ghc: C:\yesod\datasource\dist\build\HSdatasource-0.0.0.o: unknown symbol `datasourcezm0zi0zi0_HelloSub_zdfYesodSubDispatchHelloSubHandlerT_closure'
```

Man, it broke things. 
I guess I'll google "yesod unknown symbol". 
Result: [http://stackoverflow.com/questions/11485555/build-failure-during-yesod-devel](http://stackoverflow.com/questions/11485555/build-failure-during-yesod-devel)

Hmm: 

> "Unknown symbol errors when running yesod devel are often the result of failing to include a module in exposed-modules or other-modules in your application's cabal file."

Guess I'll add the modules to the .cabal file?

Woo, that worked!

Ok, let's actually start development now.

### Hacking together a form

Let's try out some form input. [http://www.yesodweb.com/book/forms](http://www.yesodweb.com/book/forms)

Copy over a whole bunch of stuff.

This is pretty straightforward, but I haven't looked at the compilation log yet :D

```Haskell
HelloSub\Data.hs:18:31:
    Not in scope: type constructor or class `Int'
    Perhaps you meant `Ints' (imported from Yesod)

HelloSub\Data.hs:19:32:
    Not in scope: type constructor or class `Int'
    Perhaps you meant `Ints' (imported from Yesod)

HelloSub\Data.hs:21:12:
    Not in scope: type constructor or class `Show'
```
    
Man, this just gets better and better. Doesn't Yesod use `classy-prelude`? 

```Haskell
import ClassyPrelude
```

Ok, fixed.

```Haskell
HelloSub.hs:25:25:
    Not in scope: type constructor or class `Handler'
    Perhaps you meant `HandlerT' (imported from Yesod)
```
    
You tell me!

```
s/Handler/HandlerT
```

That works.

```Haskell
HelloSub.hs:19:26:
    Not in scope: data constructor `DataSourceInputR'
    Perhaps you meant `DataSourceInput' (imported from HelloSub.Data)
```
    
I already forgot why this was the case.

Oh right, the `mkYesodSubData` thing.

```Haskell
mkYesodSubData "HelloSub" [parseRoutes|
/ SubHomeR GET
/datasource DataSourceInputR POST
|]
```

```Haskell
HelloSub.hs:29:58: Not in scope: `show'
```

DAMMIT

Replace import `Prelude (IO,($))` with `ClassyPrelude`.

```Haskell
HelloSub.hs:25:25:
    Expecting two more arguments to `HandlerT Html'
    Expected a type, but `HandlerT Html' has kind `(* -> *) -> * -> *'
    In the type signature for `postDataSourceInputR':
      postDataSourceInputR :: HandlerT Html
```
      
Woo, a kind error, something I can probably fix!

currently: `postDataSourceInputR :: HandlerT Html`

I guess I'll want something like the example HelloSub thing: `getSubHomeR :: Yesod master => HandlerT HelloSub (HandlerT master IO) Html`

Let's do that.

```Haskell
HelloSub.hs:30:14:
    Couldn't match type `IO' with `HandlerT master IO'
    Expected type: HandlerT HelloSub (HandlerT master IO) Html
      Actual type: HandlerT HelloSub IO Html
    Relevant bindings include
      postDataSourceInputR :: HandlerT HelloSub (HandlerT master IO) Html
        (bound at HelloSub.hs:26:1)
    In the expression:
      defaultLayout
        (do { (asWidgetT GHC.Base.. toWidget)
                ((blaze-markup-0.7.0.2:Text.Blaze.Internal.preEscapedText
                  GHC.Base.. Data.Text.pack)
                   "<p>Invalid input, let's try again.</p>\n\
                   \<form method=\"post\" action=\"");
              (getUrlRenderParams
               >>=
                 (\ urender_antS
                    -> (asWidgetT GHC.Base.. toWidget)
                         (toHtml (\ u_antT -> urender_antS u_antT [] DataSourceInputR))));
              (asWidgetT GHC.Base.. toWidget)
                ((blaze-markup-0.7.0.2:Text.Blaze.Internal.preEscapedText
                  GHC.Base.. Data.Text.pack)
                   "\" enctype=\"");
              (asWidgetT GHC.Base.. toWidget) (toHtml enctype);
              (asWidgetT GHC.Base.. toWidget)
                ((blaze-markup-0.7.0.2:Text.Blaze.Internal.preEscapedText
                  GHC.Base.. Data.Text.pack)
                   "\">");
              .... })
    In a case alternative:
        _ -> defaultLayout
               (do { (asWidgetT GHC.Base.. toWidget)
                       ((blaze-markup-0.7.0.2:Text.Blaze.Internal.preEscapedText

                         GHC.Base.. Data.Text.pack)
                          "<p>Invalid input, let's try again.</p>\n\
                          \<form method=\"post\" action=\"");
                     (getUrlRenderParams
                      >>=
                        (\ urender_antS
                           -> (asWidgetT GHC.Base.. toWidget)
                                (toHtml (\ u_antT -> urender_antS u_antT [] DataSourceInputR))));
                     (asWidgetT GHC.Base.. toWidget)
                       ((blaze-markup-0.7.0.2:Text.Blaze.Internal.preEscapedText

                         GHC.Base.. Data.Text.pack)
                          "\" enctype=\"");
                     (asWidgetT GHC.Base.. toWidget) (toHtml enctype);
```

JESUS

```Haskell
 Couldn't match type `IO' with `HandlerT master IO'
 Expected type: HandlerT HelloSub (HandlerT master IO) Html
```
 
Oh ok
 
Let's fix that:

```Haskell
postDataSourceInputR :: HandlerT HelloSub IO Html
```

```Haskell
 HelloSub.hs:39:26:
    Couldn't match type `IO' with `HandlerT parent1 IO'
    Expected type: HandlerT HelloSub (HandlerT parent1 IO) Html
      Actual type: HandlerT HelloSub IO Html
    Relevant bindings include
      env6990_ansz :: Yesod.Core.Types.YesodSubRunnerEnv
                        HelloSub parent1 (HandlerT parent1 IO)
        (bound at HelloSub.hs:39:26)
      inner :: Yesod.Core.Types.YesodSubRunnerEnv
                 HelloSub parent1 (HandlerT parent1 IO)
               -> wai-3.0.2.3:Network.Wai.Internal.Request
               -> (wai-3.0.2.3:Network.Wai.Internal.Response
                   -> IO wai-3.0.2.3:Network.Wai.Internal.ResponseReceived)
               -> IO wai-3.0.2.3:Network.Wai.Internal.ResponseReceived
        (bound at HelloSub.hs:39:26)
    In the first argument of `yesod-core-1.4.8.3:Yesod.Core.Class.Dispatch.subHe
lper
                              GHC.Base.. (fmap toTypedContent)', namely
      `postDataSourceInputR'
    In the expression:
      (yesod-core-1.4.8.3:Yesod.Core.Class.Dispatch.subHelper
       GHC.Base.. (fmap toTypedContent))
        postDataSourceInputR
        env6990_ansz
        (Just DataSourceInputR)
        req6990_ansA
```

Damn, this is in Template Haskell.

I'll google for `YesodSubDispatch`.

[http://www.yesodweb.com/blog/2013/03/big-subsite-rewrite](http://www.yesodweb.com/blog/2013/03/big-subsite-rewrite)

Ooh, examples.

Ok, so I guess I'll go back to `Yesod master => HandlerT HelloSub (HandlerT master IO) Html`.

Ah, it's a type error. I'll replace `defaultLayout` with `lift $ defaultLayout` in the handlers.

```Haskell
HelloSub.hs:19:33:
    Could not deduce (master ~ HelloSub)
    from the context (Yesod master)
      bound by the type signature for
                 getSubHomeR :: Yesod master =>
                                HandlerT HelloSub (HandlerT master IO) Html
      at HelloSub.hs:16:16-74
      `master' is a rigid type variable bound by
               the type signature for
                 getSubHomeR :: Yesod master =>
                                HandlerT HelloSub (HandlerT master IO) Html
               at HelloSub.hs:16:16
    Expected type: WidgetT
                     master IO (Route HelloSub -> [(Text, Text)] -> Text)
      Actual type: WidgetT
                     master
                     IO
                     (Route (HandlerSite (WidgetT master IO)) -> [(Text, Text)] -> Text)
    Relevant bindings include
      getSubHomeR :: HandlerT HelloSub (HandlerT master IO) Html
        (bound at HelloSub.hs:17:1)
    In the first argument of `(>>=)', namely `getUrlRenderParams'
    In a stmt of a 'do' block:
      (getUrlRenderParams
       >>=
         (\ urender_aneD
            -> (asWidgetT GHC.Base.. toWidget)
                 (toHtml (\ u_aneE -> urender_aneD u_aneE [] DataSourceInputR)))
)

HelloSub.hs:31:22:
    Could not deduce (master ~ HelloSub)
    from the context (Yesod master)
      bound by the type signature for
                 postDataSourceInputR :: Yesod master =>
                                         HandlerT HelloSub (HandlerT master IO) Html
      at HelloSub.hs:25:25-83
      `master' is a rigid type variable bound by
               the type signature for
                 postDataSourceInputR :: Yesod master =>
                                         HandlerT HelloSub (HandlerT master IO) Html
               at HelloSub.hs:25:25
    Expected type: WidgetT
                     master IO (Route HelloSub -> [(Text, Text)] -> Text)
      Actual type: WidgetT
                     master
                     IO
                     (Route (HandlerSite (WidgetT master IO)) -> [(Text, Text)] -> Text)
    Relevant bindings include
      postDataSourceInputR :: HandlerT HelloSub (HandlerT master IO) Html
        (bound at HelloSub.hs:26:1)
    In the first argument of `(>>=)', namely `getUrlRenderParams'
    In a stmt of a 'do' block:
      (getUrlRenderParams
       >>=
         (\ urender_anjY
            -> (asWidgetT GHC.Base.. toWidget)
                 (toHtml (\ u_anjZ -> urender_anjY u_anjZ [] DataSourceInputR)))
```

Man, this just gets better and better, doesn't it. TH is being a pain in my *ASS*.

Snoyman has a wiki example in a Github, let's check that out : [https://github.com/yesodweb/yesod/blob/new-subsite/demo/WikiRoutes.hs](https://github.com/yesodweb/yesod/blob/new-subsite/demo/WikiRoutes.hs)

I have no clue what the problem is.

Ok, screw it, let's just go back to the basics:

```Haskell
postDataSourceInputR :: Yesod master => HandlerT HelloSub (HandlerT master IO) Html
postDataSourceInputR = do lift $ defaultLayout [whamlet|test|]

-- And we'll spell out the handler type signature.
getSubHomeR :: Yesod master => HandlerT HelloSub (HandlerT master IO) Html
getSubHomeR = do lift $ defaultLayout [whamlet|test|]
```

Ok, that compiles.

```Haskell
getSubHomeR :: Yesod master => HandlerT HelloSub (HandlerT master IO) Html
getSubHomeR = do 
  (widget, enctype) <- generateFormPost simpleSourceForm
  lift $ defaultLayout [whamlet|
    <form method=post action=@{DataSourceInputR} enctype=#{enctype}>
    ^{widget}
    <button>Submit
  |]
  
simpleSourceForm = renderDivs $ DataSourceInput
  <$> areq intField "Start" Nothing
  <*> areq intField "End" Nothing
```

That doesn't: 

```Haskell
HelloSub.hs:25:33:
    Could not deduce (master ~ HelloSub)
    from the context (Yesod master)
      bound by the type signature for
                 getSubHomeR :: Yesod master =>
                                HandlerT HelloSub (HandlerT master IO) Html
      at HelloSub.hs:22:16-74
      `master' is a rigid type variable bound by
               the type signature for
                 getSubHomeR :: Yesod master =>
                                HandlerT HelloSub (HandlerT master IO) Html
               at HelloSub.hs:22:16
    Expected type: WidgetT
                     master IO (Route HelloSub -> [(Text, Text)] -> Text)
      Actual type: WidgetT
                     master
                     IO
                     (Route (HandlerSite (WidgetT master IO)) -> [(Text, Text)] -> Text)
    Relevant bindings include
      getSubHomeR :: HandlerT HelloSub (HandlerT master IO) Html
        (bound at HelloSub.hs:23:1)
    In the first argument of `(>>=)', namely `getUrlRenderParams'
    In a stmt of a 'do' block:
      (getUrlRenderParams
       >>=
         (\ urender_anFs
            -> (asWidgetT GHC.Base.. toWidget)
                 (toHtml (\ u_anFt -> urender_anFs u_anFt [] DataSourceInputR))))
```

Stupid equality constraints.

**MORE GOOGLING**

[https://github.com/yesodweb/yesod/wiki/Intra-subsite-links](https://github.com/yesodweb/yesod/wiki/Intra-subsite-links)

AHA

```Haskell
getSubHomeR :: Yesod master => HandlerT HelloSub (HandlerT master IO) Html
getSubHomeR = do 
  (widget, enctype) <- generateFormPost simpleSourceForm
  toMaster <- getRouteToParent
  lift $ defaultLayout [whamlet|
    <form method=post action=@{toMaster DataSourceInputR} enctype=#{enctype}>
    ^{widget}
    <button>Submit
  |]
```
  
Dammit:

```Haskell
HelloSub.hs:26:33:
    Could not deduce (master ~ HelloSub)
    from the context (Yesod master)
      bound by the type signature for
                 getSubHomeR :: Yesod master =>
                                HandlerT HelloSub (HandlerT master IO) Html
      at HelloSub.hs:22:16-74
      `master' is a rigid type variable bound by
               the type signature for
                 getSubHomeR :: Yesod master =>
                                HandlerT HelloSub (HandlerT master IO) Html
               at HelloSub.hs:22:16
    Relevant bindings include
      toMaster :: Route HelloSub -> Route master
        (bound at HelloSub.hs:25:3)
      getSubHomeR :: HandlerT HelloSub (HandlerT master IO) Html
        (bound at HelloSub.hs:23:1)
    In the second argument of `(GHC.Base..)', namely `toWidget'
    In the expression: asWidgetT GHC.Base.. toWidget
    In a stmt of a 'do' block: (asWidgetT GHC.Base.. toWidget) widget
```

Reduce example again:

```Haskell
getSubHomeR :: Yesod master => HandlerT HelloSub (HandlerT master IO) Html
getSubHomeR = do
 toMaster <- getRouteToParent
 lift $ defaultLayout [whamlet|<a href=@{toMaster DataSourceInputR}> |]
```
  
YES THAT WORKS

```Haskell
Internal Server Error
Application.hs:60:36-43: Missing field in record construction Foundation.getHelloSub
```

NO IT DOESNT

This probably has to do with the warning message it keeps puking out:

```Haskell
Application.hs:60:36: Warning:
    Fields of `App' not initialised: getHelloSub
    In the expression: App {..}
    In an equation for `mkFoundation':
        mkFoundation appConnPool = App {..}
    In the expression:
      do { appHttpManager <- newManager;
           appLogger <- newStdoutLoggerSet defaultBufSize >>= makeYesodLogger;
           appStatic <- (if appMutableStatic appSettings then
                             staticDevel
                         else
                             static)
                          (appStaticDir appSettings);
           let mkFoundation appConnPool = ...
               tempFoundation
                 = mkFoundation $ error "connPool forced in tempFoundation"
               ....;
           .... }
```
           
How do I initialize `getHelloSub`? Is that even a solution?

Screw it, I'll contact Snoyberg.

### THE NEXT DAY

Snoyberg contacted.

> the problem is that you haven't populated the `getHelloSub` field in the `makeFoundation` function.

Ok.

```Haskell
makeFoundation :: AppSettings -> IO App
makeFoundation appSettings = do
    -- Some basic initializations: HTTP connection manager, logger, and static
    -- subsite.
    appHttpManager <- newManager
    appLogger <- newStdoutLoggerSet defaultBufSize >>= makeYesodLogger
    appStatic <-
        (if appMutableStatic appSettings then staticDevel else static)
        (appStaticDir appSettings)
        
    getHelloSub <- return HelloSub

    -- We need a log function to create a connection pool. We need a connection
    -- pool to create our foundation. And we need our foundation to get a
    -- logging function. To get out of this loop, we initially create a
    -- temporary foundation without a real connection pool, get a log function
    -- from there, and then create the real foundation.
    let mkFoundation appConnPool = App {..}
        tempFoundation = mkFoundation $ error "connPool forced in tempFoundation"
        logFunc = messageLoggerSource tempFoundation appLogger

    -- Create the database connection pool
    pool <- flip runLoggingT logFunc $ createSqlitePool
        (sqlDatabase $ appDatabaseConf appSettings)
        (sqlPoolSize $ appDatabaseConf appSettings)

    -- Perform database migration using our application's logging settings.
    runLoggingT (runSqlPool (runMigration migrateAll) pool) logFunc
    
    -- Return the foundation
    return $ mkFoundation pool
```
    
There we go.

It works now. Nice. Let's add the form stuff now.

```Haskell
getDataSourceInputR :: HelloSubHandler Html
getDataSourceInputR = do 
  toMaster <- getRouteToParent
  (widget, enctype) <- generateFormPost simpleSourceForm
  lift $ defaultLayout
            [whamlet|
                <p>
                    The widget generated contains only the contents
                    of the form, not the form tag itself. So...
                <form method=post action=@{toMaster DataSourceInputR} enctype=#{enctype}>
                    ^{widget}
                    <p>It also doesn't include the submit button.
                    <button>Submit
            |]
```

Added the form stuff, have the equality constraint problem again:

```Haskell
HelloSub.hs:27:22:
    Could not deduce (master ~ HelloSub)
    from the context (YesodHelloSub master)
      bound by the type signature for
                 getDataSourceInputR :: YesodHelloSub master =>
                                        HandlerT HelloSub (HandlerT master IO) Html
      at HelloSub.hs:22:24-43
      `master' is a rigid type variable bound by
               the type signature for
                 getDataSourceInputR :: YesodHelloSub master =>
                                        HandlerT HelloSub (HandlerT master IO) Html
               at HelloSub.hs:19:33
    Relevant bindings include
      toMaster :: Route HelloSub -> Route master
        (bound at HelloSub.hs:24:3)
      getDataSourceInputR :: HandlerT HelloSub (HandlerT master IO) Html
        (bound at HelloSub.hs:23:1)
    In the second argument of `(GHC.Base..)', namely `toWidget'
    In the expression: asWidgetT GHC.Base.. toWidget
    In a stmt of a 'do' block: (asWidgetT GHC.Base.. toWidget) widget
```

Let's look at Snoyman's wiki example again: [https://github.com/yesodweb/yesod/blob/new-subsite/demo](https://github.com/yesodweb/yesod/blob/new-subsite/demo)

Let's add the type alias, it's nice.

```Haskell
class (RenderMessage master FormMessage, Yesod master) => YesodHelloSub master where
    dummyThing :: HandlerT master IO Bool
    dummyThing = return True
    
type HelloSubHandler a = forall master. YesodHelloSub master 
                         => HandlerT HelloSub (HandlerT master IO) a
                         
instance YesodHelloSub master => YesodSubDispatch HelloSub (HandlerT master IO) where
    yesodSubDispatch = $(mkYesodSubDispatch resourcesHelloSub)
```
   
```Haskell   
-- In Foundation.hs
instance YesodHelloSub App
```

(might be fine to put that elsewhere? Who knows.)

```Haskell
getSubHomeR :: HelloSubHandler Html
```

Ooh, squeaky clean. Didn't fix the equality constraint type error, though.

It's got something to do with the widget, apparently? Screw it, I'll just lift it.

```Haskell
(widget, enctype) <- lift $ generateFormPost simpleSourceForm
```

WOO THAT WORKED

Ok, let's introduce the POST part.

```Haskell
postDataSourceInputR :: HelloSubHandler Html
postDataSourceInputR = do
    toMaster <- getRouteToParent
    ((result, widget), enctype) <- runFormPost simpleSourceForm
    case result of
        FormSuccess datasource -> lift $ defaultLayout [whamlet|<p>#{show datasource}|]
        _ -> lift $ defaultLayout
            [whamlet|<a href=@{toMaster SubHomeR}> |]
```


```Haskell 
            HelloSub.hs:52:36:
    Could not deduce (RenderMessage HelloSub FormMessage)
      arising from a use of `runFormPost'
    from the context (YesodHelloSub master)
      bound by the type signature for
                 postDataSourceInputR :: YesodHelloSub master =>
                                         HandlerT HelloSub (HandlerT master IO) Html
      at HelloSub.hs:49:25-44
    In a stmt of a 'do' block:
      ((result, widget), enctype) <- runFormPost simpleSourceForm
    In the expression:
      do { toMaster <- getRouteToParent;
           ((result, widget), enctype) <- runFormPost simpleSourceForm;
           case result of {
             FormSuccess datasource
               -> lift
                  $ defaultLayout
                      (do { (asWidgetT GHC.Base.. toWidget)
                              ((blaze-markup-0.7.0.2:Text.Blaze.Internal.preEscapedText
                                GHC.Base.. Data.Text.pack)
                                 "<p>");
                            .... })
             _ -> lift
                  $ defaultLayout
                      (do { (asWidgetT GHC.Base.. toWidget)
                              ((blaze-markup-0.7.0.2:Text.Blaze.Internal.preEscapedText
                                GHC.Base.. Data.Text.pack)
                                 "<a href=\"");
                            .... }) } }
    In an equation for `postDataSourceInputR':
        postDataSourceInputR
          = do { toMaster <- getRouteToParent;
                 ((result, widget), enctype) <- runFormPost simpleSourceForm;
                 case result of {
                   FormSuccess datasource -> lift $ defaultLayout (do { ... })
                   _ -> lift $ defaultLayout (do { ... }) } }
```

Ooh boy.

Hmm: `RenderMessage HelloSub FormMessage`

Sounds like we need to lift it to the master implementation again.

```Haskell
((result, widget), enctype) <- lift $ runFormPost simpleSourceForm
```

Works! (finally!)

## CABAL HELL

Let's make some repos for each package and see how horrible it all plays with cabal sandboxes.

Yesod doesn't like to play with cabal sandboxes AT ALL. How do I pass sandbox arguments to `yesod devel`?

```
yesod devel -e "--sandbox-config-file=../sandbox/cabal.sandbox.config"
Yesod devel server. Press ENTER to quit
cabal: unrecognized 'configure' option
`--sandbox-config-file=../sandbox/cabal.sandbox.config'
```

Nope.

```
yesod devel -e "--sandbox-config-file=\"../sandbox/cabal.sandbox.config\""
Yesod devel server. Press ENTER to quit
cabal: unrecognized 'configure' option
`--sandbox-config-file="../sandbox/cabal.sandbox.config"'
cabal: unrecognized 'configure' option
`--sandbox-config-file="../sandbox/cabal.sandbox.config"'
```

Nope.

```
cabal help sandbox
Usage: cabal sandbox init
   or: cabal sandbox delete
   or: cabal sandbox add-source  [PATHS]

   or: cabal sandbox hc-pkg      -- [ARGS]
   or: cabal sandbox list-sources

Flags for sandbox:
 -h --help        Show this help text
 -v --verbose[=n] Control verbosity (n is 0--3, default verbosity level is 1)
    --snapshot    Take a snapshot instead of creating a link (only applies to
                  'add-source')
    --sandbox=DIR Sandbox location (default: './.cabal-sandbox').
```

Perhaps...

```
>yesod devel -e "--sandbox=../sandbox/.cabal-sandbox"
Yesod devel server. Press ENTER to quit
cabal: unrecognized 'configure' option `--sandbox=../sandbox/.cabal-sandbox'
```

Nope.

```
yesod devel -e "sandbox --sandbox=../sandbox/.cabal-sandbox"
Yesod devel server. Press ENTER to quit
Resolving dependencies...
cabal: Unrecognised flags: sandbox --sandbox=../sandbox/.cabal-sandbox
```

Nope.

Google?

[http://blog.pangyanhan.com/haskell/2013-11-31-using-cabal-sandbox-libraries.html](http://blog.pangyanhan.com/haskell/2013-11-31-using-cabal-sandbox-libraries.html)

Nope.

## Conclusion

There is still a lot to be done to make Yesod development productive without boatloads of [yak shaving](https://en.wiktionary.org/wiki/yak_shaving). It was very hard to find examples of the functionality I wanted, or even a usable (and working with the latest stable release!) list of Yesod subsites.