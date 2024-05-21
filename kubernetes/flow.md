## Namespaces

There are four namespaces by deafult
- Kube-system (systyem pods)
- Default (default namespace for all the pods you run)
- Kube-node-lease (holds "lease" which are special terms given to heartbeats)
- kube-public

> Note: Run command `kubectl get lease -n kube-node-lease`
> Note: Run command `kubectl get cm -n kube-public`

Namespaces are used for isolating things in the same kubernetes cluster. Things like Dev, Prod etc.
