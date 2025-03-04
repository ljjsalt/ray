(serve-handle-explainer)=

# ServeHandle: Calling Deployments from Python

Ray Serve enables you to query models both from HTTP and Python. This feature
enables seamless [model composition](serve-model-composition-guide). You can
get a `ServeHandle` corresponding to deployment, similar how you can
reach a deployment through HTTP via a specific route. When you issue a request
to a deployment through `ServeHandle`, the request is load balanced across
available replicas in the same way an HTTP request is.

To call a Ray Serve deployment from python, use {mod}`Deployment.get_handle <ray.serve.api.Deployment>`
to get a handle to the deployment, then use
{mod}`handle.remote <ray.serve.handle.RayServeHandle.remote>` to send requests
to that deployment. These requests can pass ordinary args and kwargs that are
passed directly to the method. This returns a Ray `ObjectRef` whose result
can be waited for or retrieved using `ray.wait` or `ray.get`.

```{literalinclude} ../serve/doc_code/handle_guide.py
:start-after: __basic_example_start__
:end-before: __basic_example_end__
:language: python
```

If you want to use the same deployment to serve both HTTP and ServeHandle traffic, the recommended best practice is to define an internal method that the HTTP handling logic will call:

```{literalinclude} ../serve/doc_code/handle_guide.py
:start-after: __async_handle_start__
:end-before: __async_handle_end__
:language: python
```

Now we can invoke the same logic from both HTTP or Python:

```{literalinclude} ../serve/doc_code/handle_guide.py
:start-after: __async_handle_print_start__
:end-before: __async_handle_print_end__
:language: python
```

(serve-sync-async-handles)=

## Sync and Async Handles

Ray Serve offers two types of `ServeHandle`. You can use the `Deployment.get_handle(..., sync=True|False)`
flag to toggle between them.

- When you set `sync=True` (the default), a synchronous handle is returned.
  Calling `handle.remote()` should return a Ray `ObjectRef`.
- When you set `sync=False`, an asyncio based handle is returned. You need to
  Call it with `await handle.remote()` to return a Ray ObjectRef. To use `await`,
  you have to run `Deployment.get_handle` and `handle.remote` in Python asyncio event loop.

The async handle has performance advantage because it uses asyncio directly; as compared
to the sync handle, which talks to an asyncio event loop in a thread. To learn more about
the reasoning behind these, checkout our [architecture documentation](serve-architecture).
