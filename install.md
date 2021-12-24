---
title: Installation
layout: default
has_children: false
parent: 
grand_parent: 
nav_order: 2
---

# Installation
You can install `Traffic Refinery` using either Docker or by compiling from
source.

## Using Docker
To simplify dependency management it is possible to use a docker container to
develop or execute `Traffic Refinery`. There are two docker releases that can
be used under different scenarios.

### Pre-built release

We provide a precompiled container with Traffic Refinery.

#### Create image
```
docker image build --tag traffic-refinery:linux-amd64 .
```

#### Start traffic refinery 
```
docker run --detach --mount source=/tmp,target=/tmp \
    --network=host --name tr --rm \
    traffic-refinery:linux-amd64
```

This runs the container in the background and the output of Traffic Refinery is
store inside the `/tmp/` directory (see more about the `mount` command on
Docker's website). 

To use a custom configuration mount the desired file into the container and
reference to it using the command line parameters:

#### Start traffic refinery with custom configuration file
```
docker run --detach --mount source=/tmp,target=/tmp \
    --network=host --name tr --rm \
    -mount type=bind,source=path/to/config/newconf.json,destination=/config/newconf.json,readonly
    traffic-refinery:linux-amd64 -conf /config/newconf.json
```

### Development release

We have provided a release to develop and quickly test new features.

#### Create image
```
docker image build --tag traffic-refinery:devel -f ./Dockerfile.devel .
```

#### Run (from your local $GO_HOME/go/src/github.com/traffic-refinery/traffic-refinery/)
```
docker run -it --entrypoint /bin/bash --rm --mount \
    type=bind,source=$(pwd),destination=/go/src/github.com/traffic-refinery/traffic-refinery/ \
    traffic-refinery:devel
```

Once run using this command, the terminal remains open inside the
`traffic-refinery` directory inside your container. This directory is fully
synchronized (see more about the `mount` command on Docker's website) with your
host directory. You can follow the instructions in the [Compilation From
Source](##Compilation-From-Source) section for compiling the code.

## Compilation From Source 

### From source on your native host
In addition to go, `Traffic Refinery` requires libpcap and PF_RING installed in
your system to correctly compile and run.

* [Install Go and set up your local environment](https://golang.org/doc/install).
  * `sudo apt-get install golang`
* Get the libpcap and libpcap-dev packages for your dev platform.
  * `sudo apt-get install libpcap-dev`
* [Install PF_RING](https://www.ntop.org/guides/pf_ring/get_started/index.html).
  * `sudo apt-get install pfring`


Once you have installed all the dependencies, you can proceed to install
`Traffic Refinery`.

* Clone the traffic refinery source code `go get
  github.com/traffic-refinery/traffic-refinery` and change to the downloaded
  directory `cd $GOHOME/src/github.com/traffic-refinery/traffic-refinery/`.
* Download the necessary modules
  <!-- * `go mod init github.com/traffic-refinery/traffic-refinery` -->
  * `go mod tidy`
* Run `make`. 