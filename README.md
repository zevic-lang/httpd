# Httpd

An HTTP/1.1 server for [Zevic](https://github.com/zevic-lang/zevic), written with `Zevic.Net` (no C code, no libraries).
It is the server that was `Zevic.Http` in the standard library before Zevic 0.6.0. For a client, use the package [Curl](https://github.com/zevic-lang/curl).

```
zevpm get zevic-lang/httpd@v0.6.0
```

or in `zevic.toml`:

```
[deps]
Httpd = { git = "https://github.com/zevic-lang/httpd", tag = "v0.6.0" }
```

## A server in a few lines

```
module Main

import Zevic.Console
import Httpd

fn Main() {
    let server: Httpd.Server = Httpd.Listen("127.0.0.1", 8080).Unwrap()
    Console.Println("listening on port ${server.Port()}")
    let r: Result<Unit, Error> = server.Serve((req: Httpd.Request) => {
        if (req.Path() == "/hello") {
            return Httpd.Response.Text(200, "Hello, ${req.Query()}!")
        }
        Httpd.Response.Text(404, "nothing at ${req.Path()}")
    })
}
```

`Serve` waits for connections until `server.Close()` is called (from another task; see `async` in the Zevic specification).
Port 0 picks a free port, and `server.Port()` tells which.

## What it does

| | |
| --- | --- |
| `Httpd.Listen(host, port)` | `Result<Server, Error>`; `"127.0.0.1"` for this computer only, `"0.0.0.0"` for every network |
| `Server.Serve(handler)` | answers requests until `Close()`; each connection gets its own task and one answer |
| `Request` | `Method`, `Url` (`/path?query`), `Body`, `Headers`, `Header(name)` (not case sensitive), `Path()`, `Query()` |
| `Response` | `Status`, `Body`, `Headers`; `Response.Text(status, body)`, `SetHeader(name, value)`, `Header(name)`, `IsOk()` |

- No TLS and no keep-alive (every answer has `Connection: close`). For HTTPS, put a reverse proxy in front.
- A body is read with `Content-Length` or `chunked`; 16 MiB at most. The header lines may be 64 KiB, and a read or a write may wait 30 seconds.
- A request that is not HTTP gets `400`; a body that is too long gets `413`.

## Building and testing

Zevic 0.6 or later and clang. `zevpm test` starts a server on this computer and talks to it over a plain TCP connection.
