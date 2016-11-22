# Future of Streams

# Streams v4
- Current Problems:
  - error handling
  - destroy()
  - flush for write stream
- Testing regressions
- (auto?)Decoding strings for Writable
- Removing dependencies from Internal EventEmitter
- HTTP2 in core
- Migrating old modules to Streams3


# userland modules
- p2p: e.g. discovery swarm
- stream multiplexing: e.g. multiplex, tentacoli
- Streaming protocol parsers: e.g. JSONstream2
- pull-stream
- flush-write-stream

# pull stream
have back pressure, so suit system programming.
observables are just about reactivity - good for UI
