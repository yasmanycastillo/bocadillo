# Response
```python
Response(self, request: starlette.requests.Request, media_type: str, media_handler: Callable[[Any], str])
```
The response builder, passed to HTTP views and typically named `res`.

At the lower-level, `Response` behaves like an ASGI application instance:
it is a callable and accepts `receive` and `send` as defined in the [ASGI
spec](https://asgi.readthedocs.io/en/latest/specs/main.html#applications).

[media]: ../guides/http/media.md
[Content-Disposition]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Disposition

__Parameters__

- __request (Request)__: the currently processed request.
- __media_type (str)__: the configured media type (given by the `API`).
- __media_handler (callable)__: the configured media handler (given by the `API`).

__Attributes__

- `content (bytes or str)`: the raw response content.
- `status_code (int)`: the HTTP status code. If not set, defaults to `200`.
- `headers (dict)`:
    a case-insensitive dictionary of HTTP headers.
    If not set, `content-type` header is set to `text/plain`.
- `chunked (bool)`: sets the `transfer-encoding` header to `chunked`.
- `attachment (str)`:
    a file name that this response should be sent as.
    This is done by setting the [Content-Disposition] header, and
    typically makes the client browser trigger a "Save As…" dialog or
    download and save the file locally.

## html
Write-only property that sets `content` to the set value and sets the `Content-Type` header to `"text/html"`.
## media
Write-only property that sets `content` to the set value serializer using the `media_handler`, sets the `Content-Type` header to the `media_type`.
## text
Write-only property that sets `content` to the set value and sets the `Content-Type` header to `"text/plain"`.
## file
```python
Response.file(self, path: str, attach: bool = True)
```
Send a file asynchronously using [aiofiles].

This is typically used when the file should be downloaded by the client.

[aiofiles]: https://github.com/Tinche/aiofiles

__Parameters__

- __path (str)__:
    A path to a file on this machine.
- __attach (bool, optional)__:
    Whether to send the file as an [attachment](#response).
    Defaults to `True`.

## background
```python
Response.background(self, func: Callable[..., Coroutine], *args, **kwargs) -> Callable[..., Coroutine]
```
Register a coroutine function to be executed in the background.

This can be used either as a decorator or a regular function.

__Parameters__

- __func (callable)__:
    A coroutine function.
- __*args, **kwargs__:
    Any positional and keyword arguments to pass to `func` when
    executing it.

## stream
```python
Response.stream(self, func: Callable[[], AsyncIterable[Union[str, bytes]]]) -> Callable[[], AsyncIterable[Union[str, bytes]]]
```
Stream the response.

Should be used to decorate a no-argument asynchronous generator
function.

