---
title: regctl registry
layout: single
warning: Auto generated content
---

## Synopsis

Retrieve or update registry configurations.
By default, the configuration is loaded from $HOME/.regctl/config.json.
This location can be overridden with the REGCTL_CONFIG environment variable.
Note that these commands do not include logins imported from Docker or values injected with --host.

## Available Commands

- [regctl registry config](./config)
- [regctl registry login](./login)
- [regctl registry logout](./logout)
- [regctl registry set](./set)
- [regctl registry whoami](./whoami)

## Options from parent commands

```text
      --host stringArray     Registry hosts to add (reg=registry,user=username,pass=password,tls=enabled)
      --logopt stringArray   Log options
      --user-agent string    Override user agent
  -v, --verbosity string     Log level (trace, debug, info, warn, error) (default "WARN")
```
