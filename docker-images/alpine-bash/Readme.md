
Build the image and give it a name (default tag is latest):

```
docker build -t "mandric/alpine-bash" .
```

Or specify a tag like:

```
docker build -t "mandric/alpine-bash:tshark" .
```

Run an interactive bash shell in a temporary container with the new image:

```
docker run -ti --rm mandric/alpine-bash bash
```

Mount a local directory and run tests in the container:

```
docker run --rm \
  -v "$PWD:/opt/project" \
  mandric/alpine-bash \
  /bin/sh -c 'cd /opt/project; make'
```

Push to docker hub:

```
docker login --username=mandric
docker push mandric/alpine-bash
docker push mandric/alpine-bash:tshark
```
