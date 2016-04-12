multistream
===========

[![](https://img.shields.io/badge/made%20by-Protocol%20Labs-blue.svg?style=flat-square)](http://ipn.io)
[![](https://img.shields.io/badge/project-IPFS-blue.svg?style=flat-square)](http://ipfs.io/)
[![](https://img.shields.io/badge/freenode-%23ipfs-blue.svg?style=flat-square)](http://webchat.freenode.net/?channels=%23ipfs)

> Friendly protocol multiplexing. It enables a multicodec to be negotiated between two entities.

## Motivation

Some protocols have sub-protocols or protocol-suites. Often, these sub protocols are optional extensions. Selecting which protocol to use -- or even knowing what is available to chose from -- is not simple.

What if there was a protocol that allowed mounting or nesting other protocols, and made it easy to select which protocol to use. (This is sort of like ports, but managed at the protocol level -- not the OS -- and human readable).

### Protocol

The actual protocol is very simple. It is a multistream protocol itself, it has a multicodec header. And it has a set of other protocols available to be used by the remote side. The remote side must enter:

```
> <multicodec-header-of-multistream>
> <multicodec-header-for-whatever-protocol-that-we-want-to-speak>
```

for example:

```
> /ipfs/QmdRKVhvzyATs3L6dosSb6w8hKuqfZK2SyPVqcYJ5VLYa2/multistream-select/0.3.0
> /ipfs/QmVXZiejj3sXEmxuQxF2RjmFbEiE9w7T82xDn3uYNuhbFb/ipfs-dht/0.2.3
```

- The `<multicodec-header-of-multistream>` ensures a protocol selection is happening.
- The `<multistream-header-for-whatever-protocol-is-then-selected>` hopefully describes a valid protocol listed. Otherwise we return a `na`("not available") error:

```
na\n

# in hex (note the varint prefix = 3)
# 0x036e610a
```

for example:

```sh
# open connection + send multicodec headers, inc for a protocol not available
> /ipfs/QmdRKVhvzyATs3L6dosSb6w8hKuqfZK2SyPVqcYJ5VLYa2/multistream-select/0.3.0
> /ipfs/QmVXZiejj3sXEmxuQxF2RjmFbEiE9w7T82xDn3uYNuhbFb/some-protocol-that-is-not-available

# open connection + signal protocol not available.
< /ipfs/QmdRKVhvzyATs3L6dosSb6w8hKuqfZK2SyPVqcYJ5VLYa2/multistream-select/0.3.0
< na

# send a selection of a valid protocol + upgrade the conn and send traffic
> /ipfs/QmVXZiejj3sXEmxuQxF2RjmFbEiE9w7T82xDn3uYNuhbFb/ipfs-dht/0.2.3
> <dht-traffic>
> ...

# receive a selection of the protocol + sent traffic
< /ipfs/QmVXZiejj3sXEmxuQxF2RjmFbEiE9w7T82xDn3uYNuhbFb/ipfs-dht/0.2.3
< <dht-traffic>
< ...
```

#### Listing

It is also possible to "list" the available protocols. A list message is simply:

```
ls\n

# in hex (note the varint prefix = 3)
0x036c730a
```

So a remote side asking for a protocol listing would look like this:

```sh
# request
<multistream-header-for-multistream-select>
ls\n

# response
# TODO: maybe include a varint number of protocols here ?
<multicodec-for-multistream>
<multicodec-of-available-protocol>
<multicodec-of-available-protocol>
<multicodec-of-available-protocol>
...
```

For example

```sh
# send request
> /ipfs/QmdRKVhvzyATs3L6dosSb6w8hKuqfZK2SyPVqcYJ5VLYa2/multistream-select/0.3.0
> ls

# get response
< /ipfs/QmdRKVhvzyATs3L6dosSb6w8hKuqfZK2SyPVqcYJ5VLYa2/multistream-select/0.3.0
< /ipfs/QmVXZiejj3sXEmxuQxF2RjmFbEiE9w7T82xDn3uYNuhbFb/ipfs-dht/0.2.3
< /ipfs/QmVXZiejj3sXEmxuQxF2RjmFbEiE9w7T82xDn3uYNuhbFb/ipfs-dht/1.0.0
< /ipfs/QmVXZiejj3sXEmxuQxF2RjmFbEiE9w7T82xDn3uYNuhbFb/ipfs-bitswap/0.4.3
< /ipfs/QmVXZiejj3sXEmxuQxF2RjmFbEiE9w7T82xDn3uYNuhbFb/ipfs-bitswap/1.0.0

# send selection, upgrade connection, and start protocol traffic
> /ipfs/QmVXZiejj3sXEmxuQxF2RjmFbEiE9w7T82xDn3uYNuhbFb/ipfs-dht/0.2.3
> <ipfs-dht-request-0>
> <ipfs-dht-request-1>
> ...

# receive selection, and upgraded protocol traffic.
< /ipfs/QmVXZiejj3sXEmxuQxF2RjmFbEiE9w7T82xDn3uYNuhbFb/ipfs-dht/0.2.3
< <ipfs-dht-response-0>
< <ipfs-dht-response-1>
< ...
```

### Example

```
# greeting
> /http/multiproto.io/multistream-select/1.0
< /http/multiproto.io/multistream-select/1.0

# list available protocols
> /http/multiproto.io/multistream-select/1.0
> ls
< /http/google.com/spdy/3
< /http/w3c.org/http/1.1
< /http/w3c.org/http/2
< /http/bittorrent.org/1.2
< /http/git-scm.org/1.2
< /http/ipfs.io/exchange/bitswap/1
< /http/ipfs.io/routing/dht/2.0.2
< /http/ipfs.io/network/relay/0.5.2

# select protocol
> /http/multiproto.io/multistream-select/1.0
> ls
> /http/w3id.org/http/1.1
> GET / HTTP/1.1
>
< /http/w3id.org/http/1.1
< HTTP/1.1 200 OK
< Content-Type: text/html; charset=UTF-8
< Content-Length: 12
<
< Hello World
```

# Implementations

- [js-multistream](https://github.com/diasdavid/js-multistream) - JavaScript Implementation
- [go-multistream](https://github.com/whyrusleeping/go-multistream) - Go Implementation
- [mss-nc](https://github.com/whyrusleeping/mss-nc) - multistream netcat written in Go
