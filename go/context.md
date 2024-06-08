# Contexts in Go

Contexts are used to carry deadlines, cancellation signals and other request scoped values across API boundaries. In case of a graceful shutdown of a server. the context is used to set a timeout for how long the server should take to finish the ongoing requests before shutting down.

- context.Background(): It is used to create a root context. I never cancels, has no values and no deadlines. It can be used at the start of application and used as the root of context tree.
- context.WithTimeout(): Creates a context that will automatically cancel after a specified duration. It can be used when you want an operation to be canceled when it has run longer than a specified period of time.
- context.WithDeadline(): Create a context that will automatically cancel at a specific time in the future. It can be used when want to cancel an operation at a specific deadline.
