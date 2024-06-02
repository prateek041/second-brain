# Writer Interface

Any type, that implements a _Write_ method can be used to write through packages like os, fmt etc.

## File Descriptors

File descriptors are the mechanism through which Operating System keeps track of all the open files, Sockets or pipes. When a program wants to write/read from a file, it does not do so through the file name,
but file descriptors.

There are some standards:

- stdin: 0
- stdout: 1
- stderr: 2

> Note: Closing the file frees the file descriptor to be re-used.

functions like fmt.Fprintln pass the arguments to the Write method of whatever the type the println is writing to. by default it is stdout, i.e. the console but you can select the output
