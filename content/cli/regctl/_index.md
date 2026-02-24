---
title: regctl
layout: single
warning: Auto generated content
---

## Synopsis

Utility for accessing docker registries
More details at <https://regclient.org>

## Available Commands

- [regctl artifact](./artifact)
- [regctl blob](./blob)
- [regctl completion](./completion)
- [regctl config](./config)
- [regctl image](./image)
- [regctl index](./index)
- [regctl manifest](./manifest)
- [regctl registry](./registry)
- [regctl repo](./repo)
- [regctl tag](./tag)
- [regctl version](./version)

## Examples

```shell
# login to ghcr.io
regctl registry login ghcr.io

# configure a local registry for http
regctl registry set --tls disabled registry.example.org

# copy an image from ghcr.io to local registry
regctl image copy ghcr.io/regclient/regctl:latest registry.example.org/regctl:latest

# show debugging output from a command
regctl tag ls ghcr.io/regclient/regctl -v debug

# format log output in json
regctl image ratelimit --logopt json alpine

# override registry config for a single command
regctl image digest --host reg=localhost:5000,tls=disabled localhost:5000/repo:v1
```

## Options

```text
      --host stringArray     Registry hosts to add (reg=registry,user=username,pass=password,tls=enabled)
      --logopt stringArray   Log options
      --user-agent string    Override user agent
  -v, --verbosity string     Log level (trace, debug, info, warn, error) (default "WARN")
```
