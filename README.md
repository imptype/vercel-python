# @imptype/vercel-python

This is a copy of [@vercel/python@4.3.0][1] except with 2 things changed in `vc_init.py`:

1. Uses a shared event loop.
> Vercel does `asyncio.new_event_loop()` every new request, which leads to `RuntimeError` being raised if 2 requests use/create a resource that was created on a different event loop, for example `asyncio.Tasks` and a global `aiohttp.ClientSession` object. It also looks like Vercel makes every new request go through the sync function `vc_handler()`. So we run an event loop on a separate thread and make all requests use that one loop.
2. Runs the `__aenter__` part of lifespan.
> Vercel does not support lifespan. Lifespan is used init async code, like creating a database connection pool. This runs once per instance, not per request, so whatever is in there contributes to the coldstart time. A reference is kept so the Garbage Collector won't silently raise `GeneratorExit` on it. Finally, this only supports Starlette's lifespan which located in `app.router.lifespan_context`. This part will be skipped if no lifespan was found.

## Usage
In your `vercel.json` file, change the npm package to use to `@imptype/vercel-python@4.3.34`, for example:
```json
{
  "builds": [
    {
      "src": "main.py",
      "use": "@imptype/vercel-python@4.3.34"
    }
  ],
  "routes": [
    {
      "src": "/(.*)",
      "dest": "main.py"
    }
  ]
}
```

## Stable versions
- 3.4.12 - Uses shared event loop
- 3.4.33 - Uses shared event loop and runs the `__aenter__` part of lifespan

[1]: https://www.npmjs.com/package/@vercel/python/v/4.3.0
