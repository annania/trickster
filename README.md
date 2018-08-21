# Trickster ![Build Status](https://travis-ci.org/Comcast/trickster.svg)

[![Go Report Card](https://goreportcard.com/badge/github.com/Comcast/trickster)](https://goreportcard.com/report/github.com/Comcast/trickster)
[![FOSSA Status](https://app.fossa.io/api/projects/git%2Bgithub.com%2Fannania%2Ftrickster.svg?type=shield)](https://app.fossa.io/projects/git%2Bgithub.com%2Fannania%2Ftrickster?ref=badge_shield)

Trickster is a reverse proxy cache for the [Prometheus](https://github.com/prometheus/prometheus) [HTTP APIv1](https://prometheus.io/docs/prometheus/latest/querying/api/) that dramatically accelerates dashboard rendering times for any series queried from Prometheus.

<img src="./docs/images/high-level.png" width=512 />

## How it works

### 1. Delta Proxy
Most dashboards request the entire time range of data from the time series database, every time a dashboard loads or reloads. Trickster's Delta Proxy inspects the time range of a client query to determine what data points are already cached, and requests from Prometheus only the data points still needed to service the client request. This results in dramatically faster chart load times for everyone, since Prometheus is queried only for tiny incremental changes on each dashboard load, rather than several hundred data points of duplicative data.

<img src="./docs/images/partial-cache-hit.png" width=1024 />

### 2. Step Boundary Normalization
When Trickster requests data from Prometheus, it adjusts the clients's requested time range slightly to ensure that all data points returned by Prometheus are aligned to normalized step boundaries. For example, if the step is 300s, all data points will fall on the clock 0's and 5's. This ensures that the data is highly cacheable, is conveyed visually to users in a more familiar way, and that all dashboard users see identical data on their screens.

<img src="./docs/images/step-boundary-normalization.png" width=640 />

### 3. Fast Forward
Trickster's Fast Forward feature ensures that even with step boundary normalization, real-time graphs still always show the most recent data, regardless of how far away the next step boundary is. For example, if your chart step is 300s, and the time is currently 1:21p, you would normally be waiting another four minutes for a new data point at 1:25p. Trickster will break the step interval for the most recent data point and always include it in the response to clients requesting real-time data.

<img src="./docs/images/fast-forward.png" width=640 />

## Install

### Docker

Docker images are available on Docker Hub:

    $ docker run --name trickster -d -v /path/to/trickster.conf:/etc/trickster/trickster.conf -p 0.0.0.0:9090:9090 tricksterio/trickster

See the 'deploy' Directory for more information about using or creating Trickster docker images.

### Kubernetes and Helm

See the 'deploy' Directory for both Kube and Helm deployment files and examples.

### Building from source

To build Trickster from the source code yourself you need to have a working
Go environment with [version 1.9 or greater installed](http://golang.org/doc/install).

You can directly use the `go` tool to download and install the `trickster`
binary into your `GOPATH`:

    $ go get github.com/Comcast/trickster
    $ trickster -origin http://prometheus.example.com:9090


You can also clone the repository yourself and build using `make`:

    $ mkdir -p $GOPATH/src/github.com/Comcast
    $ cd $GOPATH/src/github.com/Comcast
    $ git clone https://github.com/Comcast/trickster.git
    $ cd trickster
    $ make build
    $ ./trickster -origin http://prometheus.example.com:9090

The Makefile provides several targets:

  * *build*: build the `trickster` binary
  * *docker*: build a docker container for the current `HEAD`
  * *clean*: delete previously-built binaries and object files

## More information

  * Refer to the docs directory for additional info.

## Contributing

Refer to [CONTRIBUTING.md](CONTRIBUTING.md)


## License
[![FOSSA Status](https://app.fossa.io/api/projects/git%2Bgithub.com%2Fannania%2Ftrickster.svg?type=large)](https://app.fossa.io/projects/git%2Bgithub.com%2Fannania%2Ftrickster?ref=badge_large)