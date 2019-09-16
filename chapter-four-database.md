# Chapter Six: Database DSL

## The Legend of Selda

Haskell has many database libraries, and there's a recent trend in leveraging the type system to giving increasingly safe database access. Despite the goofy name, [Selda](https://selda.link/) is a very robust database DSL, and works equally well for PostgreSQL or SQLite.

{% hint style="info" %}
The website for Selda is literally `https://selda.link`⚔️🛡️🤣
{% endhint %}

{% hint style="info" %}
We may later switch to [Beam](https://tathougies.github.io/beam/), which is more feature rich, has generally faster compilation, and has a clearer tutorial. We switched to Selda due to a bug in a dependency, which has since been fixed.
{% endhint %}

The flip side of having so much power in a database library is that the types can get pretty out of hand sometimes. This is a trade-off that we make in a number of places throughout the application, and is highly beneficial 98% of the time. Selda uses a lot of constraint aliases to merge multiple constraints into one. GHC or Intero will give you hints with the underlying constraints. We'll touch of a few things to keep in mind below.

## MonadSelda

{% code-tabs %}
{% code-tabs-item title="Fission.Storage.Types" %}
```haskell
type SeldaPool = Database.Pool (SeldaConnection SQLite)

newtype Pool = Pool { getPool :: SeldaPool }
  deriving Show
```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% code-tabs %}
{% code-tabs-item title="Fission.Internal.Orphanage" %}
```haskell
instance Has DB.Pool cfg => MonadSelda (RIO cfg) where
 type Backend (RIO cfg) = SQLite

 withConnection action = do
    DB.Pool pool <- Config.get
    withResource pool action
```
{% endcode-tabs-item %}
{% endcode-tabs %}

