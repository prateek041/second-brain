# How Writer and Reader interfaces work in Go.

Anything that implements the writer interface can be written to, and any thing that implements the Reader interface can be read from many packages into the Go standard and third party library.

http.Reader implements the io.ReaderCloser interface, which means it has Read and Close methods implemented to it, we can read from it and close it.

http.ResponseWriter has a method `Wrte` which means it implements the writer interface, hence many tools can be used that can write on a Writer interface.

handlers are function logic that run depending on the request path. Every HTTP server has a default handler, which is of type ServeMux, it is also a handler but it is special because it decides what handler to call depending on the request path.

When a request hits the server, it calls the http.ServeMux.ServeHTTP, which in turn calls the ServeHTTP method of the handler registered to it, the selection of which method's ServeHTTP to call depends on the incoming request path.

> [!NOTE]
> ServeMux also implements the handler interface.
