# Docker Image Layers

## Overview

A Docker image is just a bunch of loosely-connected read-only layers, with each layer comprising one or more files.

![Docker Image Layers Overview](./layers.png)

Docker takes care of stacking these layers and representing them as a single unified object.

## Inspecting Layers

### Docker Pull

There are a few ways to see and inspect the layers that make up an image. The following example looks closer at an image pull operation:

```bash
$ docker pull postgres

Using default tag: latest
latest: Pulling from library/postgres
dd6189d6fc13: Already exists
e83a243abe4a: Pull complete
e1e5d0e9701b: Pull complete
b8172b349685: Pull complete
83dad09f3014: Pull complete
c4849d0ca437: Pull complete
7642d443be37: Pull complete
a40f15729374: Pull complete
4ce9b636fb02: Pull complete
8775e8553947: Pull complete
c54bf09096bd: Pull complete
fd9206df829a: Pull complete
756563c6c3db: Pull complete
Digest: sha256:9eb2589e67e69daf321fa95ae40e7509ce08bb1ef90d5a27a0775aa88ee0c704
Status: Downloaded newer image for postgres:latest
docker.io/library/postgres:latest
```

### Docker Inspect

Another way to see the layers of an image is to inspect the image with the `docker image inspect` command. The following example inspects the same `postgres:latest` image.


```bash
$ docker image inspect postgres:latest

[
    {
        "Id": "sha256:d470761f9551c5e358f72a6bc562d76c6297d2b86de05337b4e1c9701f6d491c",
        "RepoTags": [
            "postgres:latest"
        ],
        "RepoDigests": [
            "postgres@sha256:9eb2589e67e69daf321fa95ae40e7509ce08bb1ef90d5a27a0775aa88ee0c704"
        ],

        <Snip>

        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:696831e2d730dba43f407bbf960b5165be6763cb9cff41fad295c71369160ad5",
                "sha256:ae8a0bbf3018195ab7b1aa791aad11756979d381e26ce49870444cb7edead01c",
                "sha256:ce277a5902d590dc0f86a50944bbebf272665f7cc5ea311f4f359ba51f41b895",
                "sha256:d77e26f694ed59dbd52a24b11db5dbb6759e56ca5c1a431df2a6d37727f1a08b",
                "sha256:fd10e3aa5db5f3c1debdfab6609bc5c9fe452ac9d4dbca57b55910b584c06dd8",
                "sha256:c0efa9ab11e5ad37536e3f91898b5d7e4cd232ed405ab986e3a5a6a177f2f912",
                "sha256:8733f9251f962ce89c4fce2c0dc1e28cb6c40ff92a9762e431d505eaafb2f5a0",
                "sha256:72001cc52c2f60e77e579b39e98b12f26274375f36b6ad566d75452205bb6bc0",
                "sha256:88caabcf3b83a37c947cf8cd9cabf43b0a0957d78cb76093ca3714131f53eafa",
                "sha256:4ce75c8a891e4b904faca48cb458c1bc319dd6912e22575e2df5b8f42d289afa",
                "sha256:9a3eb3459b84113a6f9cfd31b3a196110ec81b585cb4b1d7f8cc2c702b39a784",
                "sha256:6882c254374a962d234fc6bd3a528188e09e9c5626596a364a179afeb7d29e4c",
                "sha256:cc152a8144da69847d8e8f5ab47c68790718266c2afe44c4a506b07b167b5139"
            ]
        },
    }
]
```

The trimmed output shows 12 layers again. Only this time they’re shown using their SHA256 hashes.

The `docker image inspect` command is a great way to see the details of an image.

### Docker History

The `docker history` command is another way of inspecting an image and seeing layer data. However, it shows the build history of an image and is **not** a strict list of layers in the final image. For example, some Dockerfile instructions (“ENV”, “EXPOSE”, “CMD”, and “ENTRYPOINT”) add metadata to the image and do not result in permanent layers being created.

For example, the `postgres` image shows the following when running the `docker history` command:

```bash
$ docker history postgres

IMAGE          CREATED       CREATED BY                                      SIZE      COMMENT
d470761f9551   3 days ago    /bin/sh -c #(nop)  CMD ["postgres"]             0B
<missing>      3 days ago    /bin/sh -c #(nop)  EXPOSE 5432                  0B
<missing>      3 days ago    /bin/sh -c #(nop)  STOPSIGNAL SIGINT            0B
<missing>      3 days ago    /bin/sh -c #(nop)  ENTRYPOINT ["docker-entry…   0B
<missing>      3 days ago    /bin/sh -c #(nop) COPY file:925d466681c8349f…   12.1kB
<missing>      3 days ago    /bin/sh -c #(nop)  VOLUME [/var/lib/postgres…   0B
<missing>      3 days ago    /bin/sh -c mkdir -p "$PGDATA" && chown -R po…   0B
<missing>      3 days ago    /bin/sh -c #(nop)  ENV PGDATA=/var/lib/postg…   0B
<missing>      3 days ago    /bin/sh -c mkdir -p /var/run/postgresql && c…   0B
<missing>      3 days ago    /bin/sh -c set -eux;  dpkg-divert --add --re…   59.4kB
<missing>      3 days ago    /bin/sh -c set -ex;   export PYTHONDONTWRITE…   242MB
<missing>      3 days ago    /bin/sh -c #(nop)  ENV PG_VERSION=15.1-1.pgd…   0B
<missing>      2 weeks ago   /bin/sh -c #(nop)  ENV PATH=/usr/local/sbin:…   0B
<missing>      2 weeks ago   /bin/sh -c #(nop)  ENV PG_MAJOR=15              0B
<missing>      2 weeks ago   /bin/sh -c set -ex;  key='B97B0AFCAA1A47F044…   3.98kB
<missing>      2 weeks ago   /bin/sh -c mkdir /docker-entrypoint-initdb.d    0B
<missing>      2 weeks ago   /bin/sh -c set -eux;  apt-get update;  apt-g…   3.23MB
<missing>      2 weeks ago   /bin/sh -c #(nop)  ENV LANG=en_US.utf8          0B
<missing>      2 weeks ago   /bin/sh -c set -eux;  if [ -f /etc/dpkg/dpkg…   25.1MB
<missing>      2 weeks ago   /bin/sh -c set -eux;  savedAptMark="$(apt-ma…   4.1MB
<missing>      2 weeks ago   /bin/sh -c #(nop)  ENV GOSU_VERSION=1.14        0B
<missing>      2 weeks ago   /bin/sh -c set -eux;  groupadd -r postgres -…   333kB
<missing>      2 weeks ago   /bin/sh -c set -ex;  if ! command -v gpg > /…   9.81MB
<missing>      2 weeks ago   /bin/sh -c #(nop)  CMD ["bash"]                 0B
<missing>      2 weeks ago   /bin/sh -c #(nop) ADD file:d3de9d62792244640…   74.3MB
```
