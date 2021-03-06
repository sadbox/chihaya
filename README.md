# Chihaya [![Build Status](https://api.travis-ci.org/chihaya/chihaya.svg?branch=master)](https://travis-ci.org/chihaya/chihaya)

Chihaya is a high-performance [BitTorrent tracker](http://en.wikipedia.org/wiki/BitTorrent_tracker) written in the Go programming language. It is still heavily under development and the current `master` branch should probably not be used in production unless you know what you're doing.

Features include:

- Low resource consumption, and fast, asynchronous request processing
- Full IPv6 support, including handling for dual-stacked peers
- Full compatibility with what exists of the BitTorrent spec
- Extensive metrics for visibility into the tracker and swarm's performance
- Ability to group peers in local subnets to reduce backbone contention
- Pluggable backend driver that can coordinate with an external database

## Using Chihaya

Chihaya can be ran as a public or private tracker and is intended to coordinate with existing torrent-indexing web frameworks, such as [Gazelle], [Batter] and any others that spring up. Following the Unix way, it is built to perform one specific task: handling announces and scrapes. By cleanly separating the concerns between tracker and database, we can provide an interface that can be used by system that needs its functionality.

[batter]: https://github.com/wafflesfm/batter
[gazelle]: https://github.com/whatcd/gazelle

### Installing

Chihaya requires Go 1.3+ to build. To install the Chihaya server, run:

```sh
$ go get github.com/chihaya/chihaya/cmd/chihaya
```

Make sure you have your `$GOPATH` set up correctly, and have `$GOPATH/bin` in your `$PATH`. If you're new to Go, an overview of the directory structure can be found [here](http://golang.org/doc/code.html).

### Configuring

Configuration is done in a JSON formatted file specified with the `-config` flag. An example configuration file can be found [here](https://github.com/chihaya/chihaya/blob/master/example_config.json).

#### Drivers

Chihaya is designed to remain agnostic about the choice of data storage. Out of the box, we provide only the necessary drivers to run Chihaya in public mode ("memory" for tracker and "noop" for backend). If you're interested in creating a new driver, check out the section on [customizing Chihaya].

[customizing Chihaya]: #customizing-chihaya


## Developing Chihaya

### Testing

Chihaya has end-to-end test coverage for announces in addition to unit tests for isolated components. To run the tests, use:

```sh
$ cd $GOPATH/src/github.com/chihaya/chihaya
$ go test -v ./...
```

There is also a set of benchmarks for performance-critical sections of Chihaya. These can be run similarly:

```sh
$ cd $GOPATH/src/github.com/chihaya/chihaya
$ go test -v ./... -bench .
```

### Customizing Chihaya

Chihaya is designed to be extended. If you require more than the drivers provided out-of-the-box, you are free to create your own and then produce your own custom Chihaya binary. To create this binary, simply create your own main package, import your custom drivers, then call [`chihaya.Boot`] from main.

[`chihaya.Boot`]: http://godoc.org/github.com/chihaya/chihaya

#### Example

```go
package main

import (
	"github.com/chihaya/chihaya"

  // Import any of your own drivers.
	_ "github.com/yourusername/chihaya-custom-backend"
)

func main() {
  // Start Chihaya normally.
	chihaya.Boot()
}
```

#### Implementing a Driver

The [`backend`] package is meant to provide announce deltas to a slower and more consistent database, such as the one powering a torrent-indexing website. Implementing a backend driver is heavily inspired by the standard library's [`database/sql`] package: simply create a package that implements the [`backend.Driver`] and [`backend.Conn`] interfaces and calls [`backend.Register`] in it's [`init()`]. Please note that [`backend.Conn`] must be thread-safe. A great place to start is to read the [`no-op`] driver which comes out-of-the-box with Chihaya and is meant to be used for public trackers.

[`init()`]: http://golang.org/ref/spec#Program_execution
[`database/sql`]: http://godoc.org/database/sql
[`backend`]: http://godoc.org/github.com/chihaya/chihaya/backend
[`backend.Register`]: http://godoc.org/github.com/chihaya/chihaya/backend#Register
[`backend.Driver`]: http://godoc.org/github.com/chihaya/chihaya/backend#Driver
[`backend.Conn`]: http://godoc.org/github.com/chihaya/chihaya/backend#Conn
[`no-op`]: http://godoc.org/github.com/chihaya/chihaya/backend/noop

### Contributing

If you're interested in contributing, please contact us via IRC in **[#chihaya] on
[freenode]** or post to the GitHub issue tracker. Please don't write
massive patches with no prior communication, as it will most
likely lead to confusion and time wasted for everyone. However, small
unannounced fixes are always welcome!

[#chihaya]: http://webchat.freenode.net?channels=chihaya
[freenode]: http://freenode.net

And remember: good gophers always use gofmt!
