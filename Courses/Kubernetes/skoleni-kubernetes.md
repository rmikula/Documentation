# Kubernetes

## Architektura Kubernetes

![kubernetes arch](https://kubernetes.io/images/docs/components-of-kubernetes.svg)

- [Dokumentace](https://kubernetes.io/docs/concepts/overview/components)

# Ovládání a instalace Kubernetes

kubectl
- HTTP požadavky na API server
    - https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/
- minikube: distribuce Kubernetes        
    - https://minikube.sigs.k8s.io/docs/start/

# Definice v Kubernetes

- Manifesty
- YAML
- JSON

# Definice v Kubernetes: YAML

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    resources: {}
```

```json
{
    "kind": "Pod",
    "apiVersion": "v1",
    "metadata": {
        "name": "nginx",
        "creationTimestamp": null,
        "labels": {
            "run": "nginx"
        }
    },
    "spec": {
        "containers": [
            {
                "name": "nginx",
                "image": "nginx",
                "resources": {}
            }
        ],
        "restartPolicy": "Always",
        "dnsPolicy": "ClusterFirst"
    },
    "status": {}
}
```

# kubectl bash-completion

- kubectl completion bash > ~/.kubectl_completion
- echo "source ~/.kubectl_completion" >> ~/.bashrc
- . ~/.bashr

# Jednoduchá aplikace

- Vzpomínáte na pody?
- `kubectl run nginx --image=nginx`

# kubectl get

```
kubectl get all
kubectl get <resource type> [<name>] [-o yaml|json]
kubectl get events [-w]
```

# kubectl describe

```
kubectl describe <resource type> [<name>]
```

# kubectl api-resources

```
➜  ~ kubectl api-resources 
NAME                SHORTNAMES   APIVERSION   NAMESPACED   KIND
bindings                         v1           true         Binding
componentstatuses   cs           v1           false        ComponentStatus
configmaps          cm           v1           true         ConfigMap
endpoints           ep           v1           true         Endpoints
events              ev           v1           true         Event
limitranges         limits       v1           true         LimitRange
...
```

# kubectl explain

- kubectl explain <api.resource.ktery.vas.zajima>

```
kubectl explain pod

FIELDS:
  apiVersion    <string>
    APIVersion defines the versioned schema of this representation of an object.

kubectl explain pod.apiVersion

KIND:       Pod
VERSION:    v1

FIELD: apiVersion <string>


DESCRIPTION:
    APIVersion defines the versioned schema of this representation of an object.
    Servers should convert recognized schemas to the latest internal value, and
    may reject unrecognized values. More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources
...
```

# Nápověda v kubectl

- bash completion
- kubectl help
- kubectl \<command> --help