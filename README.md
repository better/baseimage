# Better base Docker images

[![Docker Build Status](https://img.shields.io/docker/build/better/baseimage.svg)](https://hub.docker.com/r/better/baseimage/)

## Stable images

* Alpine: `better/baseimage:alpine-3.7`
* Alpine: `better/baseimage:alpine-3.5`
* Alpine: `better/baseimage:alpine-3.5-analytics`

## Alpine

The base Alpine image sets up the `app` user and installs several useful libraries, along with Python 3, the GCC toolchain, and RDS SSL certificates.

## Analytics

The analytics image builds on the Alpine image, adding Postgres and Python data science packages.
