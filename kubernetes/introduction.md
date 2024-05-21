## Architecture

## Control Plane

How the flow of a simple command "kubectl run nginx --image=nginx" looks like from beginning to the end.

### API Server
The request first hits the API server, which is the main brain of the entire Kubernetes Cluster.

First few steps are to:
- Authenticate: through the headers passed.
- Authorize: Authorized using RBAC (are they allowed to do what they are trying to do ?).
- Admissions: Saved to ETCD.

> TODO: Checkout webhook validation of things like image.

### ETCD

ETCD is the key-value datastore of the Kubernetes Cluster. It stores the kubernetes state into it, things like "what is running, where is it running, how long is it running " etc.

### Schedular

Schedular is a very intelligent component of the kubernetes, it tells which node to run the selected pod on.

It tries to find the best fit node based on the taints, tolerations, affinity, nodeselector and updates the Pod Spec with the node information.

As a user who wants to run a pod on Kubernetes, we can provide information like "Request, Limits, Resource" etc. that the pod will need, Schedular selects a node in the cluster while considering all that information.

### Controller Manager

For a simple example like running a Pod, controller manager is not significant, but when we are working with other objects like Deployments, Controller manager comes in.

It is a binary, that contains "controller logics" of how to create an run a specific object. So, if we are creating a Deployment, Deployment controller will handle the life-cycle of that obejct which in turn will create the replica set object, which itself will be managed by the replica set controller and the cycle goes on.

Controllers are present for these Kubernetes objects:
- ReplicaSet
- Deployment
- Job
- StatefulSets
- DaemonSets

## Worker Plane

There are a certain components which are present in all the worker planes.

When it comes to running the containers, three major components are to be configured.
- CRI (Container Runtime Interface) eg: (ContainerD, CRI-O, Docker)
- CNI (Container Networking Interface) eg: (Flanner, Cilium, calico)
- CSI (Container Storage Interface) eg: (LongHorn, openEBS)

> Note: we have Open Standards defined and all the projects mentioned above are implementations of those standards

### CRI (Container Runtime Interface)

By default, kubernetes uses ContainerD, but we can use anyone like CRI-O, Docker etc.

### Kubelet

Kubelet keeps polling the API server for "is there anything I need to run ?"

### KubeProxy

This component maintains the network rules. This allows communication with pods, from inside and outside the cluster.

It holds things like IPTables, IPVs etc.
