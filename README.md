# docker-protobufjs
A docker container image for generating code with
[protobuf.js](https://github.com/dcodeIO/protobuf.js).

This project is from `schottra`and is inspired by
[docker-protoc](https://github.com/namely/docker-protoc). It mimics its
behavior.

# Install

```sh
docker build -t protobufjs:latest https://github.com/loris-intergalactique/docker-protobufjs.git#master
```

# Basic Usage
The primary use case is generating JS and TS files via the protobufjs command
line tools (`pbjs` / `pbts`). Nearly all arguments that can be passed to those
tools can also be passed to the container and will be forwarded accordingly.

Files are generated to `/gen`, input/output setting is not implemented.

The only two hard requirements are:

> [!IMPORTANT]
> You **MUST**:
> * Mount your root protobuf directory to the `/defs` volume in the docker container.
> * Specify a `--module-name` parameter when running the container

To specify your target Protobuf file(s):
* Use `-d` or `--dir` to specifies one or more directories to search for
  .proto files.
* Pass individual file names directly to pbjs by placing them after the argument
  list, after `--`

## Basic example

The command below processes ./protos non-recursively to search for `*.proto`
files, and bundles them as a static es6 module into `/gen/pb-js/mymodule.js`.

```sh
docker run --m -it \
    -v "${PWD}:/defs" \
    -w /defs \
    docker-protobufjs:latest \
        --module-name mymodule \
        -d protos \
        -- \
        -t static-module \
        -w es6
```

It will also generate Typescript definitions into `/gen/pb-js/mymodule.d.ts`.

**_Note:_** All arguments placed after the `--` are forwarded to `pbjs`.
This gives you complete control over the behavior of the output so that you can
tailor it to your needs.

Here are the pbjs docs: [command line reference](https://github.com/dcodeIO/protobuf.js#command-line)


## More examples
Process multiple directories (`protos/core` and `protos/client`) using a shared
root include path (`/defs/protos`):

```sh
docker run --m -it \
    -v "${PWD}:/defs" \
    -w /defs \
    docker-protobufjs:latest \
        --module-name mymodule \
        -d protos/core \
        -d protos/client \
        -- \
        -t static-module \
        -w es6 \
        -p /defs/protos
```

Use advanced `pbjs` options to trim out unnecessary functionality and set a root
protobuf namespace name:

```sh
docker run --m -it \
    -v "${PWD}:/defs" \
    -w /defs \
    docker-protobufjs:latest \
        --module-name mymodule \
        -d protos/core \
        -d protos/client \
        -- \
        --root mymodule \
        -t static-module \
        -w es6 \
        --no-delimited \
        --force-long \
        --no-convert
```
