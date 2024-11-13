---
layout: page
title: Source Code Repositories
excerpt: "Source Code"
image:
  feature: header_1900x100.jpg
---

{% include _toc.html %}

All mod_cluster project modules and repositories are fully open-source and are licenced
under the [Apache License 2.0](https://www.apache.org/licenses/LICENSE-2.0).
All repositories are using [Git](https://git-scm.com/) for source code management.

## Container integration modules source code

* Written in Java
* Branches 1.3.x and older contain native httpd modules

[https://github.com/modcluster/mod_cluster](https://github.com/modcluster/mod_cluster)

## Proxy Implementations

### Apache HTTP Server (httpd) modules source code

* Written in C (APR) (2.x and newer)

[https://github.com/modcluster/mod_proxy_cluster](https://github.com/modcluster/mod_proxy_cluster)

* Older versions (1.3.x, 1.2.x, 1.1.x, and 1.0.x)

[https://github.com/modcluster/mod_cluster/tree/1.3.x](https://github.com/modcluster/mod_cluster/tree/1.3.x)

### Undertow

* Written in Java

[https://github.com/undertow-io/undertow](https://github.com/undertow-io/undertow)

## Other sources

* Documentation – contains AsciiDoc format documentation for the whole mod_cluster project hosted on GitHub

[https://github.com/modcluster/docs.modcluster.io](https://github.com/modcluster/docs.modcluster.io)

* CI Tooling – Apache HTTP Server `postinstall` scripts, build scripts and patches

[https://github.com/modcluster/ci.modcluster.io](https://github.com/modcluster/ci.modcluster.io)

* Website – Jekyll sources for the modcluster.io website

[https://github.com/modcluster/modcluster.io](https://github.com/modcluster/modcluster.io)
