# docker-postgres

This repository holds Dockerfiles for building custom postgres container images.

## Prerequisites

### For Build

* Docker CE (Tested with: Docker version 20.10.17, build 100c701)
* Linux kernel >= 4.8 (or equivalent)
* OS-level `binfmt_misc` support
* OS-level QEMU support
* OS-level `binfmt-support` support

### For Push

* You have configured a remote repository to accept the image. By default this is Docker Hub.
* You are logged in to an account that is authorized to push to this remote via `docker login`

## Usage

In general this repo is used to build and push multi-arch images to Docker Hub. This is done by first building the images for target architectures, in this case `linux/arm64/v8` and `linux/amd64`, then pushing them to Docker Hub. After these images are pushed to Docker Hub the images can be pulled just like any other Docker image.

### Run Script

The `run` script in the root of the repository can be used to perform various Docker image management tasks. For a list of available commands run: `./run`.

We are currently using the `factalinc/postgres:11.16-buster` image by default in local environments. `factalinc/postgres:15.1-bookworm` is considered experimental at this time. Using the `*_15_bookworm` verison of the commands below works the same way as noted but builds PostgreSQL 15 container images instead.

#### buildx_11_buster

Builds multi-architecture images for PostgreSQL 11.16 on Debian 10 (Buster) using `docker buildx build`. You may pass any number of additional flags for `docker buildx build` to this command.

For reference, the built image is: `factalinc/postgres:11.16-buster`

```
# Build images.
# This does not load them into the local docker instance or push to remote registry.
./run buildx_11_buster
```

```
# Build images for `amd64` and `arm64` and push to Docker Hub registry
./run buildx_11_buster --push
```

#### build_11_buster

Builds a single image for PostgreSQL 11.16 on Debian 10 (Buster) using `docker buildx build`. This image matches the build system's architecture. You may pass any number of additional flags for `docker buildx build` to this command.

```
# Build the image and load it into the local docker runtime.
./run build_11_buster --load
```

#### remove_11_buster

Removes the factalinc/postgres:11.16-buster image from the local environment. 

```
./run remove_11_buster
```
