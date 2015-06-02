# multistream-select - self-described protocol multiplexing

Friendly protocol multiplexing.

## Motivation

Some protocols have sub-protocols or protocol-suites. Often, these sub protocols are optional extensions. Selecting which protocol to use -- or even knowing what is available to chose from -- is not simple.

What if there was a protocol that allowed mounting or nesting other protocols, and made it easy to select which protocol to use. (This is sort of like ports, but managed at the protocol level -- not the OS -- and human readable).

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
