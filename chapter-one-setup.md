# Chapter One: Setup

## Build Tool

We use [Stack](https://docs.haskellstack.org/en/stable/README/) as our build tool, and [HPack](https://github.com/sol/hpack) for package configuration. Dependencies, test modules, executables to build, compiler flags, and the like are found in `packag.yaml`.

## Intero

 If you install [Intero](https://haskell-lang.org/intero), you'll get a bunch of nice editor/IDE features out of the box [🚀](https://emojipedia.org/rocket/)

## Language Extensions

GHC \(the main Haskell compiler\) has a number of "language extensions" available. These are things that are available in GHC, but not yet in the language spec \(which is updated every 10 years or so\). These are primarily to modernize to the lanaguge, and make your life much easier. For reference, here's the list of extension that we have enabled standard.

`ApplicativeDo`, `BangPatterns`, `BlockArguments`, `ConstraintKinds`, `DataKinds`, `DeriveAnyClass`, `DeriveFoldable`, `DeriveFunctor`, `DeriveGeneric`, `DeriveLift`, `DeriveTraversable`, `DerivingStrategies`, `FlexibleContexts`, `FlexibleInstances`, `FunctionalDependencies`, `GADTs`, `GeneralizedNewtypeDeriving`, `KindSignatures`, `LambdaCase`, `LiberalTypeSynonyms`, `MultiParamTypeClasses`, `MultiWayIf`, `NamedFieldPuns`, `NoImplicitPrelude`, `NoMonomorphismRestriction`, `OverloadedStrings`, `OverloadedLabels`, `OverloadedLists`, `RankNTypes`, `RecordWildCards`, `ScopedTypeVariables`, `StandaloneDeriving`, `TupleSections`, `TypeSynonymInstances`, `TemplateHaskell`, `TypeOperators`, `ViewPatterns`

Wow, that looks like a long list! Most of these enable quality-of-life improvements with new syntax or remove boilerplate. Almost all of these are used widely in industry, and are  considered totally safe to use \(and in some cases, it's masochistic to _not_ have them enabled, like `OverloadedStrings`\). You can read up  more on these at the extremely informative [What I Wish I Knew When Learning Haskell](http://dev.stephendiehl.com/hask/).

We mention this here becasue you may find reference to them while on Stack Overflow, or some syntax that's not available in older references like the now woefully out-of-date Learn You a Haskell for Great Good \(which I'm not even going to link\).

## Project Structure

A project is broken into a reusable library, executables, tests, and benchmarks.

### Library

You will often see library code in a `/src` directory. We're following the other convention, which is to place library code in a directory shocking titled... `library`.

### Quality

Quaity checks live in `/test`. We have several kinds of tests:

| Type | Directory |
| :--- | :--- |
| Unit Tests | `/test/testsuite` |
| Doctests | Setup in `/test/doctest`; actual tests in each source file in `/library` |
| Code Linter | `/test/lint` |
| Code Coverage | `/test/coverage-code` |
| Doc Coverage | `/test/coverage-docs` |

### Benchmarks

Benchmarks live at `/bench`

### Executables

If a project contains a single executable, it tends to live in `/app` or `/exe`. Since our project contains multiple executables, we use the name of the executable itself \(for example `/fission-web`\).

## Where to Find Packages

### Stackage

We use [Stackage](https://www.stackage.org/) for nearly all packages. These are stable packages, grouped into "Stack resolver versions", so you don't need to worry about version numbers. Just stick them in your dependencies, and everything just works™.

{% code-tabs %}
{% code-tabs-item title="package.yaml" %}
```yaml
dependencies:
  - aeson
  - aeson-casing
  - base
  - [...]
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Searching in Stackage can be done by name, _or by type signature_, which is very useful.

### Hackage

You can find even more packages on [Hackage](http://hackage.haskell.org/). It's a very similar process as with Stackage, but to include packages

{% code-tabs %}
{% code-tabs-item title="stack.yaml" %}
```yaml
extra-deps:
- alex-3.2.4
- ekg-wai-0.1.0.3
- happy-1.19.10
- servant-multipart-0.11.4
- tasty-rerun-1.1.14
```
{% endcode-tabs-item %}
{% endcode-tabs %}



