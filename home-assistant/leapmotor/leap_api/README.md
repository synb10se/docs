# Internal Leapmotor API Layer

This package contains Home Assistant independent API primitives used by the
integration:

- `exceptions.py`: API/auth exception classes
- `models.py`: raw API data models
- `remote.py`: verified remote-control command payloads
- `crypto.py`: session and operatePassword derivation helpers
- `transport.py`: synchronous curl transport

The public integration still imports `LeapmotorApiClient` through
`custom_components.leapmotor.api` for compatibility. The client can be moved
into this package in a later step once the remaining status normalizer has been
split from the Home Assistant entity layer.
