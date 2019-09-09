---
description: "Type classes are an indefensible part of how we write Haskell. A lot of the rest of this material won't make a lot of sense without them\U0001F92A"
---

# Chapter Two: Type Classes

## What's in a Name?

Calling them "type classes" is both a stroke of genius and a terrible mistake. There are absolutely strong parallels with classes from object-oriented programming. They also mostly work the exact opposite way that you'd expect coming from OOP.

## Abstraction

Type classes let you define a common interface across many types. The concrete function that gets called gets determined by how you define an `instance` of that `class`. 

For example, let's say that you have these functions:

```haskell
add :: Int -> Int -> Int
add a b = a + b

(++) :: [a] -> [a] -> [a]
[]       ++ bs = bs
(a : as) ++ bs = a : as ++ bs

append :: Text -> Text -> Text
append a b = -- Long complex function

-- Function composition
(.) :: (b -> c) -> (a -> b) -> (a -> c)
f . g = \x -> f (g x)
```

In a sense, all of these do the same thing: they combine values of a common type together \(i.e. they all have the form `a -> a -> a`\) While we may expect them to behave in some common ways, the exact meaning of that depends on the specific type. Why not write functions that could work on any of these types with respect to this combining behaviour?

We can define a common interface for this "combinable" behaviour!

```haskell
class Combinable a where
  combine :: a -> a -> a
```

This is called a semigroup in maths, so the version in the library is actually called `Semigroup`. Don't let the math-y name scare you! It's actually very straightforward for a "combinable" thing.

```haskell
class Semigroup a where
  (<>) :: a -> a -> a
```

Let's write some instances!

```haskell
instance Semigroup Int where
  a <> b = a + b
  
instance Semigroup [a] where
  as <> bs = as ++ bs
  
instance Semigroup Text where
  a <> b = append a b
  
intance Semigroup (-> a) where
  f <> g = f . g
```

Finally, let's use it in a function. We place "class constraints" before the arguments in the type signature, separated by a `=>`.

```haskell
triple :: Semigroup a => a -> a
triple x = x <> x <> x

triple 10
-- 30

triple [1, 2, 3]
-- [1, 2, 3, 1, 2, 3, 1, 2, 3]

triple "hi"
-- "hihihi"

(triple (*2)) 5
-- 40
```

Libraries tend to give you a lot of "functions for free" to go with your instance. It's typically worth it to define an instance for a new data type.

## Recursive Abstraction

#### It's Roughly Like Inheritance™

![](.gitbook/assets/unlimitedpower-funny-gifs.gif)

Unlike OO where inheritance happens on concrete data, Haskell's "inheritance" on interfaces is more like saying "assuming that you have an `x`, then you can also do `y`!" It lets you extend abstract interfaces with more functions. This usually makes for a smaller number of types that have the new interface, because they need the old type class, plus an instance for the new one, which is not always even possible!

Let's extend that Semigroup to have an element that when used with `<>` is a no op. We'll call this `mempty`, because of how it works on lists \(\[\] ++ xs = xs\) and because the name `id` was already taken.

```haskell
class Semigroup a => Monoid a where
  mempty :: a
```

By definition, a Monoid is assumed to have a Semigroup instance kicking around. It has both `mempty` and `<>` automatically available. Let's look at a few of these:

```haskell
intance Monoid Int where
  mempty = 0

instance Monoid [a] where
  mempty = []
  
instance Monoid Text where
  mempty = ""
  
instance Monoid (-> a) where
  mempty = id
```

## Functor Hierarchy

The most widely-used type class heirarchy is the `Functor` tree, which includes the infamour `Monad` class. These also have special rules that you need to obey when writing instances for, but you'll mostly be using someone else's so don't worry about them too much for now.

There's also highly-recommended visual guide for the following material at [Functors, Applicatives, And Monads In Pictures](http://adit.io/posts/2013-04-17-functors,_applicatives,_and_monads_in_pictures.html).

### Functor

You may not know it, but you're already familair with functors!

```haskell
class Functor where
  fmap :: (a -> b) -> f a -> f b
```

You'll see a lot of `fmap`'s operator variants `<$>` and `<&>` floating around

```haskell
fmap show [1,2,3] -- ["1", "2", "3"]
show <$> [1,2,3]  -- ["1", "2", "3"]
```

The pun here is that it's the same as `$` \(application\) but working on something that's been put into a container \(practically any sum or product type\), hence the visual pun of `<$>`. Same goes for `&` \(pipe\) and `<&>`.

```haskell
show  $   1        -- "1"
show <$> [1, 2, 3] -- ["1", "2", "3"]

       1   &  show -- "1"
[1, 2, 3] <&> show -- ["1", "2", "3"]
```

### Applicative Functor

`Applicative`s add a lot of power to `Functors` by defining two functions:

1. `pure` \(also known as the somewhat confusingly named `return`\) takes a plain value and wraps it in a container.
2. `<*>` does function application when both functions and arguments are wrapped

```haskell
class Functor f => Applicative f where
    pure  :: a -> f a
    (<*>) :: f (a -> b) -> f a -> f b
```

{% hint style="info" %}
The `<*>` is often referred to as the [Tie Fighter](https://starwars.fandom.com/wiki/TIE/LN_starfighter) operator 🤣 It's also called `ap`, short for "applicative apply".
{% endhint %}

For example, on `Maybe`:

```haskell
instance Applicative (Maybe a) where
  pure x = Just x

  Just f <*> Just x = Just $ f x
  _ <*> _ = Nothing

pure 4    :: Just Int  -- Just 4
pure "hi" :: Just Text -- Just "hi"

Just (*10) <*> Just 6  -- Just 60
Nothing    <*> Just 6  -- Nothing
Just (*10) <*> Nothing -- Nothing
```

### Monad

```haskell
class Applicative m => Monad m where
    (>>=) :: m a -> (a -> m b) -> m b
    join :: m (m a) -> m a
```

{% hint style="success" %}
`>>=` is pronounced "bind". This is because when you use it to link into the next function, it "binds" to the first argument, like a supercharged pipe. Further, because of how scoping rules work, argument name bindings last until the end of the chain:

```haskell
[1,2,3] >>= \a -> 
  [a + 1] >>= \b -> 
    [(show b, a * 10)]
-- [("2", 10), ("3", 20), ("4", 30)]
```
{% endhint %}

{% hint style="warning" %}
There's also an inverted bind operator, =&lt;&lt; with arguments reversed
{% endhint %}

Let's look at a couple instances:

```haskell
instance Monad [a] where
  join xs = foldr (++) [] xs
  xs >>= f = join (f <$> xs)
  
instance Monad (Maybe a) where
  join Nothing      = Nothing
  join (Just inner) = inner
  
  Nothing >>= f = Nothing
  Just x  >>= f = Just (f x)
```

`>>=` is special compared to `<$>` and `<*>`. It doesn't run out of arguments, and can be chained forever. 

```haskell
[1,2,3] >>= \a -> 
  pure (a + 1) >>= \b -> 
    pure (b * 10) >>= \c ->
      pure (c - 5)
-- [15, 25, 35]
```

It can also be used to change behaviour, changing its structure based on arguments:

```haskell
[1,2,3] >>= \a ->
  if rem a 2 == 0 then [a + 1] else [a, a * 2, a * 3] >>= \b ->
    if b > 10 then [a] else [b]
-- [1, 2, 3, 3, 3, 6, 9]

Just 42 >>= \a ->
  if a > 10 then Nothing else Just a >>= \b ->
    Just (b * 10) >>= \c ->
      Just (show c)
-- Nothing
```

#### Do Notation

Above there's a lot of `>>=` and nesting to keep it straight. Haskell has "do notation" to clean this up, make it feel like an linear set of instructions, and also pun on a lot of imperative programming.

When using do notion, we often replace `pure` with `return`, which is the same function but renamed for extra imperative punning.

{% hint style="danger" %}
`return` does not exit out of the function chain. It is the same function as `pure` and only wraps values in the correct container.
{% endhint %}

{% tabs %}
{% tab title="Manual" %}
```haskell
[1,2,3] >>= \a -> 
  pure (a + 1) >>= \b -> 
    pure (b * 10) >>= \c ->
      pure (c - 5)
```
{% endtab %}

{% tab title="Do Notation" %}
```haskell
do
  a <- [1, 2, 3]
  b <- return $ a + 1
  c <- return $ b * 10
  return $ c - 5
```
{% endtab %}
{% endtabs %}

There's also a special variant of `let` that lets you skip the `<-` for simple values.

{% tabs %}
{% tab title="Without Let" %}
```haskell
do
  a <- [1, 2, 3]
  b <- return (a * 10)
  return (b + 1)
```
{% endtab %}

{% tab title="With Let" %}
```haskell
do
  let as = [1, 2, 3]
  let b = fmap (*10) as
  return (b + 1)
```
{% endtab %}
{% endtabs %}

### Cheat Sheet

How these all relate to each other can be hard to keep straight at first. Here's a handy guide in pseudocode:

```haskell
-- WARNING: Abuse of notation!

  (a ->   b)  $    a ==   b -- Plain function
  (a ->   b) <$> m a == m b -- Functor
m (a ->   b) <*> m a == m b -- Applicative
  (a -> m b) =<< m a == m b -- Monad
  
{-| Legend
    a is a value
    b is a value
    m is a data wrapper / container
-}
```

