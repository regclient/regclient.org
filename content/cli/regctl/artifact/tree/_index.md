---
title: regctl artifact tree
layout: single
warning: Auto generated content
---

## Synopsis

Return a graph of manifests and referrers to those manifests.
This command will recursively query referrers to all child images.
For a single image, it is better to run "regctl artifact list".

```shell
regctl artifact tree <reference> [flags]
```

## Examples

```shell
# list all referrers to the latest regsync image
regctl artifact tree ghcr.io/regclient/regsync:latest

# include digest tags (used by sigstore)
regctl artifact tree --digest-tags ghcr.io/regclient/regsync:latest
```

## Options

```text
      --digest-tags                     Include digest tags
      --external string                 Query referrers from a separate source
      --filter-annotation stringArray   Filter descriptors by annotation (key=value)
      --filter-artifact-type string     Filter descriptors by artifactType
      --format string                   Format output with go template syntax (default "{{printPretty .}}")
      --slow-referrers                  Search for referrers with slower queries
```

## Options from parent commands

```text
      --host stringArray     Registry hosts to add (reg=registry,user=username,pass=password,tls=enabled)
      --logopt stringArray   Log options
      --user-agent string    Override user agent
  -v, --verbosity string     Log level (trace, debug, info, warn, error) (default "WARN")
```
