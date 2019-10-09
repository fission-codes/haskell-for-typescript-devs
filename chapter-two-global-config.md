# Chapter Three: Ambient Config

## Why Ambient?

Often things like configuration variables read in from the command line or a file, shared database pools, or functions that we want to use in our application at runtime but swapped out for development or at test-time \(i.e. [dependency injection](https://nehalist.io/dependency-injection-in-typescript/)\).

Haskell is a "pure" language where everything is explicit. As a naive approach, this can sometimes lead to boilerplate function arguments for threading a value around your application. This also means many changes when something breaks. To get around this, we sometimes want to have something available "ambiently" in an application, as if it were a _global constant_, and have all that threading done for us by helper functions.

This is a standard pattern known as a `Reader`. It's so common that the Haskell community has started rapidly embracing it as _the_ wrapper for applications. You can have global and local `Reader`s, but we'll be focusing here on the global one for our application.

## App Config

The first part of this is to create a data structure that will contain our global context. Many people call this `Env`, but that can conflict with standard terminology for environment variables, which environment an application is running in \(test, development, staging, production\), and so on. We have opted to call this `Config`.

{% code-tabs %}
{% code-tabs-item title="Fission.Config.Types" %}
```haskell
data Config = Config 
  { _logFunc :: !LogFunc
  , _host    :: !Web.Host
  , _dbPath  :: !DB.Path
  , _dbPool  :: !DB.Pool
  -- and so on
  }
  
  makeLenses ''Config
```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% hint style="info" %}
We may move to [SuperRecord](https://www.athiemann.net/2017/07/02/superrecord.html) in the future, for _even less_ boilerplate plus some nifty additional super powers [🦸](https://emojipedia.org/superhero/)
{% endhint %}

{% hint style="success" %}
Keeping your Config as flat as possible is generally a good idea. While many people have an intuition that nesting things by concept \(e.g. database\) is a good idea, it's generally more trouble than it's worth in practice.
{% endhint %}

## Explicit Constraints

While we have all of this threading done for us, we still want to know which part of the config is required by which part of the application. This approach has a few upsides:

1. Easy to read labels help you keep track of what a function can access
2. Functions don't depend on specific concrete `Config`s, just fields
3. The compiler can help you refactor if change the `Config`
4. Constraints bubble up to callers, so dependencies can't hide

Here's an example that retrieves the web host name, and combines it with a ncie message that is passed in as an argument:

```haskell
hostMsg :: MonadRIO     cfg m
        => Has Web.Host cfg 
        => Text
        -> m Text
hostMsg greeting = do
  Web.Host hostname <- Config.get
  return $ greeting <> ", the app is live at " <> hostname
```

`Config.get` pulls out a value from the `Config`. It knows which value you want because of the expected constructor \(`Web.Host`\) on the left side of the `<-`.

{% hint style="danger" %}
`MonadRIO` is a constraint defined in our application. It's an alias for the very common scenario in this style of wanting both `MonadIO m` and `MonadReader cfg m`. 
{% endhint %}

### Logging

Our prelude \([`RIO`](https://www.fpcomplete.com/blog/2017/07/the-rio-monad)\) depends on having functions available ambiently in this way. One common case is with logging, which needs a `LogFunc` ready for use. 

```haskell
logHost :: MonadRIO     cfg m
        => Has Web.Host cfg
        => HasLogFunc   cfg
        -> m ()
logHost = logInfo $ "Host name is " <> display hostname
```

{% hint style="success" %}
Unlike the first example, the constraint has no space after the `Has`. This is because we're using the [Has library](http://hackage.haskell.org/package/data-has) to clean up some of the boilerplate associated with creating so many constraints.
{% endhint %}

## Adding a New Field to the Config

You are likely to want to add custom fields to the `Config` record. The first step is to ensure that the type is unique to the application, wrapping common types in `newtype`:

{% code-tabs %}
{% code-tabs-item title="Fission.Web.Types" %}
```haskell
newtype Port = Port { getPort :: Int }
  deriving Show
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Next, add it to the `Config` itself:

{% code-tabs %}
{% code-tabs-item title="Fission.Config.Types" %}
```haskell
data Config = Config 
  { _logFunc :: !LogFunc
  , _host    :: !Web.Host
  , _port    :: !Web.Port -- THIS LINE
  , _dbPath  :: !DB.Path
  , _dbPool  :: !DB.Pool
  }
  
makeLenses ''Config
```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% hint style="success" %}
Because of the `makeLenses` declaration, you automagically get a lens \(superpowered accessor\) for the new field called `port`

`Unsure of what lenses are? check out our`[`Common Lens Legend`](appendix-t.md)\`\`
{% endhint %}

Finally, write a [`Has` instance](https://www.stackage.org/haddock/lts-14.5/data-has-0.3.0.0/Data-Has.html):

```haskell
instance Has Web.Port Config where
  hasLens = port
```

That's it! It's available everywhere in the application now!

```haskell
logPort :: MonadRIO     cfg m
        => Has Web.Port cfg
        => HasLogFunc   cfg
        => m ()
logPort = do
  Web.Port port <- Config.get
  logInfo $ displayShow port
```

