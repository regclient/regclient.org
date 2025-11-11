---
title: regctl repo copy
layout: single
warning: Auto generated content
---

## Synopsis

Copy images from the source to destination repository.
Existing images in the destination are not deleted, but tags may be overwritten.
If include/exclude options are provided, only entries that match one include
option and are not excluded by any exclude option are copied.

```shell
regctl repo copy <source_repo> <dest_repo> [flags]
```

## Aliases

- cp

## Examples

```shell
# copy all tags from the a to b
regctl repo copy registry-a.example.org/repo registry-b.example.org/repo

# copy all tags beginning with v1.2
regctl repo copy --include 'v1\\.2.*' registry-a.example.org/repo registry-b.example.org/repo
```

## Options

```text
      --concurrent int        Number of concurrent images to copy (default 2)
      --exclude stringArray   Exclude tags by regexp
      --include stringArray   Include tags by regexp
      --new-tags              Only copy tags that do not exist in destination repo
      --referrers             Include referrers
```

## Options from parent commands

```text
      --host stringArray     Registry hosts to add (reg=registry,user=username,pass=password,tls=enabled)
      --logopt stringArray   Log options
      --user-agent string    Override user agent
  -v, --verbosity string     Log level (trace, debug, info, warn, error) (default "WARN")
```
