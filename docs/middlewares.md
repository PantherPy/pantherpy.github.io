> <b>Variable:</b> `MIDDLEWARES` 
> 
> <b>Type:</b> `list` 
> 
> <b>Default:</b> `[]`

Panther has several `built-in` middleware:

- Database Middleware

- Redis Middleware

And you can write your own custom middlewares too


## Structure of middlewares
`MIDDLEWARES` itself is a `list` of `tuples` which each `tuple` is like below:

(`Address of Middleware Class`, `kwargs as dict`)


## Database Middleware
This middleware will create a `db` connection that uses in `ODM` or you can use it manually from:
```python
from panther.db.connection import db
```

We only support 2 database: `PantherDB` & `MongoDB`

- Address of Middleware: `panther.middlewares.db.DatabaseMiddleware`
- kwargs:
    * `{'url': f'pantherdb://{BASE_DIR}/{DB_NAME}.pdb'}`

    * `{'url': f'mongodb://{DB_HOST}:27017/{DB_NAME}'}`

- Example of `PantherDB` (`Built-in Local Storage`):
  ```python
  MIDDLEWARES = [
      ('panther.middlewares.db.DatabaseMiddleware', {'url': f'pantherdb://{BASE_DIR}/{DB_NAME}.pdb'}),
  ]
  ```
- Example of `MongoDB`:
  ```python
  MIDDLEWARES = [
      ('panther.middlewares.db.DatabaseMiddleware', {'url': f'mongodb://{DB_HOST}:27017/{DB_NAME}'}),
  ]
  ```
  
## Redis Middleware
- Address of Middleware: `panther.middlewares.redis.RedisMiddleware`
- kwargs: 
    ```python
    {'host': '127.0.0.1', 'port': 6379, ...}
    ```

- Example
  ```python
  MIDDLEWARES = [
      ('panther.middlewares.redis.RedisMiddleware', {'host': '127.0.0.1', 'port': 6379}),
  ]
  ```
  
## Custom Middleware
Write a `class` and inherit from
```python
from panther.middlewares.base import BaseMiddleware
```

Then you can write your custom `before()` and `after()` methods

- The `methods` should be `async`
- `before()` should have `request` parameter
- `after()` should have `response` parameter
- overwriting the `before()` and `after()` are optional
- The `methods` can get `kwargs` from their `__init__`

### Custom Middleware Example
core/middlewares.py
```python
from panther.request import Request
from panther.response import Response
from panther.middlewares.base import BaseMiddleware


class CustomMiddleware(BaseMiddleware):

    def __init__(self, something):
        self.something = something

    async def before(self, request: Request) -> Request:
        print('Before Endpoint', self.something)
        return request

    async def after(self, response: Response) -> Response:
        print('After Endpoint', self.something)
        return response
```
core/configs.py
```python
  MIDDLEWARES = [
      ('core.middlewares.CustomMiddleware', {'something': 'hello-world'}),
  ]
```