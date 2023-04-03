---
title: How to build podman images that work on both Mac, Linux, and PI
date: 2023-03-31 12:00:00
tags:
    - posts
layout: layouts/post.njk
---

Oh hack-day March 23 2023 Michael Clayton and I finished building a Mimir Image Builder
POC which we had started earlier in the week.

## How to build podman images for Mac and Linux

In order to build container images that will run on both mac and linux
you need to use the podman build flags `--platform` and `--manifest`

Example build:

    podman build --platform linux/amd64,linux/arm64 --manifest httpd-solr -f components/load/podman/Containerfile.httpd-solr .

Example push:

    podman manifest push httpd-solr quay.io/offline/httpd-solr

You need to do this for both your base image, and your final image. The manifest builds a separate image for both platforms. And side benifit of using `--platform linx/amd64,linux/arm64` is that you get a build that works on Mac, Linux, _and_ Rasperry PI!

NOTE! `qemu-user-static` is a dependancy for this multi-platform build to work. Fortunately `podman` comes bundled with this for Mac. But on you compiling linux host it needs to be installed with `sudo dnf install qemu-user-static`, and RHEL intentianly does _not_ offer it as a package. We had to build using Fedora 37.
