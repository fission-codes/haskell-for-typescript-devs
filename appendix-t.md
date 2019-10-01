---
description: Haskell Tips & Tricks
---

# Appendix T

## How do I change my REPL prompt?

Here's a simple one liner to set your prompt:

`:set prompt "λ> "`

To set it permanently, look at the config file in `~/.ghci`. Here's an example custom prompt:

```text
:set prompt "\ESC[1;34m\n\ESC[0;34mλ> \ESC[m"
-- :seti -XNoImplicitPrelude
:seti -XOverloadedStrings
:seti -XScopedTypeVariables

:set -Wall
:set -fno-warn-type-defaults
:set -package pretty-show

import Text.Show.Pretty (pPrint)
:set -interactive-print pPrint
:set +s
:set +t

:def rt const $ return $ unlines [":r", ":main --rerun-update"]
:def rtfe const $ return $ unlines [":r", ":main --rerun-update --rerun-filter failures,exceptions"]
:def rtn const $ return $ unlines [":r", ":main --rerun-update --rerun-filter new"]
```

## More tips go here

This would be another tip.



