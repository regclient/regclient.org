---
title: regctl image copy
layout: single
warning: Auto generated content
---

## Synopsis

Copy or retag an image. This works between registries and only pulls layers
that do not exist at the target. In the same registry it attempts to mount
the layers between repositories. And within the same repository it only
sends the manifest with the new tag.

The destination reference may include go template syntax, expanded using the
source reference (e.g. "{{.Source.Tag}}"), allowing the destination tag to be
derived from the source.

```shell
regctl image copy <src_image_ref> <dst_image_ref> [flags]
```

## Aliases

- cp

## Examples

```shell
# copy an image
regctl image copy \
  ghcr.io/regclient/regctl:edge registry.example.org/regclient/regctl:edge

# retag using a template to derive the destination tag from the source
regctl image copy \
  ghcr.io/regclient/regctl:edge 'registry.example.org/regclient/regctl:{{.Source.Tag}}-mirror'

# copy an image with signatures
regctl image copy --digest-tags \
  ghcr.io/regclient/regctl:edge registry.example.org/regclient/regctl:edge

# copy only the local platform image
regctl image copy --platform local \
  ghcr.io/regclient/regctl:edge registry.example.org/regclient/regctl:edge

# retag an image
regctl image copy registry.example.org/repo:v1.2.3 registry.example.org/repo:v1

# copy an image to an OCI Layout including referrers
regctl image copy --referrers \
  ghcr.io/regclient/regctl:edge ocidir://regctl:edge

# copy a windows image, including foreign layers
regctl image copy --platform windows/amd64,osver=10.0.17763.4974 --include-external \
  golang:latest registry.example.org/library/golang:windows
```

## Options

```text
      --digest-tags            Include digest tags ("sha256-<digest>.*") when copying manifests
      --fast                   Fast check, skip referrers and digest tag checks when image exists, overrides force-recursive
      --force-recursive        Force recursive copy of image, repairs missing nested blobs and manifests
      --format string          Format output with go template syntax
      --include-external       Include external layers
  -p, --platform string        Specify platform (e.g. linux/amd64 or local)
      --referrers              Include referrers
      --referrers-src string   External source for referrers
      --referrers-tgt string   External target for referrers
```

## Options from parent commands

```text
      --host stringArray     Registry hosts to add (reg=registry,user=username,pass=password,tls=enabled)
      --logopt stringArray   Log options
      --user-agent string    Override user agent
  -v, --verbosity string     Log level (trace, debug, info, warn, error) (default "WARN")
```
