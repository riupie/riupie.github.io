---
title: Container Linux
weight: 210
menu:
  notes:
    name: Container
    identifier: notes-sysadmin-container
    parent: notes-sysadmin
    weight: 10
---

{{< note title="Generate Kubernetes secret and configmap YAML manifest" >}}

```bash
# Secret
kubectl create secret generic my-config --from-file=configuration/ -o yaml --dry-run

# Configmap
kubectl create configmap my-config --from-file=configuration/ -o yaml --dry-run
```

{{< /note >}}



{{< note title="Generate docker credential secret for Kubernetes" >}}

```bash
kubectl create -n registry secret docker-registry registry-auth-dockerconfig-secret --docker-server=registry.rahmatawe.com --docker-username=[YOUR_USERNAME] --docker-password=[YOUR_REGISTRY_PASSWORD] --dry-run=client -oyaml
```

{{< /note >}}


{{< note title="Running short-lived container to compile source code" >}}

```bash
$ sudo docker run --rm -it -v ${PWD}:/app -w /app openjdk:11-jdk-slim /bin/sh -c ./gradlew buildRun
```

{{< /note >}}