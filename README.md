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

The `run` script in the root of the repository can be used to perform various Docker image management tasks. For a list of available commands run: `./run`.

We are currently using the `factalinc/postgis:2.5-11.16-buster` image by default in local environments. `factalinc/postgis:3-15.3-bookworm` is considered experimental at this time.

### Building and deploying images to Docker Hub

As stated above make sure you are logged in to Docker Hub. We currently only have 3 seats available in our organization on Docker Hub so this functionality is only available to Ben, Ryan, and Brian.
```
docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: <your hub.docker.com username>
Password: 
Login Succeeded
```

Build multi-arch images. I recommend building without pushing first to make sure everything is working as expected. Images will be stored in your local build cache so subsequent build and push operations should be fairly fast. Docker also helpfully gives you a warning pointing out that you've built images but aren't currently doing anything with them.
```
./run buildx_all

# <...>
# A LOT of output
# <...>

WARNING: No output specified with docker-container driver. Build result will only remain in the build cache. To push result image into registry use --push or to load image into docker use --load
```

Assuming everything looks good with the build you can now go ahead and push the images to Docker Hub. This makes them available for pulling in other projects, such as https://github.com/factal-inc/web.
```
./run buildx_all --push ; echo $?
```

You can view the pushed images by logging in to Docker Hub and navigating to the repositories page: https://hub.docker.com/repositories/factalinc.

### Building and testing images locally

You shouldn't often need to do this but if you're trying to debug issues with images you can build and load specific images into your local docker environment without having to push them to Docker Hub first.

For example if you wanted to build and load the PostgreSQL 15 / PostGIS 3 image on your local you would do so as follows. Note that is using `build_15` **not** `buildx_15`.
```
./run build_15_bookworm_postgis --load
```

Assuming the build and load succeeds you can then start a container using this image. For example:
```
# Run a container named test-postgres. -d to daemonize and detach from the console.
docker run --name test-postgres -e POSTGRES_PASSWORD=mysecretpassword -d factalinc/postgis:3-15.3-bookworm

docker ps -a
CONTAINER ID   IMAGE                               COMMAND                  CREATED         STATUS                      PORTS      NAMES
f50b797b7937   factalinc/postgis:3-15.3-bookworm   "docker-entrypoint.s…"   2 seconds ago   Up 1 second                 5432/tcp   test-postgres
```

When you're finished testing whatever it is you needed to test you can remove the container as follows:
```
docker rm -f test-postgres
```

You can remove the image via the following run command
```
./run remove_15_bookworm_postgis
```

### Removing all images

If you want to quickly remove all images from your local environment you can use the following run command. `Error response from daemon: No such image` means there was no image to remove.
```
./run remove_all
```

## Other Notes

### PostgreSQL 15 on Debian 12 (Bookworm)

At the time of this writing (2023-07-13) `arm64` is not built by the official `postgres` image maintainers. They support some other ARM variants such as `v5` and `v7` but not `linux/arm64`. Because we're out on the bleeding edge you may see errors like this when trying to build the `factalinc/postgres:15.3-bookworm` image.

```
8.785 Package postgresql-15 is not available, but is referred to by another package.
8.785 This may mean that the package is missing, has been obsoleted, or
8.785 is only available from another source
8.785 
8.788 E: Version '15.3' for 'postgresql-15' was not found
```

This means the `PG_VERSION` specified in `15/bookworm/postgres/Dockerfile` is not available in the upstream package repositories. Most often this is because they have moved on to a newer version. To fix this problem we need to go find the current package version label.

* Go here: https://packages.debian.org/bookworm/database/
* Search for postgresql-15 (ctrl-f)
* Copy the package version label. This typically follows the package name. So if it says `postgresql-15 (15.3-0+deb12u1)` then the version label is `15.3-0+deb12u1`.
* Update `PG_VERSION` to match the current version label.
* Attempt your build again.

If there was a new minor version make sure to update the following.
* All references to 15.x in this repo should be updated.
  * This includes docker build tags in the `run` script.
  * Also make sure to update the `postgis` Dockerfile `FROM` directive.
* Build and push new images to Docker Hub.
* Update https://github.com/factal-inc/web to use the new image version.

### Intermittent image build failures

When images are built we're pulling down a ton of package data from upstream Debian package repos. This can very occasionally time out. If you see error messages implying that some Debian package server didn't answer try the build again.
