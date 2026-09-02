---
title: regctl artifact get
layout: single
warning: Auto generated content
---

## Synopsis

Download artifacts from the registry.

```shell
regctl artifact get <reference> [flags]
```

## Aliases

- pull

## Examples

```shell
# download a helm chart
regctl artifact get registry.example.org/helm-charts/chart:0.0.1 > chart.tgz

# retrieve the SPDX SBOM for the latest regsync image for this platform
regctl artifact get \
  --subject ghcr.io/regclient/regsync:latest \
  --filter-artifact-type application/spdx+json \
  --platform local | jq .
  
# retrieve the artifact config rather than the artifact itself
regctl artifact get registry.example.org/artifact:0.0.1 --config
```

## Options

```text
      --config                          Show the config, overrides file options
      --config-file string              Output config to a file
      --external string                 Query referrers from a separate source
  -f, --file stringArray                Filter by artifact filename
  -m, --file-media-type stringArray     Filter by artifact media-type
      --filter-annotation stringArray   Filter referrers by annotation (key=value)
      --filter-artifact-type string     Filter referrers by artifactType
      --latest                          Get the most recent referrer using the OCI created annotation
  -o, --output string                   Output directory for multiple artifacts
  -p, --platform string                 Specify platform of a subject (e.g. linux/amd64 or local)
      --slow-referrers                  Search for referrers with slower queries
      --sort-annotation string          Annotation used for sorting results
      --sort-desc                       Sort in descending order
      --strip-dirs                      Strip directories from filenames in output dir
      --subject string                  Get a referrer to the subject reference
```

## Options from parent commands

```text
      --host stringArray     Registry hosts to add (reg=registry,user=username,pass=password,tls=enabled)
      --logopt stringArray   Log options
      --user-agent string    Override user agent
  -v, --verbosity string     Log level (trace, debug, info, warn, error) (default "WARN")
```
