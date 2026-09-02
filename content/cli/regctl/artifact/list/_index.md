---
title: regctl artifact list
layout: single
warning: Auto generated content
---

## Synopsis

List artifacts that have a subject to the given reference.

```shell
regctl artifact list <reference> [flags]
```

## Aliases

- ls

## Examples

```shell
# list all referrers of the regsync package for the local platform
regctl artifact list ghcr.io/regclient/regctl --platform local

# return the original referrers response
regctl artifact list registry.example.com/repo:v1 --format body

# pretty print the referrers response
regctl artifact list registry.example.com/repo:v1 --format '{{jsonPretty .Manifest}}'
```

## Options

```text
      --digest-tags                     Include digest tags
      --external string                 Query referrers from a separate source
      --filter-annotation stringArray   Filter descriptors by annotation (key=value)
      --filter-artifact-type string     Filter descriptors by artifactType
      --format string                   Format output with go template syntax (default "{{printPretty .}}")
      --latest                          Sort using the OCI created annotation
  -p, --platform string                 Specify platform (e.g. linux/amd64 or local)
      --slow-referrers                  Search for referrers with slower queries
      --sort-annotation string          Annotation used for sorting results
      --sort-desc                       Sort in descending order
```

## Options from parent commands

```text
      --host stringArray     Registry hosts to add (reg=registry,user=username,pass=password,tls=enabled)
      --logopt stringArray   Log options
      --user-agent string    Override user agent
  -v, --verbosity string     Log level (trace, debug, info, warn, error) (default "WARN")
```
