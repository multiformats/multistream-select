# msg-stream -- a length-delimited packetizing multistream (WIP)

msg-stream is a trivial protocol that "packetizes" an arbitrary stream. It is used to wrap arbitrary streams of data as varint-length delimited messages. This ensures the other size can stop.

Related:
- [msgio](https://github.com/jbenet/go-msgio)

msg-stream is a multistream. it's identifier is:

```
/multiproto.io/msg-stream/<version>
```

## Motivation

msg-streams give the other side the ability to know how large an incomming message is. This makes it easy to allocate appropriately sized buffers, and makes the _end_ of the message known.

--- WIP
