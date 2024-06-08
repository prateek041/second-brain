# Receivers in Go methods

## What are Receivers

In Go, methods are functions with a special Receiver argument. It allows to define methods on types.

The receiver in a method is the instance of the type on which the method is called

### Receiver types

- **Pointer Receiver** : A method with pointer Receiver can modify the receiver because it points to the actual data. It is used when you want to save space or mutate the receiver.

**Syntax**:

```go
func (h *Hello)ServeHTTP(...)
```

- **Value Receiver** : A method with a value receiver gets a copy of the receiver, here the original cannot be modified. Here the receivers are usually small.

**Syntax**

```go
func (h Hello)ServeHTTP(...)
```
