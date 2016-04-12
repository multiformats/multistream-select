lpm-stream
==========

[![](https://img.shields.io/badge/made%20by-Protocol%20Labs-blue.svg?style=flat-square)](http://ipn.io)
[![](https://img.shields.io/badge/project-IPFS-blue.svg?style=flat-square)](http://ipfs.io/)
[![](https://img.shields.io/badge/freenode-%23ipfs-blue.svg?style=flat-square)](http://webchat.freenode.net/?channels=%23ipfs)

> length-prefixed-message for length-delimited packetizing multistream

# Description

length-prefixed-message is a trivial protocol that "packetizes" an arbitrary stream. It is used to wrap arbitrary streams of data as varint-length delimited messages. This ensures the other size can stop.

Related:
- [msgio](https://github.com/jbenet/go-msgio)
- [length-prefixed-message](https://www.npmjs.com/package/length-prefixed-message)

# Motivation

length-prefixed-message give the other side the ability to know how large an incomming message is. This makes it easy to allocate appropriately sized buffers, and makes the _end_ of the message known.
