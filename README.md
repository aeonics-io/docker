# How to deploy Aeonics on Docker in shared mode

Aeonics can handle multiple versions and different instance configuration from the same
installation directory and binaries. Therefore, it is not required to embed static 
assets in a docker image.
Instead, it is recommended to run Aeonics from a Docker directly against a mounted partition.
This means that you can use the same Docker image once and for all.

## 1) Prepare the Docker image (file: ./aeonics/Dockerfile) :
```
FROM debian
CMD /opt/aeonics/jre/bin/java $AEONICS_JAVA_OPTIONS -jar aeonics.jar
```
## 2) Build the image
```
/home/docker# docker build -t aeonics aeonics
```
```
Sending build context to Docker daemon  2.048kB
Step 1/2 : FROM debian
latest: Pulling from library/debian
Digest: sha256:6137c67e2009e881526386c42ba99b3657e4f92f546814a33d35b14e60579777
Status: Image is up to date for debian:latest
 ---> c4905f2a4f97
Step 2/2 : CMD /opt/aeonics/jre/bin/java $AEONICS_JAVA_OPTIONS -jar aeonics.jar
 ---> Running in c994c3dd7a3b
Removing intermediate container c994c3dd7a3b
 ---> d252e62cb53a
Successfully built d252e62cb53a
Successfully tagged aeonics:latest
```
## 3) Run the image using instance environment parameters
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
The different `-e` directive set specific environment parameters for this Docker instance. 
You can thus run multiple instances in parallel by changing those values.

The `-p` option maps the web server port on the host. Once again, adjust this if multiple instances are running at the same time.

The `--mount` option makes the shared repository available to the Docker instance. Other instances will see modifications of the shared files.

## 4) Check that instances are running
```
/home/docker# docker ps
```
```
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS              PORTS                               NAMES
4ce6e7ea2363   ae        "/bin/sh -c '/opt/ae…"   2 minutes ago   Up About a minute   0.0.0.0:81->80/tcp, :::81->80/tcp   recursing_golick
35d5606b6b82   ae        "/bin/sh -c '/opt/ae…"   2 minutes ago   Up 2 minutes        0.0.0.0:80->80/tcp, :::80->80/tcp   wonderful_volhard
```
