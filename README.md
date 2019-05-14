# multistream-select

[![](https://img.shields.io/badge/made%20by-Protocol%20Labs-blue.svg?style=flat-square)](https://protocol.ai)
[![](https://img.shields.io/badge/project-multiformats-blue.svg?style=flat-square)](https://github.com/multiformats/multiformats)
[![](https://img.shields.io/badge/freenode-%23ipfs-blue.svg?style=flat-square)](https://webchat.freenode.net/?channels=%23ipfs)
[![](https://img.shields.io/badge/readme%20style-standard-brightgreen.svg?style=flat-square)](https://github.com/RichardLitt/standard-readme)

> Friendly protocol multiplexing. It enables a multicodec to be negotiated between two entities.

## Table of Contents

- [Motivation](#motivation)
  - [Protocol](#protocol)
    - [Listing](#listing)
  - [Example](#example)
  - [ls example in detail](#ls-example-in-detail)
- [Implementations](#implementations)
- [Maintainers](#maintainers)
- [Contribute](#contribute)
- [License](#license)

## Motivation

Some protocols have sub-protocols or protocol-suites. Often, these sub protocols are optional extensions. Selecting which protocol to use -- or even knowing what is available to chose from -- is not simple.

What if there was a protocol that allowed mounting or nesting other protocols, and made it easy to select which protocol to use. (This is sort of like ports, but managed at the protocol level -- not the OS -- and human readable).

### Protocol

The actual protocol is very simple. It is a multistream protocol itself, it has a multicodec header. And it has a set of other protocols available to be used by the remote side. The remote side must enter:

```
> <multistream-header>
> <multistream-header-for-whatever-protocol-that-we-want-to-speak>
```

for example:

```
> /ipfs/QmdRKVhvzyATs3L6dosSb6w8hKuqfZK2SyPVqcYJ5VLYa2/multistream-select/0.3.0
> /ipfs/QmVXZiejj3sXEmxuQxF2RjmFbEiE9w7T82xDn3uYNuhbFb/ipfs-dht/0.2.3
```

- The `<multistream-header-of-multistream>` ensures a protocol selection is happening.
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

**Note 1:** Every multistream message is a "length-prefixed-message", which means that every message is preprended by a varint that describes the size of the message.

**Node 2:** Every multistream message is appended by a `\n` character, this character is included in the byte count that is accounted by the prepended varint.

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
<varint-total-response-size-in-bytes><varint-number-of-protocols>
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

### ls example in detail

```
> <varint-length>ls<newline>
< <varint-length><varint-of-list-of-protos-size-in-bytes><varint-number-of-protocols><newline>
< <varint-length><protocol><newline>
# ...
< <varint-length><protocol><newline>
```

Note: Each `varint-length` contains the size of the rest of the line, including the newline bytes

## Implementations

- [js-multistream-select](https://github.com/multiformats/js-multistream-select) - JavaScript Implementation
- [go-multistream](https://github.com/multiformats/go-multistream) - Go Implementation
- [mss-nc](https://github.com/whyrusleeping/mss-nc) - multistream-select netcat written in Go

## Maintainers

Captain: [@diasdavid](https://github.com/diasdavid).

## Contribute

Contributions welcome. Please check out [the issues](https://github.com/multiformats/multistream-select/issues).

Check out our [contributing document](https://github.com/multiformats/multiformats/blob/master/contributing.md) for more information on how we work, and about contributing in general. Please be aware that all interactions related to multiformats are subject to the IPFS [Code of Conduct](https://github.com/ipfs/community/blob/master/code-of-conduct.md).

Small note: If editing the README, please conform to the [standard-readme](https://github.com/RichardLitt/standard-readme) specification.

## License

This repository is only for documents. All of these are licensed under the [CC-BY-SA 3.0](https://ipfs.io/ipfs/QmVreNvKsQmQZ83T86cWSjPu2vR3yZHGPm5jnxFuunEB9u) license © 2016 Protocol Labs Inc. Any code is under a [MIT](LICENSE) © 2016 Protocol Labs Inc.
