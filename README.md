# docker-postgres

This repository holds Dockerfiles for building custom postgres container images.

## Usage

### Run Script

The `run` script in the root of the repository can be used to perform various Docker image management tasks. For a list of available commands run: `./run`.

#### Build

Builds a new Docker image from the specified Dockerfile.

```
# Simple usage
./run build_11_buster

# You may also pass additional flags which are in turn passed to docker build
./run build_11_buster --no-cache
```

#### Remove

Remove the specified Docker image from the local environment

```
./run remove_11_buster
```

#### Push

Push the specified Docker image to Docker Hub. This assumes the following:

* You have configured a remote repository to accept the image. By default this is Docker Hub.
* You are logged in to an account that is authorized to push to this remote via `docker login`

```
./run push_11_buster
```
