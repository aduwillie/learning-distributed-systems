# The Sidecar Pattern

This is a single node pattern made up of 2 containers. The first is the **application container** and the second is the **sidecar container**. The application container contains the core logic (application to run). The role of the sidecar is to augment and improve the application container without the application container's knowledge. Most often, the sidecar container is use dto add functionality to the application container that might otherwise be difficult to improve.

Sidecar containers are coschedueled onto the same machine via an atomic _container group_ such as a pod API object in Kubernetes. This way, they share a number of resources including filesystem, hostname and network, etc.

A good example of this pattern can be adding https to a legacy service. Using the sidecar pattern, the legacy application is bundled into a container. After an nginx container is setup as a sidecar in order to introduce the feature of https. Since the nginx container is a sidecar, it can access the same network space as the legacy application on localhost. Also, it can terminate the https traffic on the external IP and proxy the traffic to the legacy application.

Another good example is the need for configuration synchronization. Most applications uses a configuration file (either txt, JSON or XML) to store and parameterize an application. In this case, many pre-existing applications able to make assumptions of this file and read configuration from there. Similar to the case of HTTPS, the sidecar pattern again can be used to provide new functionality that augments the applications without changing the existing application. The application and its sidecar share a directory between themselves. This shared directory is where the configureation file is maintained.

Outside, the notion of augmenting legacy applications, the sidecar pattern can be used in modern application development as well. In today's world of building reliable applications, there is occasionally the need to gather system/container metrics similar to the `top` linux command line tool. This monitoring feature could benefit from the sidecar pattern.

## Building the `topz` application

First, deploy the application container:
```
docker run -d <app-image>
```
This would return the `container_hash_value`. If missed, a simple `docker ps` command can be used to retrieve it as it would show all currently running containers. We can then stash that hash into an environment variable called `APP_ID`.
```
export APP_ID=<container_hash_value>
```
At this point, the sidecar can be run in the same PID namespace as the application container. It should be noted that the port of the sidecar container should be different from the application container (assuming its 8080).
```
docker run --pid=container:${APP_ID} \
    -p 8080:8080 \
    brendanburns/topz:db0fa58 \
    /server --addr=0.0.0.0:8080
```

## Building Repo Updater

In the scenario, where we would like to pull new changes from a git repository, we could benefit more from the modularit of a sidecar pattern. This in turn can be deployed to various other projects.

The sidecar container could be the following bash script:
```
#!/bin/bash

while true; do
    git pull origin master
    sleep 20
done
```
Since both the sidecar and application can share the same filesystem resources, it is easier to deploy this added functionality. Assuming the name of that image is `aduwillie/repo-updater`, a corresponding kubernetes pod could be defined as:
```
apiVersion: v1
kind: Pod
metadata:
  name: updater
  labels:
    app: updater
spec:
  shareProcessNamespace: true # this is true by default
  containers:
  - name: nodejs-app
    image: aduwillie/sample-node-app
  - name: repo-updater
    image: node-alpine
    command: ["/bin/sh"]
    args: ["-c", "while true; do git pull origin master; sleep 20; done"]
    restartPolicy: OnFailure
```
