# Chapter Four: Lifting

Something that we're going to sidestep almost entirely is a concept called [monad transformers](https://en.wikibooks.org/wiki/Haskell/Monad_transformers). We rarely use them _directly_ in practice, even though they're all through our dependencies, or isolated to type class instances or more semantic helper functions. Remember that the purpose of this guide is to give a pragmatic up-and-running style introduction to being productive in Haskell. No one asks how the plumbing _inside_ TypeScript promises work, but it's easy to follow along with the high-level syntax and concepts.

## Levels

Your application will sometimes consist of several nested data types. For example:

```haskell
Either Error (Maybe [a])
```

You probably actually care about the `a` values, but need to work with type class instances for `Either`, `Maybe`, and `List`. We can think of these as "levels" of wrappers:

```text
+--------Either--------+
|          +--Maybe--+ |
|          |         | |
| Error    |  [ a ]  | |           
|          |         | |
|          +---------+ |
+----------------------+
```

### Mixing Levels

Clearly there's a lot of layers that you can think about. If all you care about is the outermost `Either`, then it's pretty straightforward. To access the inner `Maybe`, you need to find a way to get into `Either` structure, do what you need at that level, and get back out again. You can have all of your functions wrap and unwrap the various layers, but that composes badly and leads to a lot of boilerplate.

The solution is to `lift` a function through the layers. Take something that works on one layer, and make it work on another. We can do this manually, but for all of the common data types, there are type classes to help us do this in a straightforward way. By far the most common one is `liftIO`.

{% hint style="info" %}
Everything in a Haskell application occurs inside the `IO` type, which is where the horrible, unpredictable, tainted imperative world touches your beautiful, pristine functional clockwork.

Haskell doesn't expose the `IO` constructor \(i.e. it's not exported, something you can do in your modules, too\), so we want to avoid touching `IO` directly as much as possible because it's impossible to get fully out of it. At the end of the day your function will get used in a `IO` context \(even if you're several layers down\). Most of the Haskell style is creating clean, pure, high-level DSLs, and calling _those_ inside `IO`.
{% endhint %}

## MonadIO

`IO` actions occur frequently. Accessing CLI input, talking to the database, and making a network request are all things that you'll want to do on a regular basis. If the monad that you're working in also has a `MonadIO` instance, you get access to `liftIO`. Fission uses `MonadRIO` very frequently, which inherits from `MonadIO`. The best way to read `liftIO` is as "turn an `IO` function into one that happens in my current wrapper."

{% hint style="info" %}
This example uses an `IO`-enabled wrapper: `m`. It's done this way to make it match easily with any other `MonadIO` instance, rather than using IO directly.
{% endhint %}

{% code-tabs %}
{% code-tabs-item title="Fission.Timestamp" %}
```haskell
addM :: MonadIO m => Unstamped r -> m r
addM record = do
  now <- liftIO getCurrentTime
  return $ add now record
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### Specifically RIO

Let's look at something that uses a concrete wrapping type, in this case `RIO`.

{% hint style="info" %}
This example uses `RIO` directly because it's used while setting up the database pool for our ambient config on app startup, and it's more convenient phrased this way for this scenario.
{% endhint %}

{% code-tabs %}
{% code-tabs-item title="Fission.Storage.SQLite" %}
```haskell
connPool :: HasLogFunc cfg => DB.Path -> RIO cfg DB.Pool
connPool (DB.Path {getPath = path}) = do
  logDebug $ "Establishing DB pool for " <> displayShow path

  rawPool <- liftIO $ createPool (sqliteOpen path) seldaClose 4 2 10
  logDebug $ "DB pool stats: " <> displayShow rawPool

  return $ DB.Pool rawPool
```
{% endcode-tabs-item %}
{% endcode-tabs %}

The function `createPool :: IO (Pool SeldaConnection)` needs to be brought into `RIO`. Remember that `IO` doesn't export a constructor for us to explicitly destructure and repackage! We need a way to turn `IO a` into `RIO cfg a`.

{% hint style="success" %}
`RIO` stands for "`Reader` + `IO`." Clearly this has a `MonadIO` instance! 
{% endhint %}

 We can always write this function manually, but by far the easiest is the `liftIO` instance for `RIO`. The upshot is that it makes `IO` actions compatible with the rest of your `RIO` function.

