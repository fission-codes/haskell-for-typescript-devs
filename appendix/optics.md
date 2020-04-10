---
description: Common Lens Legend
---

# Appendix II: Optics

My guess is that you're seeing lenses and going "what are all these funky looking operators? 😱" In short, they're getters and setters that let you work on nested data, enums, `Maybe`s, and so on. They compose nicely, so you can all kinds of mutable-feeling updates in a totally controlled way.

The good news is that once you know the lens library, it's used in a TON of places, so it's not wasted effort. You can really get by with knowing a handful of them, and following the patterns.

The idea is that you're zooming into a data structure. You can grab values out, or edit them. Then you zoom back out and the outer structures have also been updated to reflect point at the inner changes.

#### Getters `^`

* `.` go deeper / compose \(normal composition operator\)
* `^.` get / lookup
* `^?` get nullable field \(see `Nullable` below\)

#### Setters `~`

* `&` mutate with / and also \(it's the normal pipe / reverse application operator\)
* `.~` set / replace
* `?~` set a nullable field \(see `Nullable` below\)
* `%~` update with a function \(run function on current value and set it to that\)
* `+~` add to the current value \(e.g. counter\)
* `-~` subtract from current value
* `*~` multiple current value by

#### Nullable `?`

* `^?` get inside a `Maybe` \(i.e. stop lensing if it's a `Nothing`\)
* `?~` set a nullable field \(i.e. `.~ Just newValue`\)

These read nicely, and let you make deeply nested updates either with composition \(`.`\) or with a named lens that is a composition of others.

### Why not use record-update syntax?

You totally can! Doing this manually, you end up having to update each nested record recursively, including the updated sub-records as you go. For example:

```haskell
data Person = Person
  { _name    :: Text
  , _address :: Address
  , _pet     :: Pet
  }

data Pet = Pet
  { _name         :: Text
  , _variety      :: Animal
  , _favouriteToy :: Toy
  }

data Animal = Dog | Cat | Fish

data Toy = Toy 
  { _ name     :: Text
  , _condition :: Condition
  }

data Condition = New | Good | Fair | Bad
```

Let's define an `owner :: Person`:

```haskell
owner = Person
  { _name    = "Alice"
  , _address = "123 Fake Street"
  , _pet     = Pet 
      { _name         = "Fluffy"
      , _variety      = Cat
      , _favouriteToy = Toy 
          { _name      = "wind-up mouse"
          , _condition = New 
          }
      }
  }
```

Manually updating deeply nested records is a bit ugly

```haskell
wearAndTear :: Person
wearAndTear = owner { _pet = myPet { _favouriteToy = myToy { _condition = Fair } } }
  where
    Person { _pet = myPet@(Pet { _favouriteToy = myToy })} = owner
```

Lenses read better

```haskell
wearAndTear :: Person
wearAndTear = owner & pet . favouriteToy . condition .~ Fair
```

Let's break that down 🕺

```haskell
--                Pipe                            Replace
--                  |                                |
--                  v                                v
wearAndTear = owner & pet . favouriteToy . condition .~ Fair
--              ^       ^         ^           ^          ^
--              |       |         |           |          |
--              |       +---------+-----------+     Replace with
--         Initial Value          |
--                           Nested path
--                    (like in OO dot-notation)
--                 owner.pet.favouriteToy.condition
```

These can also be turned into helper functions, chained together, and so on

```haskell
playWithPet :: Person -> Person
playWithPet = pet . favouriteToy . condition .~ Fair

-- Use

playWithPet owner

-- Oh no! Fluffy ran away 😭 Let's play with our new cat: Mittens!

owner & pet.name.~"Mittens" 
      & playWithPet

-- And exciting! We moved, and got a dog to play with

owner & address      .~ "1066 West Hastings Street"
      & pet.variety  .~ Dog
      & pet.name     .~ "Lassie"
      & pet.toy.name .~ "Tennis ball" 
      & playWithPet
```

We can also automate some behaviour while we're at it:

```haskell
playWithPet' :: Person -> Person
playWithPet' = pet . favouriteToy . condition %~ wearDown

wearDown :: Condition -> Condition
wearDown New  = Good
wearDown Good = Fair
wearDown Fair = Bad
wearDown Bad  = Bad
```



