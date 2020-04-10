---
description: We focus on storytelling in our code
---

# Appendix III: Fission Style Guide

## Type Signatures

Type signatures are required

### Single-Line

{% tabs %}
{% tab title="Right" %}
```haskell
id :: a -> a
id x = x
```
{% endtab %}

{% tab title="Wrong" %}
```haskell
id 
  :: a 
  -> a
id x = x
```
{% endtab %}
{% endtabs %}

### Multi-Line

{% tabs %}
{% tab title="Right" %}
```haskell
rawList ::
  ( MonadRIO          cfg m
  , Has IPFS.BinPath  cfg
  , Has IPFS.Timeout  cfg
  , HasProcessContext cfg
  , HasLogFunc        cfg
  )
  => m (ExitCode, Lazy.ByteString, Lazy.ByteString)
rawList = IPFSProc.run' ["bootstrap", "list"]
```
{% endtab %}

{% tab title="Wrong" %}
```haskell
rawList :: MonadRIO          cfg m
        => Has IPFS.BinPath  cfg
        => Has IPFS.Timeout  cfg
        => HasProcessContext cfg
        => HasLogFunc        cfg
        => m (ExitCode, Lazy.ByteString, Lazy.ByteString)
rawList = IPFSProc.run' ["bootstrap", "list"]
```
{% endtab %}
{% endtabs %}

Why is it like this? A major reason is that the syntax highlighter in VS Code breaks with the `::` on the next line. We've also reverted to using tuple-style constraints, because in practive they making distinguishing between the constraints and argument types.

## Pipes

Indent multi-line pipelines by 2 from the first item. This makes it consistent in `do`-notation and normal notations.

{% tabs %}
{% tab title="Right" %}
```haskell
app
  |> condDebug 
  |> CORS.middleware
  |> runner settings
  |> liftIO
```
{% endtab %}

{% tab title="Wrong" %}
```haskell
app
|> condDebug 
|> CORS.middleware
|> runner settings
|> liftIO
```
{% endtab %}
{% endtabs %}

As much as possible, pipe in one direction

{% tabs %}
{% tab title="Right" %}
```haskell
Hku.Password <| encodeUtf8 <| Hku.password <| Hku.api <| manifest
```
{% endtab %}

{% tab title="Wrong" %}
```haskell
Hku.Password <| encodeUtf8 (manifest |> Hku.api |> Hku.password)
```
{% endtab %}
{% endtabs %}

Break long pipelines into multiple lines where possible

{% tabs %}
{% tab title="Right" %}
```haskell
app
  |> condDebug
  |> CORS.middleware
  |> runner settings
  |> liftIO

```
{% endtab %}

{% tab title="Wrong" %}
```haskell
app |> condDebug |> CORS.middleware |> runner settings |> liftIO
```
{% endtab %}
{% endtabs %}

Do not mix pipes with the bind operator

{% tabs %}
{% tab title="Right" %}
```haskell
app <- Web.app

app
  |> condDebug 
  |> CORS.middleware
  |> runner settings
  |> liftIO
```
{% endtab %}

{% tab title="Wrong" %}
```haskell
Web.app
  >>= condDebug
  |> CORS.middleware
  |> runner settings
  |> liftIO
```
{% endtab %}
{% endtabs %}

## Avoid the Point-Free Style

Refrain from defining functions in point-free style:

{% tabs %}
{% tab title="Right" %}
```haskell
docs :: Web.Host -> Swagger
docs host' =
  host'
    |> app (Proxy @Web.API)
    |> dns
    |> ipfs
    |> heroku
    |> ping
    |> user
```
{% endtab %}

{% tab title="Wrong" %}
```haskell
docs :: Web.Host -> Swagger
docs = app (Proxy @Web.API)
     . dns
     . ipfs
     . heroku
     . ping
     . user

```
{% endtab %}
{% endtabs %}

However, they often do read well in a pipeline:

{% tabs %}
{% tab title="Right" %}
```haskell
alphaNum :: MonadIO m => Natural -> m Text
alphaNum len =
  len
    |> bsRandomLength
    |> fmap (decodeUtf8Lenient . BS.filter isAsciiAlphaNum)
```
{% endtab %}

{% tab title="Acceptable" %}
```haskell
alphaNum :: MonadIO m => Natural -> m Text
alphaNum len =
  len
    |> bsRandomLength
    |> fmap (\str ->
         str 
           |> BS.filter isAsciiAlphaNum
           |> decodeUtf8Lenient)
```
{% endtab %}
{% endtabs %}

## `MonadIO`

Whenever possible, generalize `IO` functions to `MonadIO`. This prevents you from needing to generalize the function at the call site, while still able to use it in `IO` contexts.

{% tabs %}
{% tab title="Right" %}
```haskell
fromHandler :: MonadIO m => Handler a -> m a
fromHandler handler =
  liftIO <| runHandler handler >>= \case
    Right inner     -> pure inner
    Left servantErr -> throwM servantErr
    
-- In use...

server appHost = Web.Swagger.server fromHandler appHost
            :<|> IPFS.server
            :<|> const Heroku.server
            :<|> User.server
            :<|> pure Ping.pong
            :<|> DNS.server
```
{% endtab %}

{% tab title="Wrong" %}
```haskell
fromHandler :: Handler a -> IO a
fromHandler handler =
  runHandler handler >>= \case
    Right inner     -> pure inner
    Left servantErr -> throwM servantErr

-- In use...

server appHost = Web.Swagger.server (liftIO . fromHandler) appHost
            :<|> IPFS.server
            :<|> const Heroku.server
            :<|> User.server
            :<|> pure Ping.pong
            :<|> DNS.server
```
{% endtab %}
{% endtabs %}

