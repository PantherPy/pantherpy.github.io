Panther supports `WebSockets` routing just like `APIs`

## Structure & Requirements
### Create WebSocket Class

Create the `BookWebsocket()` in `app/websockets.py` which inherited from `GenericWebsocket`: 


```python
from panther.websocket import GenericWebsocket


class BookWebsocket(GenericWebsocket):
    async def connect(self):
        await self.accept()
        print(f'{self.connection_id=}')

    async def receive(self, data: str | bytes = None):
        # Just Echo The Message
        await self.send(data=data)
```

> We are going to discuss it below ...

### Update URLs

Add the `BookWebsocket()` in `app/urls.py`:

```python
from app.websockets import BookWebsocket


urls = {
    'ws/book/': BookWebsocket,
}
```

## How It Works?

1. Client tries to connect to your `ws/book/` url with `websocket` protocol
2. The `connect()` method of your `BookWebsocket` is going to call
3. You should validate the connection with `self.headers`, `self.query_params` or etc
4. Then `accept()` the connection with `self.accept()`
5. Now you can see the unique `connection_id` which is specified to this user with `self.connection_id`, you may want to store it somewhere (`db`, `cache`, or etc.)
6. If client sent you any message you are going to receive it in `receive()` method, the client message can be `str` or `bytes`.
7. If you want to send anything to the client:
    - In websocket class scope: You can send it with `self.send()` which only takes `data`.
    - Out of websocket class scope: You can send it with `send_message_to_websocket()` from `panther.websocket`, it's an `async` function which takes 2 args, `connection_id` and `data` which can have any type, like below:
        ```python
        from panther.websocket import send_message_to_websocket
        await send_message_to_websocket(connection_id='connection_id', data='New Message From WS') 
        ```
8. If you want to use `webscoket` in `multi-tread` or `multi-instance` backend, you should add `RedisMiddleware` in your `configs` or it won't work well.
[[Adding Redis Middleware]](https://pantherpy.github.io/middlewares/#redis-middleware)
9. If you want to close a connection:
    - In websocket class scope: You can close connection with `self.close()` method which takes 2 args, `code` and `reason`, like below:
        ```python
        from panther import status
        await self.close(code=status.WS_1000_NORMAL_CLOSURE, reason='I just want to close it')
        ```
    - Out of websocket class scope **(Not Recommended)**: You can close it with `close_websocket_connection()` from `panther.websocket`, it's `async` function with takes 3 args, `connection_id`, `code` and `reason`, like below: 
        ```python
        from panther import status
        from panther.websocket import close_websocket_connection
        await close_websocket_connection(connection_id='connection_id', code=status.WS_1008_POLICY_VIOLATION, reason='')
        ``` 

10. `Path Variables` are going to pass to `connect()` like below:
   ```python
    from panther.websocket import GenericWebsocket

    class UserWebsocket(GenericWebsocket):
        async def connect(self, user_id: int, room_id: str):
            await self.accept()

    url = {
        '/ws/<user_id>/<room_id>/': UserWebsocket   
    }
   ``` 
11. WebSocket Echo Example -> [Https://GitHub.com/PantherPy/echo_websocket](https://github.com/PantherPy/echo_websocket)
12. Enjoy.

