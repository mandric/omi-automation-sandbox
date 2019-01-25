
[![CircleCI](https://circleci.com/gh/mandric/omi-automation-sandbox.svg?style=svg)](https://circleci.com/gh/mandric/omi-automation-sandbox)

# Running tests

The Lua dissector testing framework has the following dependencies:

    bash, make, git, jq, bats and tshark

Once the dependencies are met, to run the tests just clone the repository and
run `make`:

```
git clone $repo_url
cd testing-dissectors
make
```

## Using Docker

A Docker image to run tests is convenient.  Here is an example Dockerfile using
[Alpine Linux](https://hub.docker.com/_/alpine) as a base image that produces a
minimal linux system image.

```
FROM alpine
MAINTAINER Jane Doe "jdoe+docker@gmail.com"
ENV REFRESHED_AT 2019-25-01
RUN apk --update --no-cache add bash bats make git jq tshark
```

Create the image (omi/alpine-bash), assuming the Dockerfile exists in the same
directory:

```
docker build -t omi/alpine-bash .
```

To run the tests using the new image, launch a container that also mounts the
repository directory, then `cd` to the directory and run `make`:

```
docker run --rm \
  -v "$PWD:/opt/project" \
  omi/alpine-bash \
  /bin/sh -c 'cd /opt/project; make'
```

# About

This project basically provides a way to compare/assert handfuls of JSON using
`bash`, [jq](https://stedolan.github.io/jq/) and the
[bats](https://github.com/sstephenson/bats) test runner.

It's mostly sugar around this core assertion, `json-compare`:

```
jq \
  --argfile actual "$1" \
  --argfile expected "$2" \
  -n -e '$actual == $expected'
```

The `parse-pcap` program takes the Lua dissector and pcap data then prints the
JSON data needed for one test/comparison:

```parse-pcap
#!/usr/bin/env bash
# Output JSON to stdout

set -o pipefail

tshark \
  -r "$1" \
  -X "lua_script:$2" \
  -T json \
  | jq ".[0]._source.layers.lua"
```

Write a program to generate test data, this creates a directory of JSON files
to run all your tests against:

```gen-data
#!/usr/bin/env bash

set -o pipefail
set -o errexit
set -o noclobber

INPUT="$1"
OUTPUT_DIR="$2"

cat "$INPUT" | while read line; do
  pcap="$(echo $line | sed 's/:.*//')"
  lua="$(echo $line | sed 's/.*://')"
  dir=$(dirname "$pcap")
  file=$(basename "$pcap" | sed 's/\.pcap/\.json/')
  mkdir -p "$OUTPUT_DIR/$dir"
  ./parse-pcap "$pcap" "$lua" > "$OUTPUT_DIR/$dir/$file";
done
```

Generate the fixtures directory:

```
./gen-data pcap-to-lua.txt fixtures
```

Write a program to generate your tests using the template and test data:

```test.tmpl
@test "${name}" {
  ./json-compare "${expected}" "${actual}"
}
```

```gen-tests
#!/usr/bin/env bash

find fixtures -name \*.json | while read line do;
  name=$(basename "$line" .json)
  expected="$line"
  actual="$(echo $line | sed 's/fixtures/actual')"
  envsubst < test.tmpl
done
```

Generate tests based on JSON files in fixtures:

```
./gen-tests fixtures > tests
bats tests
```
