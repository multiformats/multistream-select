# multistream - self-describing protocol streams

multistream is a format -- or simple protocol -- for disambiguating, and
layering streams. It is extremely simple.

multistream is one of the `multi-protocols`, a set of protocols that solve
problems with self-description.

## Motivation

To decode an incoming stream of data, a program must either (a) know the format of the data a priori, or (b) learn the format from the data itself. (a) precludes running protocols that may provide one of many kinds of formats without prior agreement on which. multistream makes (b) neat using self-description.

Moreover, this self-description allows straightforward layering of protocols without having to implement support in the parent (or encapsulating) one.

## How it works - Protocol Description

A `multistream` stream MUST begin with a simple header, followed by an arbitrary stream of data for the specified protocol:

```
<header>
<arbitrary-stream-data>
```

### The header

The header has three parts:

- `hdr-len` - a varint length, in bytes, for security and binary protocols.
- `path` - the path of the protocol in a universal namespace. UTF-8. must start with a slash.
- `\n` - a newline at the end, for the benefit of text protocols **(included in hdr-len)**.

It looks like this:
```
<hdr-len><path>\n
```

So the full protocol is:

```
<hdr-len><path>\n
<arbitrary-stream-data>
```

For example:

```sh
# this header:

/echo/1.0

# with hexdump -C style inspection:
0a 2f 65 63 68 6f 2f 31  2e 30 0a                 |./echo/1.0.|

0a    - varint hdr len   10 bytes
2f-30 - "/echo/1.0"       9 bytes
0a    - newline           1 byte
```

### The protocol path

`multistream` allows us to specify different protocols in a universal namespace, that way being able to recognize, multiplex, and embed them easily. We use the notion of a `path` instead of an `id` becuase it is meant to be a Unix-friendly URI.

A good path name should be decipherable -- meaning that if some machine or developer -- who has no idea about your protocol -- encounters the path string, they should be able to look it up and resolve how to use it.

An example of a good path name is:

```
/bittorrent.org/1.0
```

An example of a _great_ path name is:

```
/ipfs/Qmaa4Rw81a3a1VEx4LxB7HADUAXvZFhCoRdBzsMZyZmqHD/ipfs.protocol
/http/w3id.org/ipfs/ipfs-1.1.0.json
```

These path names happen to be resolvable -- not just in a "multistream muxer" but -- in the internet as a whole (provided the program (or OS) knows how to use the `/ipfs` and `/http` protocols).

## Implementations

- go-multistream (WIP)
- node-multistream (WIP)

