# Local development

The add-on image is built entirely from `pgadmin4/Dockerfile`: the pgAdmin sources are cloned
at the tag given by the `PGADMIN_VERSION` build argument (default at the top of the Dockerfile),
the JS bundle and the Python virtualenv are built in separate stages, and the PostgreSQL
client tools come from the Alpine packages. No git submodule is needed.

## Build with docker

Switch to the `pgadmin4` folder and run:

```
./build.sh --version dev
```

which builds and pushes `ghcr.io/expaso/pgadmin4/<arch>:dev` for the architectures in
`config.yaml`, using the base images from `build.yaml`. Set `REGISTRY` to push to your own
registry, e.g. when working on a fork:

```
REGISTRY=ghcr.io/<your-user>/pgadmin4 ./build.sh --version dev
```

A single architecture can also be built directly:

```
docker buildx build --platform linux/amd64 \
    --build-arg BUILD_FROM=ghcr.io/hassio-addons/base/amd64:20.2.0 \
    --build-arg BUILD_ARCH=amd64 \
    --tag ghcr.io/expaso/pgadmin4/amd64:dev .
```

## Build with GitHub Actions

The `Build add-on (dev)` workflow (`workflow_dispatch`) builds the image for the selected
architectures and pushes it to `ghcr.io/<repository owner>/pgadmin4/<arch>:<tag>`. This also works
from a fork, which makes it easy to test a branch on a Home Assistant machine: create a local
add-on folder in `/addons` with a `config.yaml` whose `image` points at that registry.

## Upgrading pgAdmin

Bump `PGADMIN_VERSION` in `pgadmin4/Dockerfile` (and the version header in `.README.j2`).
Check the release notes for changes in the supported PostgreSQL versions; the bundled client
tools are the Alpine `postgresql<major>-client` packages listed in the Dockerfile.

## Run the add-on with an interactive shell

From a system SSH (port 22222), run the docker container with data attached:

```
docker run -it --entrypoint "/bin/sh" -v /mnt/data/supervisor/addons/data/local_pgadmin4/:/data:rw  ghcr.io/expaso/pgadmin4/aarch64:dev
```
