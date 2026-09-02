---
title: regctl image mod
layout: single
warning: Auto generated content
---

## Synopsis

EXPERIMENTAL: Applies requested modifications to an image

  For time options, the value is a comma separated list of key/value pairs:
  set=${time}: time to set in rfc3339 format, e.g. 2006-01-02T15:04:05Z
  from-label=${label}: label used to extract time in rfc3339 format
  after=${time_in_rfc3339}: adjust any time after this
  base-ref=${image}: image to lookup base layers, which are skipped
  base-layers=${count}: number of layers to skip changing (from the base image)
  Note: set or from-label is required in the time options

```shell
regctl image mod <image_ref> [flags]
```

## Examples

```shell
# add an annotation to all images, replacing the v1 tag with the new image
regctl image mod registry.example.org/repo:v1 \
  --replace --annotation "[*]org.opencontainers.image.created=2021-02-03T05:06:07Z"

# convert an image to the OCI media types, copying to local registry
regctl image mod alpine:3.5 --to-oci --create registry.example.org/alpine:3.5

# append a layer to only the linux/amd64 image using the file.tar contents
regctl image mod registry.example.org/repo:v1 --create v1-extended \
  --layer-add "tar=file.tar,platform=linux/amd64"

# append a layer to all platforms using the contents of a directory
regctl image mod registry.example.org/repo:v1 --create v1-extended \
  --layer-add "dir=path/to/directory"

# set the timestamp on the config and layers, ignoring the alpine base image layers
regctl image mod registry.example.org/repo:v1 --create v1-time \
  --time "set=2021-02-03T04:05:06Z,base-ref=alpine:3"

# set the entrypoint to be bash and unset the default command
regctl image mod registry.example.org/repo:v1 --create v1-bash \
  --config-entrypoint '["bash"]' --config-cmd ""

# delete an environment variable from only the linux/arm64 image
regctl image mod registry.example.org/repo:v1 --create v1-env \
  --env "[linux/arm64]LD_PRELOAD="

# Rebase an older regctl image, copying to the local registry.
# This uses annotations that were included in the original image build.
regctl image mod registry.example.org/regctl:v0.5.1-alpine \
  --rebase --create v0.5.1-alpine-rebase
```

## Options

```text
      --annotation stringArray        set an annotation (name=value, omit value to delete, prefix with platform list [p1,p2] or [*] for all images)
      --annotation-base stringArray   set base image annotations (image/name:tag,sha256:digest)
      --annotation-promote            promote common annotations from child images to index
      --author string                 set image author
      --buildarg-rm string            delete a build arg
      --buildarg-rm-regex string      delete a build arg with a regex value
      --config-cmd string             set command in the config (json array or string, empty string to delete)
      --config-entrypoint string      set entrypoint in the config (json array or string, empty string to delete)
      --config-platform string        set platform on the config (not recommended for an index of multiple images)
      --config-time string            set timestamp for the config
      --create string                 Create image or tag
      --data-max stringArray          sets or removes descriptor data field (size in bytes)
      --digest-algo stringArray       change the digest algorithm (sha256, sha512)
      --env stringArray               set an environment variable (name=value, omit value to delete, prefix with platform list [p1,p2] for subset of images)
      --expose-add stringArray        add an exposed port
      --expose-rm stringArray         delete an exposed port
      --external-urls-rm              remove external url references from layers (first copy image with "--include-external")
      --file-tar-time stringArray     timestamp for contents of a tar file within a layer, set filename=${name} with time options
      --label stringArray             set an label (name=value, omit value to delete, prefix with platform list [p1,p2] for subset of images)
      --label-to-annotation           set annotations from labels
      --layer-add string              add a new layer (tar=file,dir=directory,platform=val)
      --layer-compress string         change layer compression (gzip, none, zstd)
      --layer-rm-created-by string    delete a layer based on history (created by string is a regex)
      --layer-rm-index uint           delete a layer from an image (index begins at 0)
      --layer-strip-file string       delete a file or directory from all layers
      --layer-time string             set timestamp for the layer contents
      --rebase                        rebase an image using OCI annotations
      --rebase-ref string             rebase an image with base references (base:old,base:new)
      --replace                       Replace tag (ignored when "create" is used)
      --reproducible                  fix tar headers for reproducibility
      --time string                   set timestamp for both the config and layers
      --to-docker                     convert to Docker schema2 media types
      --to-oci                        convert to OCI media types
      --to-oci-referrers              convert to OCI referrers
      --user string                   set default user (username, uid, with optional colon and group/gid value)
      --volume-add stringArray        add a volume definition
      --volume-rm stringArray         delete a volume definition
      --workdir string                set default workdir path
```

## Options from parent commands

```text
      --host stringArray     Registry hosts to add (reg=registry,user=username,pass=password,tls=enabled)
      --logopt stringArray   Log options
      --user-agent string    Override user agent
  -v, --verbosity string     Log level (trace, debug, info, warn, error) (default "WARN")
```
