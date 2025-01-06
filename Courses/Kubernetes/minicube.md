# minikube - cheat sheet

[Dokumentace](https://minikube.sigs.k8s.io/docs/start/?arch=%2Flinux%2Fx86-64%2Fstable%2Fbinary+download)

---
[Použití Multi-Node clusteru](https://minikube.sigs.k8s.io/docs/tutorials/multi_node/)

## Create multinode cluster

```
minikube start --nodes 2 -p multinode-demo
```

---

## Set profile as main

```
minikube profile multinode-demo
```

## Start/stop created minikube profile

```
minikube start --profile=multinode-demo
```
```
minikube stop --profile=multinode-demo
```
