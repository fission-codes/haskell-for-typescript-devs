---
description: Translating from TypeScript to Haskell
---

# Syntatic Cheat Sheet

## Basics

### Comments

{% tabs %}
{% tab title="TypeScript" %}
```typescript
// This is a TypeScript comment

/*
Need more space?
No problem!
This is a multi-line TypeScript comment :)
*/

/**
 * In TS, you write documentation like this for TypeDoc
 */
 const id = a => a
```
{% endtab %}

{% tab title="Haskell" %}
```haskell
-- This is a Haskell comment

{-
Need more space?
No problem!
This is a multi-line Haskell comment :)
-}

--| Add a pipe for a short function description
--  You can keep talking about is afterwards
--  These will turn into formatted documentation with Haddock
id a = a

{-| You can use a multi-line comment for Haddock, too
    The same rules apply.
-}
swap (a, b) -> (b, a)
```
{% endtab %}
{% endtabs %}

### Assignment

#### Implicit Type

{% tabs %}
{% tab title="TypeScript" %}
```typescript
const hi = "Hello World"
```
{% endtab %}

{% tab title="Haskell" %}
```haskell
hi = "Hello World"
```
{% endtab %}
{% endtabs %}

#### Explicit Type

{% tabs %}
{% tab title="TypeScript" %}
```typescript
const hi: string = 'Hello World'
```
{% endtab %}

{% tab title="Haskell" %}
```haskell
hi :: Text
hi = "Hello World"
```
{% endtab %}

{% tab title="Haskell \(Local Clarification Variant\)" %}
```haskell
hi = ("Hello World" :: Text)
```
{% endtab %}
{% endtabs %}

### Functions

#### Anonymous

{% tabs %}
{% tab title="TypeScript" %}
```typescript
a => a * 2
```
{% endtab %}

{% tab title="Haskell" %}
```haskell
\a -> a * 2
-- NOTE: `\` is an easier to type lambda `λ`
```
{% endtab %}

{% tab title="Haskell \(Shorthand\)" %}
```haskell
(*2)
```
{% endtab %}
{% endtabs %}

#### Named

{% tabs %}
{% tab title="TypeScript" %}
```typescript
const double = (a : number) => a * 2
```
{% endtab %}

{% tab title="Haskell" %}
```haskell
double :: Int -> Int
double a = a * 2
```
{% endtab %}

{% tab title="Haskell \(Shorthand\)" %}
```haskell
double :: Int -> Int
double = (*2)
```
{% endtab %}
{% endtabs %}

#### Infix

{% tabs %}
{% tab title="TypeScript" %}
```typescript
// Does not exist
```
{% endtab %}

{% tab title="Haskell" %}
```haskell
(*&) :: Int -> String -> String
num *& str = show num ++ str
-- NOTE: `show` turns values into strings 
```
{% endtab %}
{% endtabs %}

#### Argument Application

{% tabs %}
{% tab title="TypeScript" %}
```typescript
const result = doTheThing(1, 2, ["a", "b"], 1.1)
```
{% endtab %}

{% tab title="Haskell" %}
```haskell
result = doTheThing 1 2 ["a", "b"] 1.1
```
{% endtab %}
{% endtabs %}

#### Side-Effectful

{% tabs %}
{% tab title="TypeScript" %}
```typescript
fireTheMissiles()
```
{% endtab %}

{% tab title="Haskell" %}
```haskell
fireTheMissiles
```
{% endtab %}
{% endtabs %}

#### Inner Scope

{% tabs %}
{% tab title="TypeScript" %}
```typescript
const withInner = (toLog: string): () => {
  const innerValue = 'That is the question!'
  console.log(`${toLog}? Or not ${toLog}? ${innerValue}`)
}
```
{% endtab %}

{% tab title="Haskell \(let...in\)" %}
```haskell
withInner' toLog =
  let 
    innerValue = "That is the question!"
    msg = toLog <> "? Or not " <> toLog <> "? " <> innerValue
  in
    logInfo msg
```
{% endtab %}

{% tab title="Haskell \(where\)" %}
```haskell
withInner toLog = logInfo msg
  where
    msg = toLog <> "? Or not " <> toLog <> "? " <> innerValue
    innerValue = "That is the question!"
```
{% endtab %}
{% endtabs %}

#### Nested Application

{% tabs %}
{% tab title="TypeScript" %}
```typescript
const result = triple(prod(1, double(1), 3))

/* i.e.
  const two = double(1)
  const six = prod(1, two, 3)
  const result = triple(six)
*/
```
{% endtab %}

{% tab title="Haskell \(parens\)" %}
```haskell
result = triple (prod 1 (double 1) 3)
```
{% endtab %}

{% tab title="Haskell \(applied with $\)" %}
```haskell
result = triple $ prod 1 (double 1) 3
```
{% endtab %}

{% tab title="Haskell \(Piped with &\)" %}
```haskell
result = 3 
       & prod 1 (double 1)
       & triple
```
{% endtab %}
{% endtabs %}

#### Composition

{% tabs %}
{% tab title="TypeScript" %}
```typescript
const timesSix = (a: number): number => triple(double(a))
```
{% endtab %}

{% tab title="Haskell" %}
```haskell
timesSix :: Int -> Int
timeSix = triple . double
```
{% endtab %}
{% endtabs %}

#### Partial Application

{% tabs %}
{% tab title="TypeScript" %}
```typescript
const math = (n: number, m: number): number =
  triple(prod(10, double(1), n), m)
```
{% endtab %}

{% tab title="Haskell" %}
```haskell
math :: Int -> Int -> Int
math n m = triple (prod 10 (double 1) n) m
```
{% endtab %}

{% tab title="Haskell \(Point-Free Style\)" %}
```haskell
math :: Int -> Int -> Int
math = triple . prod 10 (double 1)
```
{% endtab %}
{% endtabs %}

### Modules

{% tabs %}
{% tab title="TypeScript" %}
```typescript
import * from "./GetEverything"
import * as HL from "./HelperLib";
import { foo, bar } from "./SomeLib"

const id = a => a
const swap = [a, b] => [b, a]
const hiddenValue = HL.downcase("not exported")

export { 
  id, 
  swap 
}

export { toReExport } from "./ExternalLib"
```
{% endtab %}

{% tab title="Haskell" %}
```haskell
module MyModule
  ( id
  , swap
  , toReExport
  ) where

import           GetEverything
import           SomeLib     (foo, bar)
import           ExternalLib (toReExport)
import qualified HelperLib   as HL

id a = a
swap (a, b) = (b, a)
hiddenValue = HL.downcase "not exported"
```
{% endtab %}
{% endtabs %}

## Types & Data

### Type Alias

{% tabs %}
{% tab title="TypeScript" %}
```typescript
type Name = string
type Balance = number
```
{% endtab %}

{% tab title="Haskell" %}
```haskell
type Name    = Text
type Balance = Int
```
{% endtab %}
{% endtabs %}

### Sum / Enum / Coproduct

{% tabs %}
{% tab title="TypeScript" %}
```typescript
type TrafficLight = 'RED' | 'YELLOW' | 'GREEN'
```
{% endtab %}

{% tab title="Haskell" %}
```haskell
data TrafficLight
  = Red
  | Yellow
  | Green
```
{% endtab %}
{% endtabs %}

### Product

{% tabs %}
{% tab title="TypeScript" %}
```typescript
type GearRatio = [number, number] // [big gear, little gear]
type Bike = [string, GearRatio] // [bikeName, gear ratio]
```
{% endtab %}

{% tab title="Haskell" %}
```haskell
data GearRatio = GearRatio Int  Int
data Bike      = Bike      Text GearRatio
```
{% endtab %}
{% endtabs %}

#### Interfaces & Classes → Records

{% tabs %}
{% tab title="TypeScript \(Interface\)" %}
```typescript
interface GearRatio {
  big: number
  little: number
}

interface Bike {
  name: string
  gear: GearRatio
}
```
{% endtab %}

{% tab title="TypeScript \(Class\)" %}
```typescript
class Geared {
  big: number
  little: number
  
  constructor(gbr: number, lgr: number) {
    big = bgr
    little = lgr
  }
}

class Bike extends Geared {
  name: string
  
  constructor(bgr: number, lgr: number, bikeName: string) {
    name = bikeName
    super(bgr, lgr)
  }
}
```
{% endtab %}

{% tab title="Haskell" %}
```haskell
data Gear = GearRatio
  { _big    :: Natural
  , _little :: Natural
  }
  
data Bike = Bike
  { _name :: Text
  , _gear :: Gear
  }
```
{% endtab %}
{% endtabs %}

#### Constructors

{% tabs %}
{% tab title="TypeScript \(Direct\)" %}
```typescript
const makeGear = (big, little): GearRatio => { big, little }

const makeBike = (big, little, name): Bike => {
  name, 
  gear = makeGear(big, little)
}
```
{% endtab %}

{% tab title="TypeScript \(Class\)" %}
```typescript
/*
  Included in the `class` delcaration. For example:
  
  constructor(bgr: number, lgr: number) {
    big = bgr
    little = lgr
  }
*/
```
{% endtab %}

{% tab title="Haskell" %}
```haskell
{-
  Constructors are the capitalized word on 
  the right of the `=` in the `data` declaration
  It auto-generates constructor functions 
  with the folling type signatures:

  GearRatio :: Natural -> Natural   -> Geared
  Bike      :: Text    -> GearRatio -> Bike
-}
```
{% endtab %}
{% endtabs %}

#### Instantiation

{% tabs %}
{% tab title="TypeScript \(Direct\)" %}
```typescript
const myBike = makeBike(2, 5, '10 Speeder')
```
{% endtab %}

{% tab title="TypeScript \(Class\)" %}
```typescript
const myBike = new Bike(2, 5, '10 Speeder')
```
{% endtab %}

{% tab title="Haskell" %}
```haskell
myBike = Bike
  { _name = "10 Speeder"  -- Use named fields
  , _gear = GearRatio 2 5 -- Or just ordered values
  }
```
{% endtab %}

{% tab title="Haskell \(Wildcard Style\)" %}
```haskell
myBike :: Bike
myBike = 
  let 
    _big    = 2
    _little = 5
    _name   = "10 Speeder"
    _gear   = GearRatio {..}
  in 
    Bike {..}
```
{% endtab %}
{% endtabs %}

### Getters

{% tabs %}
{% tab title="TypeScript \(Dot Syntax\)" %}
```typescript
const big: number = myGears.big
const little = myBike.gear.little
```
{% endtab %}

{% tab title="TypeScript \(Destructuring\)" %}
```typescript
const { bigGear } = myGears
const getBig = ({ big }) => big
const ratio = ({ big, little }) => big / little
```
{% endtab %}

{% tab title="Haskell \(Destructuring\)" %}
```haskell
GearRatio big _ = myGears
getBig (GearRatio {_big}) = _big
ratio (GearRatio {_big, _little}) = _big / _little
```
{% endtab %}

{% tab title="Haskell \(Wildcard Style\)" %}
```haskell
ratio (GearRatio {..}) = _big / _little
```
{% endtab %}

{% tab title="Haskell \(Lenses\)" %}
```haskell
import Control.Lens.TH

$(makeLenses ''Gear)
$(makeLenses ''Bike)

myGear ^. big -- NOTE: lenses have no underscores!

-- Nested

myBike ^. gear . little
```
{% endtab %}
{% endtabs %}

### Updating with Setters

{% tabs %}
{% tab title="TypeScript" %}
```typescript
// Preamble

const myGear = GearRatio 2 5
const myBike = Bike '10 Speeder' myGear

// Mutable Update

myGear.littleGear = 6
myBike.name = '12 Speeder'

// Immutable Update

const upgradedGear = Object.assign({}, myGear)
const upgradedBike = Object.assign({}, myBike, {gear: upgradedGear})
```
{% endtab %}

{% tab title="Haskell \(Record Syntax\)" %}
```haskell
-- Preamble

myGear = GearRatio 2 5
myBike = Bike "10 Speeder" myGear

-- Immutable Update

upgradedBike = myBike 
  { _name = "12 Speeder"
  , _gear = myGear { _littleGear = 6 }
  }
```
{% endtab %}

{% tab title="Haskell \(Lenses\)" %}
```haskell
-- Preamble

import Control.Lens.TH

makeLenses ''Gear
makeLenses ''Bike

myGear = GearRatio 2 5
myBike = Bike "10 Speeder" myGear

-- Immutable Update

upgradedBike = myBike & name .~ "12 Speeder"
                      & gear.littleGear .~ 6
```
{% endtab %}
{% endtabs %}

## Branching

### If...Else

{% tabs %}
{% tab title="TypeScript" %}
```typescript
if (condition) {
  branchA
} else {
  branchB
}
```
{% endtab %}

{% tab title="TypeScript Ternary" %}
```typescript
condition ? branchA : branchB
```
{% endtab %}

{% tab title="Haskell" %}
```haskell
if condition
   then branchA
   else branchB
```
{% endtab %}
{% endtabs %}

### If...Else If...Else

{% tabs %}
{% tab title="TypeScript" %}
```typescript
if (percent >= 90 || student.paidBribe) {
  return GRADES.A
} else if (percent >= 75) {
  return GRADES.B
} else if (percent >= 60) {
  return GRADES.C
} else if (percent >= 50) {
  return GRADES.D
} else {
  return GRADES.F
}
```
{% endtab %}

{% tab title="Haskell" %}
```haskell
if | percent >= 90 || paidBribe student = A
   | percent >= 75 = B
   | percent >= 60 = C
   | percent >= 50 = D
   | otherwise     = F
```
{% endtab %}
{% endtabs %}

### Switch/Case

{% tabs %}
{% tab title="TypeScript" %}
```typescript
let nextAction

switch (lightState) {
  case LIGHT.RED:
    nextAction = ACTION.STOP
    break
    
  case LIGHT.YELLOW:
    nextAction = ACTION.SLOW
    break
    
  case LIGHT.GREEN:
    nextAction = ACTION.DRIVE
    break
    
  default:
    nextAction = currentAction
}
```
{% endtab %}

{% tab title="Haskell" %}
```haskell
nextAction = 
  case lightState of
    Red    -> Stop
    Yellow -> Slow
    Green  -> Drive
    _      -> currentAction
```
{% endtab %}
{% endtabs %}

