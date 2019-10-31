# Kubernetes snippets

## Make a temporary toolbox for debugging

### Why

To easily debug from inside the Kubernetes cluster

### How

```
kubectl run toolbox --generator=run-pod/v1 --rm -it --tty --image ubuntu:latest -- bash
```
