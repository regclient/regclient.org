---
title: regctl
layout: single
date: 2025-01-01
---

{{% toc %}}

`regctl` is a command line interface to the Go library designed to facilitate adhoc queries and scripting.
See the [CLI Reference](/cli/regctl/) for a full list of commands and and their usage.
All `regctl` commands have a `--help` flag that shows available child commands, usage, flags, and often contain examples.

## Demo

{{< asciinema src="/casts/regctl-demo.cast" cols=100 rows=26 autoPlay=true loop=true >}}

## Configuring Registries

Registries you access may have different configurations, including login credentials, TLS, and mirrors.
By default, `regctl` will include credentials from Docker, and use sensible defaults.
Some `regctl` commands will persist registry configurations to a file, `${HOME}/.regctl/config.json` by default, which can be overridden with the `$REGCTL_CONFIG` environment variable.
Individual commands can change settings with the `--host` option.

### Persistent Configuration

To disable TLS (for http only registries):

```console
regctl registry set registry.example.org:5000 --tls disabled
```

To configure a self signed certificate:

```console
regctl registry set registry.example.org:5000 --cacert "$(cat ca.pem)"
```

To login to the registry (credentials are stored in clear-text):

```console
# interactive
regctl registry login registry.example.org

# from a shell script
echo "${token}" | regctl registry login registry.example.org -u "${username}" --pass-stdin
```

To use a credential helper:

```console
# docker-credential-command will vary for the helper you have installed, e.g. docker-credential-ecr
regctl registry set registry.example.org:5000 --cred-helper docker-credential-command
```

### Per-Command Configuration

To disable TLS (for http only registries):

```console
regctl --host "reg=registry.example.org,tls=disabled" ...
```

To login to the registry (credentials are passed in clear-text):

```console
regctl --host "reg=registry.example.org,user=${username},pass=${token}" ...
```

### Multiple Logins for the Same Registry

Some use cases require different configurations for accessing the same registry, e.g. to pass different credentials for accessing specific repositories.
In those situations, it is useful to define a separate registry name and use the "hostname" option.

```console
# login with your normal credentials
regctl registry login registry.example.org

# setup a production login to the same host using a separate registry name
regctl registry set prod.registry.example.org --hostname registry.example.org
regctl registry login prod.registry.example.org

# now each entry can be used
regctl image copy registry.example.org/project/dev:v1.2.3 prod.example.org/public/app:v1.2.3
```

Note that since regclient is treating these as two separate registries, there may be additional overhead when copying content.

### Mirrors

A local mirror may be configured for pulling content:

```console
regctl registry set --mirror mirror.example.org docker.io
```

If you have multiple mirrors, you can configure them with priorities, larger numbers are tried first:

```console
regctl registry set --priority 10 mirror-build.example.org:5000
regctl registry set --priority  5 mirror-cluster.example.org:5000
regctl registry set --mirror mirror-build.example.org:5000 --mirror mirror-cluster.example.org:5000 docker.io
```

Note that mirrors are not used for pushing content.
Also note that regclient does not verify if the content on a mirror is stale, it only attempts other registries when the requested content is not found or the mirror is down.

## Logging

By default, `regctl` only outputs warnings or greater from the logging interface.
Additional logging levels are available with the `-v` option.
Logs are structured (including `key=value` in addition to a message) and the output can be switched from text based to JSON with `--logopt json`.
Log output is always sent to stderr so that other command output can be captured on stdout.

```console
regctl repo ls registry.example.org -v trace --logopt json
```

## Tab Completion

Some CLI shells offer tab completion.
For details on the supported shells and setting this up with your shell, see [`regctl completion`](/cli/regctl/completion/).
For example, setting up tab completion in the current bash shell, that may be setup with:

```console
source <(regctl completion bash)
```

## Format Flag

The `--format` flag allows you to apply a Go template to the output of some commands.
There are several useful links for understanding how to use Go templates:

- [Go template documentation](https://golang.org/pkg/text/template/)
- [regclient template functions](/usage/#template-functions)
- [OCI image spec](https://github.com/opencontainers/image-spec/tree/master/specs-go/v1)
- [Docker manifest](https://github.com/distribution/distribution/tree/main/manifest/schema2)
- [Docker manifest list](https://github.com/distribution/distribution/tree/main/manifest/manifestlist)

Several commands expand the following format strings:

- `raw`: this returns the raw headers and body.
- `rawBody`, `raw-body`, or `body`: this returns the original body of the response.
- `rawHeaders`, `raw-headers`, or `headers`: this returns the full HTTP headers of the response.

Examples:

```shell
regctl image manifest --format '{{range .Layers}}{{println .Digest}}{{end}}' openjdk:latest # show each layer digest

regctl image inspect --format '{{jsonPretty .}}' alpine:latest

regctl image inspect --format '{{range $k, $v := .Config.Labels}}{{$k}} = {{$v}}{{println}}{{end}}' ... # loop through labels

regctl image inspect --format '{{index .Config.Labels "org.opencontainers.image.version"}}' regclient/regctl:latest # output a specific label

regctl image manifest --format raw-body alpine:latest # returns the raw manifest
```
