# Building a web server in Go

## Using net/http

- ServeMux
- Handlers
- Dependency injection
- Timeouts
- Graceful Shutdown: if ctx is not provided, the server will serve the ongoing request indefinitely.
- Context
- Handlers with modular code, where the handlers don't care about how data is being read or written, the data provider should implement the logic of encoding/decoding JSON, using the io.Writer and io.Reader interfaces.
