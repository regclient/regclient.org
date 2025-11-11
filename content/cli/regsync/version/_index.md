---
title: regsync version
layout: single
warning: Auto generated content
---

## Synopsis

Show the version of regsync. Note that docker image builds will always be marked "dirty".

```shell
regsync version [flags]
```

## Examples

```shell
# display full version details
regsync version

# retrieve the version number
regsync version --format '{{.VCSTag}}'
```

## Options

```text
      --format string   Format output with go template syntax (default "{{printPretty .}}")
```

## Options from parent commands

```text
      --logopt stringArray   Log options
  -v, --verbosity string     Log level (trace, debug, info, warn, error) (default "INFO")
```
