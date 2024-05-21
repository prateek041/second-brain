This is how tetragon is handling cleanup of pod metrics, even after the pods are deleted

there are two parts to it:
- Handler
- Deleter

## Handler
Handler is resopnsible for regularly watching the informer cache for events, specifically Delete evetns. When delete happens, the handler pushes the pod related information to a worker queue.

## Deleter
Deleter is a go-routine that is responsible for picking things one-by-one from the queue and deleting them.
