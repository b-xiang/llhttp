Connection header
=================

## `keep-alive`

### Setting flag

<!-- meta={"type": "request"} -->
```http
PUT /url HTTP/1.1
Connection: keep-alive


```

```log
off=0 message begin
off=4 len=4 span[url]="/url"
off=19 len=10 span[header_field]="Connection"
off=31 len=10 span[header_value]="keep-alive"
off=45 headers complete method=4 v=1/1 flags=1 content_length=0
off=45 message complete
```

### Restarting when keep-alive is explicitly

<!-- meta={"type": "request"} -->
```http
PUT /url HTTP/1.1
Connection: keep-alive

PUT /url HTTP/1.1
Connection: keep-alive


```

```log
off=0 message begin
off=4 len=4 span[url]="/url"
off=19 len=10 span[header_field]="Connection"
off=31 len=10 span[header_value]="keep-alive"
off=45 headers complete method=4 v=1/1 flags=1 content_length=0
off=45 message complete
off=45 message begin
off=49 len=4 span[url]="/url"
off=64 len=10 span[header_field]="Connection"
off=76 len=10 span[header_value]="keep-alive"
off=90 headers complete method=4 v=1/1 flags=1 content_length=0
off=90 message complete
```

### No restart when keep-alive is off (1.0) and parser is in strict mode

<!-- meta={"type": "request", "mode": "strict"} -->
```http
PUT /url HTTP/1.0

PUT /url HTTP/1.1


```

```log
off=0 message begin
off=4 len=4 span[url]="/url"
off=21 headers complete method=4 v=1/0 flags=0 content_length=0
off=21 message complete
off=22 error code=5 reason="Data after `Connection: close`"
```

### Resetting flags when keep-alive is off (1.0) and parser is in loose mode

Even though we allow restarts in loose mode, the flags should be still set to
`0` upon restart.

<!-- meta={"type": "request", "mode": "loose"} -->
```http
PUT /url HTTP/1.0
Content-Length: 0

PUT /url HTTP/1.1
Transfer-Encoding: chunked


```

```log
off=0 message begin
off=4 len=4 span[url]="/url"
off=19 len=14 span[header_field]="Content-Length"
off=35 len=1 span[header_value]="0"
off=40 headers complete method=4 v=1/0 flags=20 content_length=0
off=40 message complete
off=40 message begin
off=44 len=4 span[url]="/url"
off=59 len=17 span[header_field]="Transfer-Encoding"
off=78 len=7 span[header_value]="chunked"
off=89 headers complete method=4 v=1/1 flags=8 content_length=0
```

## Setting flag on `close`

<!-- meta={"type": "request"} -->
```http
PUT /url HTTP/1.1
Connection: close


```

```log
off=0 message begin
off=4 len=4 span[url]="/url"
off=19 len=10 span[header_field]="Connection"
off=31 len=5 span[header_value]="close"
off=40 headers complete method=4 v=1/1 flags=2 content_length=0
off=40 message complete
```

## Parsing multiple tokens

<!-- meta={"type": "request"} -->
```http
PUT /url HTTP/1.1
Connection: close, token, upgrade, token, keep-alive


```

```log
off=0 message begin
off=4 len=4 span[url]="/url"
off=19 len=10 span[header_field]="Connection"
off=31 len=40 span[header_value]="close, token, upgrade, token, keep-alive"
off=75 headers complete method=4 v=1/1 flags=7 content_length=0
off=75 message complete
```

## `upgrade`

### Setting a flag and pausing

<!-- meta={"type": "request"} -->
```http
PUT /url HTTP/1.1
Connection: upgrade
Upgrade: ws


```

```log
off=0 message begin
off=4 len=4 span[url]="/url"
off=19 len=10 span[header_field]="Connection"
off=31 len=7 span[header_value]="upgrade"
off=40 len=7 span[header_field]="Upgrade"
off=49 len=2 span[header_value]="ws"
off=55 headers complete method=4 v=1/1 flags=14 content_length=0
off=55 message complete
off=55 error code=21 reason="Pause on CONNECT/Upgrade"
```

### Emitting part of body and pausing

<!-- meta={"type": "request"} -->
```http
PUT /url HTTP/1.1
Connection: upgrade
Content-Length: 4
Upgrade: ws

abcdefgh
```

```log
off=0 message begin
off=4 len=4 span[url]="/url"
off=19 len=10 span[header_field]="Connection"
off=31 len=7 span[header_value]="upgrade"
off=40 len=14 span[header_field]="Content-Length"
off=56 len=1 span[header_value]="4"
off=59 len=7 span[header_field]="Upgrade"
off=68 len=2 span[header_value]="ws"
off=74 headers complete method=4 v=1/1 flags=34 content_length=4
off=74 len=4 span[body]="abcd"
off=78 message complete
off=78 error code=21 reason="Pause on CONNECT/Upgrade"
```

### Upgrade request sample

_(Ported from [http_parser][0])_

<!-- meta={"type": "request"} -->
```http
GET /demo HTTP/1.1
Host: example.com
Connection: Upgrade
Sec-WebSocket-Key2: 12998 5 Y3 1  .P00
Sec-WebSocket-Protocol: sample
Upgrade: WebSocket
Sec-WebSocket-Key1: 4 @1  46546xW%0l 1 5
Origin: http://example.com

Hot diggity dogg
```

```log
off=0 message begin
off=4 len=5 span[url]="/demo"
off=20 len=4 span[header_field]="Host"
off=26 len=11 span[header_value]="example.com"
off=39 len=10 span[header_field]="Connection"
off=51 len=7 span[header_value]="Upgrade"
off=60 len=18 span[header_field]="Sec-WebSocket-Key2"
off=80 len=18 span[header_value]="12998 5 Y3 1  .P00"
off=100 len=22 span[header_field]="Sec-WebSocket-Protocol"
off=124 len=6 span[header_value]="sample"
off=132 len=7 span[header_field]="Upgrade"
off=141 len=9 span[header_value]="WebSocket"
off=152 len=18 span[header_field]="Sec-WebSocket-Key1"
off=172 len=20 span[header_value]="4 @1  46546xW%0l 1 5"
off=194 len=6 span[header_field]="Origin"
off=202 len=18 span[header_value]="http://example.com"
off=224 headers complete method=1 v=1/1 flags=14 content_length=0
off=224 message complete
off=224 error code=21 reason="Pause on CONNECT/Upgrade"
```

[0]: https://github.com/nodejs/http-parser