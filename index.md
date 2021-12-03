# Traffic Refinery

Traffic analysis using [Go packet library](https://github.com/google/gopacket).

## Compilation  

### From source on your native host
Additionally to go, Traffic Refinery requires libpcap and PF_RING installed in your system to correctly compile and run.

* [Install Go and set up your local environment](https://golang.org/doc/install).
  * `sudo apt-get install golang`

* Get the libpcap and libpcap-dev packages for your dev platform.
* [Install PF_RING](https://www.ntop.org/guides/pf_ring/get_started/index.html).
<!-- * Install other dependencies:
  * `go get github.com/spf13/viper`
  * `go get github.com/sirupsen/logrus`
  * `go get github.com/google/gopacket` -->

Once you have installed all the dependencies, you can proceed to install Traffic Refinery.

* Clone the traffic refinery source code `go get github.com/traffic-refinery/traffic-refinery` and change to the downloaded folder `cd $GOHOME/src/github.com/traffic-refinery/traffic-refinery/`.
* Download go modules
  * `go mod init github.com/traffic-refinery/traffic-refinery`
  * `go mod tidy`
* Run `build.sh`. 
  * For cross-compilation, check below.


## Run instructions

### Configuration
  
 Traffic Refinery supports configuration in json format. When executed, unless specified otherwise, Traffic Refinery searches for a configuration file named `trconfig.json` first in the execution folder and then in the `/etc/traffic-refinery/` folder. to import a configuration from a file located in a different location use the `-conf` option at execution.
 
 *NOTE: providing a configuration file is necessary.* 

 For details on the configuration of Traffic Refinery please refer to the [config](internal/config/config.go) package. Multiple examples are available in the [configs](configs/) folder.

#### Service configuration
Coming soon...


## Using Docker
To simplify dependencies management it is possible to use a docker container to develop or execute Traffic Refinery. There are three docker releases that can be used under different scenarios.

### Development

Develop and quickly test new features.

```
# Create image
docker image build --tag traffic-refinery:devel -f ./Dockerfile.devel .

# Run (from your local $GO_HOME/go/src/github.com/traffic-refinery/traffic-refinery/)
docker run -it --entrypoint /bin/bash --rm --mount type=bind,source=$(pwd),destination=/go/src/github.com/traffic-refinery/traffic-refinery/ traffic-refinery:devel
```

Once run using this command, the terminal remains open inside the `traffic-refinery` folder inside your container. This folder is fully synchronized (see more about the `mount` command on Docker's website) with your host folder. You can follow the instructions in the `Compilation` section for compiling the code.

### Running

Run a precompiled container with Traffic Refinery.

```
# Create image
docker image build --tag traffic-refinery:linux-amd64 .

# Run 
docker run --detach --mount source=/tmp,target=/tmp \
    --network=host --name tr --rm \
    traffic-refinery:linux-amd64
```

This runs the container in the background and the output of Traffic Refinery is store inside the `/tmp/` folder (see more about the `mount` command on Docker's website). To you a custom configuration mount the desired file into the container and reference to it using the command line parameters.

```
# Run 
docker run --detach --mount source=/tmp,target=/tmp \
    --network=host --name tr --rm \
    -mount type=bind,source=path/to/config/newconf.json,destination=/config/newconf.json,readonly
    traffic-refinery:linux-amd64 -conf /config/newconf.json
```

## Cross-compilation instructions (Linux)
Coming soon...
