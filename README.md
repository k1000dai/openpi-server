# openpi-server

`openpi-server` is a small WebSocket server package for exposing an OpenPI
policy over the network.

It provides `WebsocketPolicyServer`, which:

- accepts observation messages from a WebSocket client,
- runs `policy.infer(obs)`,
- returns the action response,
- exposes a `/healthz` HTTP check endpoint.

## Installation

From PyPI:

```bash
pip install openpi-server
```

For local development:

```bash
uv sync
```

## Import

Python import names use underscores (not hyphens):

```python
from openpi_server import WebsocketPolicyServer
from openpi_server import websocket_policy_server
```

## Quick Start

```python
from openpi_server import WebsocketPolicyServer


class DummyPolicy:
    def infer(self, obs: dict) -> dict:
        return {"echo": obs}


server = WebsocketPolicyServer(
    policy=DummyPolicy(),
    host="0.0.0.0",
    port=8000,
    metadata={"policy_name": "dummy"},
)
server.serve_forever()
```

## Runtime Behavior

When a client connects:

1. The server sends `metadata` as the first frame.
2. For each received observation frame, it calls `policy.infer(obs)`.
3. It sends back the action dict with timing info:
   - `server_timing.infer_ms`
   - `server_timing.prev_total_ms` (from the previous loop, when available)

If an internal error happens, the traceback is sent and the connection is closed
with an internal error code.

## Health Check

`GET /healthz` returns:

- status: `200 OK`
- body: `OK`

## Development

Run a local build:

```bash
uv build
```

Build artifacts are created in `dist/`.

## Release to PyPI (GitHub Actions)

This repository includes `.github/workflows/publish-pypi.yml`, which publishes
on tag push (`v*`).

Required setup:

1. Create a PyPI API token.
2. Add it to GitHub Actions secrets as `PYPI_API_TOKEN`.
3. Update version in `pyproject.toml`.
4. Push a release tag:

```bash
git tag v0.1.0
git push origin v0.1.0
```

## License

Apache-2.0. See `LICENSE`.

## Acknowledgements

This package uses code from the
[`Physical-Intelligence/openpi`](https://github.com/Physical-Intelligence/openpi)
repository.
