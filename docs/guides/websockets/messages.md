# Receiving and sending messages

## Basic usage

- Receive a message with `await ws.receive()`.
- Send a message with `await ws.send(message)`.

## Iterating over messages

In the [introduction example](./README.md#a-basic-example), only one message was received and sent over the WebSocket, then we closed the connection.

What if we wanted to do this repeatedly, i.e. echo all messages that the client sends us through the WebSocket?

It turns out that `ws` is also an [asynchronous iterator], so we can iterate over each received message like so:

```python
async def echo(ws: WebSocket):
    async with ws:
        async for message in ws:
            await ws.send(f"You said: {message}")
```

This code is strictly equivalent to:

```python
async def echo(ws: WebSocket):
    async with ws:
        while True:
            message = await ws.receive()
            await ws.send(f"You said: {message}")
```

## Decoding and encoding messages

Messages are decoded or encoded as text messages by default: calling `receive()` and `send()` will return or expect `str` objects.

It is possible to specify how messages should be decoded (resp. encoded) by passing the `receive_type` (resp. `send_type`) argument to the `@api.websocket_route()` decorator.
 
 ::: tip
 For convenience, `value_type` can be used to set `receive_type` and `send_type` to the same value.
:::

The possible values for `receive_type`, `send_type` and `value_type` are:

| Type | Description | Returned by `receive()` | Expected by `send()` |
|--------------|-------------|-------------------------|----------------------|
| `"text"` | Plain text | `str` | `str` |
| `"bytes"` | Plain bytes | `bytes` | `bytes` |
| `"json"` | JSON, encoded to / decoded from text | `dict` or `list` | `dict` or `list` |
| `"event"` | [ASGI event](#using-asgi-events) | `dict` | `dict` 

For example, here's a WebSocket server that exchanges JSON messages with its clients:

```python
from bocadillo import API, WebSocket

api = API()

@api.websocket_route("/ping-pong", value_type="json")
async def ping_pong(ws: WebSocket):
    async with ws:
        async for message in ws:
            resp = "pong" if message["text"] == "ping" else "ping"
            await ws.send({"text": resp})

if __name__ == '__main__':
    api.run()
```

A JavaScript client for this would be:

```javascript
const ws = new WebSocket('ws://localhost:8000/ping-pong');

ws.onopen = (event) => {
    console.log("Connected:", event);
}

ws.onclose = (event) => {
    console.log("Connection lost:", event);
}

ws.onmessage = (event) => {
    console.log("Received:", JSON.parse(event.data));
}

setInterval(() => {
    const message = {
        text: (Math.random() > 0.5) ? "ping" : "pong"
    };
    ws.send(JSON.stringify(message));
    console.log("Sent: ", message);
}, 1000);
```

::: tip
In fact, `receive()` and `send()` are shortcuts that call the corresponding receive and send methods for the WebSocket's configured `receive_type` and `send_type`.

For example, if `receive_type` is set to `"json"`, then `await ws.receive()` is equivalent to `await ws.receive_json()`.

See also the API reference for the [WebSocket] class.
:::

## Using ASGI events

It is possible to receive or send raw [ASGI events][ASGI Event] using the low-level `receive_event()` and `send_event()` methods.

For example, this is a rough equivalent of the [introduction example](./README.md#a-basic-example) that uses ASGI events only:

```python
from bocadillo import API, WebSocket

@api.websocket_route("/echo")
async def echo(ws: WebSocket):
    await ws.send_event({"type": "websocket.accept"})
    event: dict = await ws.receive_event()
    message: str = event["text"]
    await ws.send_event({{"type": "websocket.send", "text": f"You said: {message}"}})
    await ws.send_event({"type": "websocket.close"})
```

[asynchronous iterator]: https://www.python.org/dev/peps/pep-0492/#asynchronous-iterators-and-async-for
[WebSocket]: ../../api/websockets.md#websocket
[ASGI Event]: https://asgi.readthedocs.io/en/latest/specs/main.html#events
