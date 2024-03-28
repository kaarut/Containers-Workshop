# Open Container Initiative (OCI)

## Overview

The [Open Container Initiative (OCI)](https://www.opencontainers.org/) is a governance council responsible for standardizing the low-level fundamental components of container infrastructure. In particular it focusses on _image format_ and _container runtime_.

## History

From day one, use of Docker grew like crazy. More and more people used it in more and more ways for more and more things. So, it was inevitable that some parties would get frustrated. This is normal and healthy.

The TLDR of this history is that a company called CoreOS (acquired by Red Hat) didn’t like the way Docker did certain things. So, they created an open standard called [appc](https://github.com/appc/spec/) that defined things like image format and container runtime. They also created an implementation of the spec called **rkt** (pronounced “rocket”).

This put the container ecosystem in an awkward position with two competing standards.

Getting back to the story, this threatened to fracture the ecosystem and present users and customers with a dilemma. While competition is usually a good thing, _competing standards_ is usually not. They cause confusion and slowdown user adoption. Not good for anybody.

With this in mind, OCI was formed — a lightweight agile council to govern container standards.

## OCI Specifications

At the time of writing, the OCI has published two specifications (standards):

- The [image-spec](https://github.com/opencontainers/image-spec)
- The [runtime-spec](https://github.com/opencontainers/runtime-spec)

The two OCI specifications have had a major impact on the architecture and design of the core Docker product. As of Docker 1.11, the Docker Engine architecture conforms to the OCI runtime spec.

The OCI is organized under the auspices of the Linux Foundation.
