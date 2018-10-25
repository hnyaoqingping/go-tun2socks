# go-tun2socks

[![Build Status](https://travis-ci.com/eycorsican/go-tun2socks.svg?branch=master)](https://travis-ci.com/eycorsican/go-tun2socks)

A tun2socks implementation written in Go.

Tested and worked on macOS, Linux, Windows and iOS (as a library).

## Overview

```
                                      lwip.NewLWIPStack()
                                           +
                                           |
                                           |
                                           |
                                           |                TCP/UDP             lwip.RegisterTCPConnectionHandler()
                                           |
                          lwip.Input()     |           tun2socks.Connection     lwip.RegisterUDPConnectionHandler()
                                           v
Application +------> TUN +-----------> lwIP stack +------------------------------> tun2socks.ConnectionHandler +-------> Remote proxy server +--> Destination


                         <-----------+
                    lwip.RegisterOutputFn()

```

## Features

- Support both TCP and UDP (only IPv4 for now)
- Supported proxy handlers: SOCKS5, Shadowsocks, V2Ray
- DNS cache
- UDP direct relaying (no proxy)
- ICMP local echoing
- Dynamically adding routing rules according to V2Ray's routing results (V2Ray proxy handler only)

## Build

`go-tun2socks` is using `cgo`, thus a C compiler is required.

```sh
go get github.com/eycorsican/go-tun2socks
cd $GOPATH/src/github.com/eycorsican/go-tun2socks
go get -d ./...
make clean && make build
./build/tun2socks -h
```

An alternative way to build (or cross compile) tun2socks is to use [`xgo`](https://github.com/karalabe/xgo), to use `xgo`, you also need `docker`:

```sh
# install docker: https://docs.docker.com/install

# install xgo
go get github.com/karalabe/xgo

go get github.com/eycorsican/go-tun2socks
cd $GOPATH/src/github.com/eycorsican/go-tun2socks
go get -d ./...
make clean && make xbuild
ls ./build
```

## Run

```sh
./build/tun2socks -tunName tun1 -tunAddr 240.0.0.2 -tunGw 240.0.0.1 -proxyType socks -proxyServer 1.2.3.4:1086
```

Note that the TUN device may have a different name, and it should be a different name on Windows unless you have renamed it, so make sure use `ifconfig`, `ipconfig` or `ip addr` to check it out.

## Create TUN device and Configure Routing Table

Suppose your original gateway is 192.168.0.1. The proxy server address is 1.2.3.4.

The following commands will need root permissions.

### macOS

The program will automatically create a TUN device for you on macOS. To show the created TUN device, use ifconfig.

Delete original gateway:

```sh
route delete default
```

Add our TUN interface as the default gateway:

```sh
route add default 240.0.0.1
```

Add a route for your proxy server to bypass the TUN interface:

```sh
route add 1.2.3.4/32 192.168.0.1
```

### Linux

The program will not create the TUN device for you on Linux. You need to create the TUN device by yourself:

```sh
ip tuntap add mode tun dev tun1
ip addr add 240.0.0.2 dev tun1
ip link set dev tun1 up
```

Delete original gateway:

```sh
ip route del default
```

Add our TUN interface as the default gateway:

```sh
ip route add default via 240.0.0.1
```

Add a route for your proxy server to bypass the TUN interface:

```sh
ip route add 1.2.3.4/32 via 192.168.0.1
```

### Windows

To create a TUN device on Windows, you need [Tap-windows](https://openvpn.net/index.php/download/community-downloads.html), refer [here](https://code.google.com/archive/p/badvpn/wikis/tun2socks.wiki) for more information.

Add our TUN interface as the default gateway:

```sh
route add 0.0.0.0 mask 0.0.0.0 240.0.0.1 metric 6
```

Add a route for your proxy server to bypass the TUN interface:

```sh
route add 1.2.3.4 192.168.0.1 metric 5
```

## What happened to lwIP?

Take a look at this repo: https://github.com/eycorsican/lwip

## TODO
- Built-in routing rules and routing table management
- Support IPv6
- Support ICMP packets forwarding

## Acknowledgements
- https://github.com/zhuhaow/tun2socks
- https://github.com/yinghuocho/gotun2socks
- https://github.com/shadowsocks/go-shadowsocks2
- https://github.com/nadoo/glider
