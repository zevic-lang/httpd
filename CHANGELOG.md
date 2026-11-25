# Changelog

## 0.6.0

First version: the HTTP/1.1 server that was `Zevic.Http` in the standard library (removed in Zevic 0.6.0), now a package on top of `Zevic.Net`.

- `Httpd.Listen(host, port)` gives a `Server`; `Serve(handler)` answers until `Close()` (which can be called on a `let` server, from another task). A handler is a function from `Httpd.Request` to `Httpd.Response`.
- `Request`: `Method`, `Url`, `Body`, `Headers`, `Header(name)`, `Path()`, `Query()`. `Response`: `Status`, `Body`, `Headers`, `Response.Text(status, body)`, `SetHeader`, `Header`, `IsOk`.
- One task for each connection, one answer for each connection (`Connection: close`). No TLS and no keep-alive.
- Bodies with `Content-Length` or `chunked` (16 MiB at most), header lines up to 64 KiB, 30 seconds for a read or a write. A request that is not HTTP gets 400, a body that is too long gets 413.
- Needs Zevic 0.6 (`Net.TcpListener.Shutdown`).
