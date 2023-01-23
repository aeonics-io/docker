# Docker image to run Aeonics in shared mode

## Why run in shared mode ?

You can use the same Docker image to run multiple instances of the same codebase.
By mounting the host drive (or a NAS) then you have a single central point for all your
apps and shapshots. If you need to hot-deploy or update some components, it is directly applied to
all running instances. No need to rebuild a new image, no need to redeploy or restart your containers.

If you need the same codebase but different configuration, you can easily point to another snapshot using
environment parameters. Different tenants can thus store their config and apps in a central backuped location
that can be spawned by just running a new instance of the same Docker image.

```
/home/docker# docker run -e AEONICS_SNAPSHOT_STORAGE=tenant_1 ...
/home/docker# docker run -e AEONICS_SNAPSHOT_STORAGE=tenant_2 ...
```

## What do you need ?

In order to run in shared mode, you need to have the Aeonics codebase available on the host (or a NAS) so that
it can be mounted at runtime for the container.

```
/home/docker# docker run --mount type=bind,source=/opt/aeonics,target=/opt/aeonics,bind-propagation=rshared ...
```

## How to build the image

```
/home/docker# docker build -t aeonics .
```

## How to run the container

```
/home/docker# docker run \
    -e AEONICS_JAVA_OPTIONS=-Xmx1g \
    -e AEONICS_LICENSE_STORE_PATH=/opt/aeonics/aeonics.license \
    -e AEONICS_LICENSE_STORE_PASS=******* \
    -e AEONICS_SNAPSHOT_STORAGE=snapshots \
    -p 80:80 \
    --mount type=bind,source=/opt/aeonics,target=/opt/aeonics,bind-propagation=rshared \
    -w /opt/aeonics \
    -u 0 \
    aeonics
```
## Troubleshooting

When running Docker images on a server or on a Kubernetes cluster, the ranom number generator (PRNG) of the operating system may not provide enough entropy and the system may
block or be very slow. There are different alternatives to fix this issue:
- install another entropy source
```
apt install haveged
```
```
yum install haveged
```
- change the way java uses the entropy to use `urandom` instead
```
sed -i.bak \
  -e "s/securerandom.source=file:\/dev\/random/securerandom.source=file:\/dev\/urandom/g" \
  -e "s/securerandom.strongAlgorithms=NativePRNGBlocking/securerandom.strongAlgorithms=NativePRNG/g" \
  $JAVA_HOME/lib/security/java.security
```
- map the operating system PRNG to the Docker one
```
docker run -v /dev/random:/dev/random
```