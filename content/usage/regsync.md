---
title: regsync
layout: single
date: 2025-01-01
---

{{% toc %}}

`regsync` is an image mirroring tool, copying images between two locations.
See the [Image References](/usage/#image-references) documentation for details on how to reference images in a registry or on the local filesystem with an OCI Layout.

## Operating Modes

`regsync` has various operating modes based on the CLI and flags:

- `regsync server`: Deploys a persistent server syncing images based on the requested schedule.
  An initial pass is performed to copy any missing tags before waiting for the next scheduled cycle to verify all image digests.
- `regsync once`: Synchronizes all images a single time and exits.
- `regsync once --missing`: Synchronizes only the missing tags from the target and exits.
  This is useful if you have immutable tags, or if you do not care about stale content.
- `regsync check`: Reports if any images need to be synchronized, but does not copy any content.

## Configuration File

An example that copies a few images from to your local registry looks like:

```yaml
version: 1 # optional for v1 files, there is no v2 yet
creds:
  # credentials are only needed for logins that are not stored in the docker config
  - registry: registry.example.org
    user: syncuser
    pass: 'Pa$$w0rd' # typically you would store secrets outside of this file
  - registry: docker.io
    user: youruser
    pass: "{{file \"/var/run/secrets/hub_token\"}}"
defaults:
  interval: 60m # all sync checks run hourly
sync:
  - source: busybox:latest
    target: registry.example.org/library/busybox:latest
    type: image
  - source: alpine
    target: registry.example.org/library/alpine
    type: repository
    tagSets: # tag sets allow multiple criteria 
      - allow: # first set of tags is a fixed list of tags
        - "latest"
        - "edge"
      - semverRange: # second set of tags is only patch tags in a specific semver range
        - ">=3.5.1 <4.0"
        allow:
        - 'v\d+\.\d+\.\d+'
  - source: ghcr.io/regclient/regctl
    target: registry.example.org/regclient/regctl
    type: repository
    tags:
      allow:
      - "latest"
      - "edge"
      - "v[0-9\\.]+"
    referrers: true
    digestTags: true
    fastCopy: true
```

### Version

```yaml
version: 1
```

This should be left at version 1 or not included at all.
This may be incremented if future `regsync` releases change the configuration file structure.

### Registry Credentials and Settings

`regsync` will attempt to import credentials from `${HOME}/.docker/config.json` by default.
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
    The alpine based docker image for `regsync` includes `docker-credential-ecr-login` and `docker-credential-gcr`.

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

Various global options and default settings for `sync` may be specified in `defaults`.

```yaml
defaults:
  ratelimit:
    min: 20
    retry: 15m
  parallel: 3
  interval: 60m
```

- `defaults` (optional, object):
  Global settings and default values applied to each sync entry:

  - `parallel` (optional, int):
    Number of concurrent image copies to run.
    All sync steps may be started concurrently to check if a mirror is needed, but will wait on this limit when a copy is needed.
    Defaults to 1.

  - `cacheCount` (optional, int):
    Number of items to cache for various registry API requests, per item type.
    `cacheTime` must also be set for this to apply.

  - `cacheTime` (optional, duration):
    Duration for items to remain in the cache for various registry API requests.
    `cacheCount` must also be set for this to apply.

  - `skipDockerConfig` (optional, bool):
    Do not read the user credentials in `${HOME}/.docker/config.json`.
    Defaults to `true`.

  - `userAgent` (optional, string):
    Override the user-agent for http requests.
    Defaults to the regsync user agent, including a version.

  - `backup`, `interval`, `schedule`, `ratelimit`, `digestTags`, `referrers`, `referrerFilters`, `referrerSource`, `referrerTarget`, `fastCopy`, `forceRecursive`, and `mediaTypes`:
    See description under `sync`.

### Sync

Each entry in `sync` specifies a copy request with a source, target, any inclusions and exclusions, and a frequency.
Each entry can be for a single image, a repository, or an entire registry, based on the `type`.

```yaml
sync:
  - source: busybox:latest
    target: registry.example.org/library/busybox:latest
    type: image
    schedule: "15 01 * * *"
  - source: ghcr.io/regclient/regctl
    target: registry.example.org/regclient/regctl
    type: repository
    tags:
      allow:
      - "latest"
      - "edge"
      - "v[0-9\\.]+"
    referrers: true
    schedule: "30 01 * * *"
```

- `sync` (required, array of objects):

  - `source` (required, string, templates supported):
    Source registry, repository, or image.

  - `target` (required, string, templates supported):
    Target registry, repository, or image.

  - `type` (required, enum):
    "registry", "repository", or "image".
    "registry" expects a registry name (host:port) and will copy multiple repositories (note, this requires the registry to support the `_catalog` API).
    "repository" will copy multiple images from the source repository.

  - `interval` (semi-required, duration):
    How often to run each sync step in `server` mode.
    Either `interval` or `schedule` is required, and `schedule` overrides `interval` if both are defined.

  - `schedule` (semi-required, string in cron syntax):
    Cron like schedule to run each step.
    E.g. `schedule: "15 01 * * *"` runs the sync at 1:15am.
    Either `interval` or `schedule` is required, and `schedule` overrides `interval` if both are defined.

  - `backup` (optional, string, templates supported):
    Tag or image reference for backing up target image before overwriting.
    This may include a Go template syntax.
    This backup is only run when the source changes and the target exists that is about to be overwritten.
    If the backup tag already exists, it will be overwritten.

  - `digestTags` (optional, bool):
    Copies digest specific tags in addition to the manifests.
    Defaults to `false`.

  - `fastCopy` (optional, bool):
    Skip referrers and digest tag checks when image exists, overrides `forceRecursive`.

  - `forceRecursive` (optional, bool):
    Forces a copy of all manifests and blobs even when the target parent manifest already exists.

  - `mediaTypes` (optional, array of strings):
    Media types to include.
    These must also be supported by regclient.
    Defaults to: `["application/vnd.docker.distribution.manifest.v2+json", "application/vnd.docker.distribution.manifest.list.v2+json", "application/vnd.oci.image.manifest.v1+json", "application/vnd.oci.image.index.v1+json"]`

  - `platform` (optional, string):
    Single platform to pull from a multi-platform image, e.g. `linux/amd64`.
    By default all platforms are copied along with the original upstream manifest list.
    Note that looking up the platform from a multi-platform image counts against the Docker Hub rate limit, and that rate limits are not checked prior to resolving the platform.
    When run in "server" mode, multi-platform manifests are cached.

  - `ratelimit` (optional, object):
    Settings to throttle based on source rate limits.

    - `min` (optional, int):
      Minimum number of pulls remaining to start the step.
      Actions while running the step can result in going below this limit.
      Note that parallel steps and multi-platform images may each result in more than one pull happening beyond this threshold.

    - `retry` (optional, duration):
      How long to wait before checking if the rate limit has increased.

  - `referrers` (optional, bool):
    Copies referrers in addition to the selected manifests.
    Defaults to `false`.

  - `referrerFilters` (optional, array of objects):
    List of filters for referrers to include.
    Defaults to include all referrers.

    - `artifactType` (optional, string):
      Artifact types to include.

    - `annotations` (optional, map of string to string):
      Mapping of annotations for referrers.

  - `referrerSource` (optional, string, templates supported):
    Source repo for pulling referrers.
    Defaults to pulling referrers from the image source repository.

  - `referrerTarget` (optional, string, templates supported):
    Target repo for pushing referrers.
    Defaults to pushing referrers to the image target repository.

  - `repos` (optional, object):
    Implements filters on repositories for "registry" types.
    Regex values are automatically bound to the beginning and ending of each string (`^` and `$`).

    - `allow` (optional, array of strings):
      Regex to allow specific repositories.

    - `deny` (optional, array of strings):
      Regex to deny specific repositories.

  - `tags` (optional, object):
    Implements filters on tags for "registry" and "repository" types.
    Regex values are automatically bound to the beginning and ending of each string (`^` and `$`).
    A tag much match each criteria to be included, so a rule to `allow: ["latest"]` and `deny: ["dev"]` would not match any tags.
    See `tagSets` for defining multiple sets of tag criteria.

    - `allow` (optional, array of strings):
      Regex to allow specific tags.
      Multiple regex strings are combine with OR logic.

    - `deny` (optional, array of strings):
      Regex to deny specific tags.
      Multiple regex strings are combine with OR logic.

    - `semverRange` (optional, array of strings):
      Semantic version constraints to filter by version ranges.
      Space separated ranges within the same string are combine with AND logic.
      Separate range strings are combine with OR logic.
      See [Semantic Version Filtering](#semantic-version-filtering-for-tags) for syntax details.

  - `tagSets` (optional, array of objects):
    Implements filters on tags for "registry" and "repository" types.
    This is an array of `tags` entries, where each entry is combine with OR logic.
    This allows `allow` and `deny` to include fixed names and also `semverRange` to be used for a version range.

### User Extensions

Any field beginning with `x-` is considered a user extension and will not be parsed in current or future versions of the project.
These are useful for integrating your own tooling, or setting values for yaml anchors and aliases.

For example, the following configures a few sync steps with the same set to tags to copy, and another set of steps with the same backup naming scheme.

```yaml
x-backup: &backup "backup-{{.Ref.Tag}}"
x-std-repo: &std-repo
  type: repository
  tags:
    allow:
    - "latest"
    - "edge"
    - "v[0-9\\.]+"
sync:
  - <<: *std-repo
    source: upstream.example.com/repo-a
    target: registry.example.org/repo-a
  - <<: *std-repo
    source: upstream.example.com/repo-b
    target: registry.example.org/repo-b
    backup: *backup
  - source: upstream.example.com/repo-c:latest
    target: registry.example.org/repo-c:latest
    type: image
    backup: *backup
```

## Semantic Version Filtering for Tags

Semver filtering on tags compares a dot separated version numbers and the optional prerelease data.

- Operators are included in front of the version number.
- A leading `v` on the version number is permitted.
- Values after a `-` are treated as prereleases, e.g. `v1.2.3-alpha` or `v1.2.3-rc4`.
  Prerelease comparisons would also include other content after a `-`, e.g. `v1.2.3-alpine` or `v1.2.3-dev`, so additional `allow` or `deny` filters may be needed to limit matched tags.
- Multiple constraints on the version are specified with a space separator to define a range.
  For example, `>=1.2.3 <2.0.0` specifies every version starting at `v1.2.3` but less than `v2.0.0`.
- Any value that cannot be parsed as a version number is excluded from the filtered tags.

Operators:

| Syntax    | Description                                                                                                 |
| --------: | ----------------------------------------------------------------------------------------------------------- |
|  `=1.2.3` | Exactly version 1.2.3                                                                                       |
|   `1.2.3` | No operator is treated as equal, equivalent to `=1.2.3`                                                     |
|  `>1.2.3` | Greater than 1.2.3                                                                                          |
|  `<1.2.3` | Less than 1.2.3                                                                                             |
| `>=1.2.3` | Greater than or equal to 1.2.3                                                                              |
| `<=1.2.3` | Less than or equal to 1.2.3                                                                                 |
|  `~1.2.3` | Patch releases up to the next minor release, equivalent to `>=1.2.3 <1.3.0`                                 |
|  `^1.2.3` | Major releases, equivalent to `>=1.2.3 <2.0.0`                                                              |
|  `^0.2.3` | Major releases with a `0.x` version are limited to the next minor release, equivalent to `>=0.2.3 <0.3.0`   |
|  `^0.0.3` | Major releases with a `0.0.x` version are limited to the next patch release, equivalent to `>=0.0.3 <0.0.4` |

## Templates

[Go templates](https://golang.org/pkg/text/template/) are used to expand values in `registry`, `user`, `pass`, `regcert`, `clientCert`, `clientKey`, `source`, `target`, `referrerSource`, `referrerTarget`, and `backup`.

For `registry`, `user`, `pass`, `regcert`, `clientCert`, and `clientKey`, no values are provided under the `.` field.

The `source`, `target`, `referrerSource`, `referrerTarget`, `backup` templates support the following objects:

- `.Sync`: Values from the current sync step, including
  - `.Sync.Source`: Source
  - `.Sync.Target`: Target
  - `.Sync.Type`: Type
  - `.Sync.Backup`: Backup
  - `.Sync.Interval`: Interval
  - `.Sync.Schedule`: Schedule

The `backup` template supports the following objects:

- `.Ref`: Reference object about to be overwritten
  - `.Ref.Reference`: Full reference
  - `.Ref.Registry`: Registry name
  - `.Ref.Repository`: Repository
  - `.Ref.Tag`: Tag
- `.Sync`: Values from the current sync step listed above

Note that templates are expanded in the order `source`, `referrerSource`, `target`, `referrerTarget`, and then `backup`.
Using a value before it has been expanded will return the template string.

See [Template Functions](/usage/#Template-Functions) for more details on the custom functions available in templates.

## Logging

By default, `regsync` outputs informational messages or greater from the logging interface.
Additional logging levels are available with the `-v` option.
Logs are structured (including `key=value` in addition to a message) and the output can be switched from text based to JSON with `--logopt json`.
Log output is always sent to stderr.

```console
regsync check -c regsync-debug.yml -v trace --logopt json
```
