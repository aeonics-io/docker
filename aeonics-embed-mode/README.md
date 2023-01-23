# Docker image to run Aeonics in embed mode

## Why run in embed mode ?

You can package the Aeonics codebase inside the Docker image and ship that image
as a standalone container with no requirements at the host level (unlike in shared mode).

This means that you will need to update the image for each update of the codebase and/or
customize it further to persist the user snapshots, apps and other data.

## What do you need ?

In order to build the image in embed mode, you must place the Aeonics codebase in
the current directory of the Dockerfile.

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
    -p 80:80 \
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