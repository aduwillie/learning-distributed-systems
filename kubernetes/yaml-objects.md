# YAML in Kubernetes

Kubernetes uses YAML files as inputs for the creation of objects sush as pods, deployments, replicas and services.

All follow similar required root level structure of a definition:
- apiVersion
- kind
- metadata
- spec

## apiVersion

The version of the Kubernetes API version being used to create the object. Depending on what we are trying to create, we need the right version

```
apiVersion: v1 # for both Pods and Services and Secrets
apiVersion: apps/v1 # for both ReplicaSets and Deployments
apiVersion: extensions/v1beta1 # for ReplicaSets and Deployments
```


## kind

This refers to the type of object being created. This could be one of the following:

- Pod
- ReplicaSet
- Deployment
- Service
- ConfigMap
- CronJob
- Endpoint
- Node

This list is in no way exhaustive. There are others.

## metadata

This is data about the object that helps to uniquely identify it eg. name, labels. Unlike `apiVersion` and `kind`, which are string values, this is a form of a dictionary.
```
metadata:
    name: my-app-pod
    labels:
        app: my-app
        type: frontend
```
The `name` is a string value whereas the `labels` is a dictionay within the `metadata` dictionary. It can have any key/values as possible.

## spec

This is different for different objects.

### PodSpec Example

```
spec:
    containers: # this is a list (can have multiple containers)
    - name: nginx-container
        image: nginx
```

### ReplicaSetSpec

A ReplicaSet's purpose is to maintain a stable set of replica Pods running at any given time. It is used to guarantee the availability of a specified number of identical Pods. Prefer `Deployment` over `ReplicaSet` as this manages ReplicaSets.

```
spec:
    replicas: 3
    selector:
        matchLabels:
            type: frontend
    template:
        metadata:
            labels:
                type: frontend
        spec:
            containers:
            -   name: nginx-container
                image: nginx  
```

### DeploymentSpec

```
spec:
    replicas: 3
    selector:
        matchLabels:
            app: my-app
    template:
        metadata:
            labels:
                app: my-app
        spec:
            containers:
            -   name: nginx-container
                image: nginx
                ports:
                -   containerPort: 80
```

### ServiceSpec

```
spec:
    selector:
        app: my-app
    ports:
    -   protocol: tcp
        port: 80
        targetPort: 8090 # port on the pods
```
