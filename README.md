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

What if there was a protocol that allowed mounting or nesting other protocols, and made it easy to select which protocol to use?

At the operating system level, protocol selection is usually accomplished using numbered ports, with some out-of-band agreement on the mapping of port numbers to protocols. For example, HTTP requests will use TCP port 80 unless otherwise specified.

multistream-select brings this concept into the "protocol level", which gives you the flexibility to evolve a mapping of human-readable protocol names to the semantics you need, which may change over time. 

This is especially useful in environments where connections to arbitrary OS ports is difficult or impossible, for example because one or more parties is behind a NAT. In such cases, a single connection with a stream multiplexer will allow you to open many independent communication channels, but you'll need some mechanism for deciding what protocol to use for each channel. If you support multiple stream multiplexers, you can even use multistream-select to decide which one to use in the first place.

### Protocol

The actual protocol is very simple. It is a multistream protocol itself, with a multicodec header. And it has a set of other protocols available to be used by the remote side. The remote side must enter:

```
> <multistream-header-for-multistream>
> <multistream-header-for-whatever-protocol-that-we-want-to-speak>
```

for example:

```
> /multistream/1.0.0
> /ipfs/kad/1.0.0
```

- The `<multistream-header-for-multistream>` ensures a protocol selection is happening.
- The `<multistream-header-for-whatever-protocol-is-then-selected>` hopefully describes a valid protocol listed. Otherwise we return a `na`("not available") error:

```
na\n

# in hex (note the varint prefix = 3)
# 0x036e610a
```

for example:

```sh
# open connection + send multicodec headers, inc for a protocol not available
> /multistream/1.0.0
> /some-protocol-that-is-not-available

# open connection + signal protocol not available.
< /multistream/1.0.0
< na

# send a selection of a valid protocol + upgrade the conn and send traffic
> /ipfs/kad/1.0.0
> <kad-dht-traffic>
> ...

# receive a selection of the protocol + sent traffic
< /ipfs/kad/1.0.0
< <kad-dht-traffic>
< ...
```

**Note 1:** Every multistream message is a "length-prefixed-message", which means that every message is preprended by a varint that describes the size of the message.

**Note 2:** Every multistream message is appended by a `\n` character, this character is included in the byte count that is accounted by the prepended varint.

**Note 3:** Although multistream-select uses a "call and response" pattern to propose and confirm protocol selection, it is possible for both sides to send multistream headers at the same time. This is especially likely for the initial `<multistream-header-for-multistream>`, which both sides may optimistically send as soon as the connection is opened, without waiting for the other side to send it first.

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
> /multistream/1.0.0
> ls

# get response
< /multistream/1.0.0
< /ipfs/kad/0.2.3
< /ipfs/kad/1.0.0
< /ipfs/bitswap/0.4.3
< /ipfs/bitswap/1.0.0

# send selection, upgrade connection, and start protocol traffic
> /ipfs/ipfs-dht/0.2.3
> <kad-dht-request-0>
> <kad-dht-request-1>
> ...

# receive selection, and upgraded protocol traffic.
< /ipfs/kad/0.2.3
< <kad-dht-response-0>
< <kad-dht-response-1>
< ...
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
