---
title: regctl version
layout: single
warning: Auto generated content
---

## Synopsis

Show the version of regctl. Note that docker image builds will always be marked "dirty".

```shell
regctl version [flags]
```

## Examples

```shell
# display full version details
regctl version

# retrieve the version number
regctl version --format '{{.VCSTag}}'
```

## Options

```text
      --format string   Format output with go template syntax (default "{{printPretty .}}")
```

## Options from parent commands

```text
      --host stringArray     Registry hosts to add (reg=registry,user=username,pass=password,tls=enabled)
      --logopt stringArray   Log options
      --user-agent string    Override user agent
  -v, --verbosity string     Log level (trace, debug, info, warn, error) (default "WARN")
```
