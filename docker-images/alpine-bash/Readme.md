
Build the image and give it a name (or name and tag like name:tag):

```
docker build -t "omi/alpine-bash" .
```

Run an interactive bash shell in a temporary container with the new image:

```
docker run -ti --rm omi/alpine-bash bash
```

Mount a local directory and run tests in the container:

```
docker run --rm \
  -v "$PWD:/opt/project" \
  omi/alpine-bash \
  /bin/sh -c 'cd /opt/project; make'
```
