# Better base Docker images

[![Docker Build Status](https://img.shields.io/docker/build/better/baseimage.svg)](https://hub.docker.com/r/better/baseimage/)

## Stable images

* Alpine: `better/baseimage:alpine-3.10`
* Alpine: `better/baseimage:analytics-alpine-3.9`

## Alpine

The base Alpine image sets up the `app` user and installs several useful libraries, along with Python 3, the GCC toolchain, and RDS SSL certificates.

## Analytics

The analytics image builds on the Alpine image, adding Postgres and a number of useful Python packages (Numpy, Tensorflow, etc).

## Testing

You can verify that this works by building locally, e.g. `docker build . -f Dockerfile.analytics`.

## Publishing an image

The tag convention for Git and Docker is as follows: `[prefix]-[distro]-[version]`.

For the base image, there is no prefix, so a typical tag would be `alpine-1.0`.

When a tag (e.g. `alpine-3.7`) needs to be re-published, the following should be done:

* `git push origin :alpine-3.7` - this deletes the existing Git tag on remote
* `git tag -d alpine-3.7` - this deletes the existing Git tag on local
* `git tag alpine-3.7` - this tags master
* `git push origin master --tags` - this pushes the new tag to remote and kicks off Docker Hub build. Note that it can take several hours to finish.
