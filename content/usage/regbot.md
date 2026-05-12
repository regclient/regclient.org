---
title: regbot
layout: single
date: 2025-01-01
---

{{% toc %}}

`regbot` is a Lua based scripting tool on top of the `regclient` API.

## Operating Modes

`regbot` has various operating modes based on the CLI and flags:

- `regbot server`: Deploys a persistent server running scripts based on the requested schedule.
- `regbot once`: Runs scripts a single time and exits.

Each operating mode accepts the `--dry-run` flag which disables write operations to the registry (push and delete).

## Configuration File

A minimal example that logs a hello world and lists tags looks like:

```yaml
version: 1 # optional for v1 files, there is no v2 yet
creds:
  - registry: registry.example.org
    user: botuser
    pass: 'Pa$$w0rd' # typically you would store secrets outside of this file
  - registry: docker.io
    user: youruser
    pass: "{{file \"/var/run/secrets/hub_token\"}}"
defaults:
  interval: 60m # all scripts run hourly
scripts:
  - name: Hello World
    timeout: 10s
    script: |
      log "hello world"
  - name: Busybox Tags
    timeout: 30s
    script: |
      tags = tag.ls("busybox")
      table.sort(tags)
      for k, t in ipairs(tags) do
        log "Found tag " .. t
      end
```

### Version

```yaml
version: 1
```

This should be left at version 1 or not included at all.
This may be incremented if future `regbot` releases change the configuration file structure.

### Registry Credentials and Settings

`regbot` will attempt to import credentials from `${HOME}/.docker/config.json` by default.
When run from a container, the default path is `/home/appuser/.docker/config.json`.
Use the `creds` to override the imported and default settings.

```yaml
creds:
  - registry: registry.example.org
    user: youruser
    pass: 'Pa$$w0rd'
  - registry: localhost:5000
    tls: disabled
```

- `creds` (optional, array of objects):
  The `creds` objects have the following fields:

  - `registry` (required, string, templates supported):
    Hostname and port of the registry server used in image references.
    Use `docker.io` for Docker Hub.

  - `hostname` (optional, string):
    This field is rarely used and will override using the registry name for the hostname.
    This can be used when configuring the same registry with multiple settings, such as different logins.
    There is a reduced efficiency when copying content between different registry names that go to the same backend registry.

  - `user` (optional, string, templates supported):
    Username.
    This is required for using a password.

  - `pass` (optional, string, templates supported):
    Password.
    Consider using a credential helper or a [template](#templates) to securely store credentials outside of the config file.
    Credentials will be automatically imported from Docker when available.

  - `credHelper` (optional, string):
    Name of a credential helper, typically in the form `docker-credential-name`.
    The alpine based docker image for `regbot` includes `docker-credential-ecr-login` and `docker-credential-gcr`.

  - `credExpire` (optional, duration):
    Duration to use a credential from a `credHelper`.
    This defaults to 1 hour.
    Use the [Go `time.Duration`](https://pkg.go.dev/time#ParseDuration) syntax when setting, e.g. `1h15m` or `30s`.

  - `tls` (optional, enum):
    Whether TLS is enabled/verified.
    Values include "enabled" (default), "insecure", or "disabled".

  - `regcert` (optional, string, templates supported):
    Registry CA certificate for self signed certificates.
    This may be a string with `\n` for line breaks, or the yaml multi-line syntax may be used like:

    ```yaml
    regcert: |
      -----BEGIN CERTIFICATE-----
      MIIJDDCCBPSgAwIB....
      -----END CERTIFICATE-----
    ```

  - `clientCert` (optional, string, templates supported):
    Client certificate used for mTLS authentication.
    Both `clientCert` and `clientKey` need to be defined for mTLS.
    See `regcert` for details of how to include this in yaml.

  - `clientKey` (optional, string, templates supported):
    Client key used for mTLS authentication.
    Both `clientCert` and `clientKey` need to be defined for mTLS.
    See `regcert` for details of how to include this in yaml.

  - `pathPrefix` (optional, string):
    Path added before all images pulled from this registry.
    This is useful for some mirror configurations that place images under a specific path.

  - `mirrors` (optional, array of strings):
    Array of registry names to use as a mirror for this registry.
    Mirrors are sorted by `priority`, highest first.
    This registry is sorted after any listed mirrors with the same priority.
    Mirrors are not used for push commands that change the registry, only for pulling content.

  - `priority` (optional, integer):
    Non-negative integer priority used for sorting mirrors.
    This defaults to 0.

  - `repoAuth` (optional, bool):
    Configures authentication requests per repository instead of for the registry.
    This is required for some registry providers, specifically `gcr.io`.
    This defaults to `false`.

  - `blobChunk` (optional, int in bytes):
    Chunk size for pushing blobs.
    Each chunk is a separate http request, incurring network overhead.
    The entire chunk is stored in memory, so chunks should be small enough not to exhaust RAM.

  - `blobMax` (optional, int in bytes):
    Blob size which skips the single put request in favor of the chunked upload.
    Note that a failed blob put will fall back to a chunked upload in most cases.
    Disable with -1 to always try a single put regardless of blob size.

  - `reqPerSec` (optional, float):
    Requests per second to throttle API calls to the registry.
    This may be a decimal like 0.5 to limit to one request every 2 seconds.
    Disable by leaving undefined or setting to 0.

  - `reqConcurrent` (optional, int):
    Number of concurrent requests that can be made to the registry.
    Disable by leaving undefined or setting to 0.

### Defaults

Various global options and default settings for `scripts` may be specified in `defaults`.

```yaml
defaults:
  parallel: 3
  interval: 60m
  timeout: 30m
```

- `defaults` (optional, object):
  Global settings and default values applied to each sync entry:

  - `parallel` (optional, int):
    Number of concurrent image copies to run.
    All sync steps may be started concurrently to check if a mirror is needed, but will wait on this limit when a copy is needed.
    Defaults to 1.

  - `skipDockerConfig` (optional, bool):
    Do not read the user credentials in `${HOME}/.docker/config.json`.
    Defaults to `true`.

  - `userAgent` (optional, string):
    Override the user-agent for http requests.
    Defaults to the regbot user agent, including a version.

  - `interval`, `schedule`, `timeout`:
    See description under `scripts`.

### Scripts

Each entry in `scripts` specifies a Lua script to run with a frequency.

```yaml
scripts:
  - name: Hello World
    frequency: 5m
    script: |
      log "hello world"
  - name: Busybox Tags
    timeout: 30s
    script: |
      tags = tag.ls("busybox")
      table.sort(tags)
      for k, t in ipairs(tags) do
        log "Found tag " .. t
      end
```

- `scripts` (required, array of objects):

  - `name` (required, string):
    Name of the script.
    This is used to reference the script, for logging, and should be unique.

  - `script` (required, string):
    Text of the Lua script to run.

  - `interval` (semi-required, duration)
    How often to run each sync step in `server` mode.
    Either `interval` or `schedule` is required, and `schedule` overrides `interval` if both are defined.

  - `schedule` (semi-required, string in cron syntax):
    Cron like schedule to run each step.
    E.g. `schedule: "15 01 * * *"` runs the sync at 1:15am.
    Either `interval` or `schedule` is required, and `schedule` overrides `interval` if both are defined.

  - `timeout` (optional, duration):
    Time until the script is aborted.
    This timeout is enforced when calling various actions like an image copy.

### User Extensions

Any field beginning with `x-` is considered a user extension and will not be parsed in current or future versions of the project.
These are useful for integrating your own tooling, or setting values for yaml anchors and aliases.

For example, the following configures a few scripts with different schedules.

```yaml
x-hourly: &hourly
  interval: 60m
  timeout: 10m
x-nightly: &nightly
  schedule: "5 2 * * *" # 2:05am
  timeout: 60m
scripts:
  - <<: *hourly
    name: Hourly hello 1
    script: |
      log "hello every hour 1"
  - <<: *hourly
    name: Hourly hello 2
    script: |
      log "hello every hour 2"
  - <<: *nightly
    name: Nightly hello 1
    script: |
      log "hello every night 1"
  - <<: *nightly
    name: Nightly hello 2
    script: |
      log "hello every night 2"
```

## Templates

[Go templates](https://golang.org/pkg/text/template/) are used to expand values in `registry`, `user`, `pass`, `regcert`, `clientCert`, and `clientKey`.

For `registry`, `user`, `pass`, `regcert`, `clientCert`, and `clientKey`, no values are provided under the `.` field.

See [Template Functions](/usage/#Template-Functions) for more details on the custom functions available in templates.

## Lua Extensions

The Lua script interface is based on Lua 5.1.
The [Lua manual is available online](https://www.lua.org/manual/5.1/index.html).
The following additional functions are available:

- `log <msg>`:
  Log a message (preferred over Lua's print).

- `reference.new <ref>`:
  Accepts an image reference, returning a reference object.
  Other functions that accept an image name or repository will accept a reference object.

- `<ref>:digest`:
  Get or set the digest on a reference.

- `<ref>:tag`:
  Get or set the tag on a reference.
  This is useful when iterating over tags within a repository.

- `repo.ls <host:port> [opts]`:
  List the repositories on a registry server.
  This depends on the registry supporting the API call.
  Opts is a table that can have the following values set:
  - `limit`: number of results to return
  - `last`: last received repo, next batch of results will start after this

  e.g. `list = repo.ls("example.com", {limit = 500})`

- `tag.ls <repo> [opts]`:
  Returns an array of tags found within a repository.
  Opts is a table that can have the following values set:
  - `limit`: number of results to return
  - `last`: last received tag, next batch of results will start after this

- `tag.delete <ref>`:
  Deletes a tag from a registry.
  This uses the regclient tag delete method that first pushes a dummy manifest to the tag, which avoids deleting other tags that point to the same manifest.

- `manifest.descriptor`:
  Returns the descriptor of a manifest. A descriptor contains fields like `digest`. Please check out [the documentation](https://pkg.go.dev/github.com/regclient/regclient/types/descriptor#Descriptor) for a full list of fields.

- `manifest.get`:
  Returns the image manifest.
  The current platform will be resolved, or it may be specified as a second arg.

- `manifest.getList`:
  Retrieves a manifest list without resolving the current platform.
  If the manifest is not a multi-platform manifest list, the single manifest will be returned instead.

- `manifest.head`:
  Retrieves the manifest using a head request.
  This pulls the digest and current rate limit and can be used with the manifest delete and ratelimit functions.

- `manifest.put <manifest> <ref>`:
  Pushes a manifest to the provided reference.

- `<manifest>:config`:
  See `image.config`

- `<manifest>:delete`:
  Deletes a manifest.
  Note that a manifest list or manifest head request to retrieve the manifest is recommended, otherwise the registry may delete a single platform's manifest without deleting the entire multi-platform image, leading to errors when attempting to access the remaining manifest.
  If multiple tags can point to the same manifest, then using `tag.delete` is recommended.

- `<manifest>:descriptor`:
  Returns the descriptor of a manifest. See `manifest.descriptor` for more details.

- `<manifest>:export`:
  Returns a new manifest created with user changes to the current manifest data (user changes are ignored by all other calls).

- `<manifest>:get`:
  See `image.manifest`.
  This is useful for pulling a manifest when you've only run a head request.

- `<manifest>:put <ref>`:
  See `manifest.put`

- `<manifest>:ratelimit`:
  Return the ratelimit seen when the manifest was last retrieved.
  The ratelimit object includes `Set` (boolean indicating if a rate limit was returned with the manifest), `Remain` (requests remaining), `Limit`
  (maximum limit possible).

- `<manifest>:ratelimitWait <limit> <poll> <timeout>`:
  See `image.ratelimitWait`

- `blob.get <ref> <optional digest>`:
  Retrieve a blob from the repository in the reference.
  If a separate digest is not provided, the reference must include a digest.

- `blob.head <ref> <optional digest>`:
  Same as `blob.get` but only performs a head request.

- `blob.put <ref> <content>`:
  Reference is used to lookup the repository where the blob is pushed.
  Content is a string, another blob, or a config object.
  The digest and size of the pushed blob are returned.

- `<blob>:put <content>`:
  See `blob.put`.

- `<config>:export`:
  Returns a new config created with user changes to the current config data (user changes are ignored by all other calls).

- `image.config <ref>`:
  Returns the image configuration, see `docker image inspect`.

- `image.copy <src-ref> <tgt-ref>`:
  Copies an image.
  This may be retagging within the same repository, copying between repositories, or copying between registries.
  There's an optional 3rd argument with a table of options:
  - `{digestTags = true}`: copies digest specific tags in addition to the manifests.
  - `{forceRecursive = true}`: forces a copy of all manifests and blobs even when the target parent manifest already exists.

- `image.exportTar <src-ref> <tar-filename>`:
  Exports an image from the registry to a tar file.

- `image.importTar <tgt-ref> <tar-filename>`:
  Imports an image from a tar file to the registry.

- `image.ratelimitWait <ref> <limit> <poll> <timeout>`:
  Polls a registry for the rate limit remaining to increase at or above the specified limit.
  By default the polling interval is `5m` and timeout is `6h`.

## Logging

By default, `regbot` outputs informational messages or greater from the logging interface.
Additional logging levels are available with the `-v` option.
Logs are structured (including `key=value` in addition to a message) and the output can be switched from text based to JSON with `--logopt json`.
Log output is always sent to stderr.

```console
regbot once --dry-run -c regbot-debug.yml -v trace --logopt json
```
