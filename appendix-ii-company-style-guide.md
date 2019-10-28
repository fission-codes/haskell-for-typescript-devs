---
description: We focus on storytelling in our code
---

# Appendix II: Fisson Style Guide

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
rawList 
  :: MonadRIO          cfg m
  => Has IPFS.BinPath  cfg
  => Has IPFS.Timeout  cfg
  => HasProcessContext cfg
  => HasLogFunc        cfg
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

## Pipes

We use pipes

