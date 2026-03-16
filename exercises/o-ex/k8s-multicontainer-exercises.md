# Kubernetes Multi‑Container Pod Exercises (with Solutions)

This file contains practical exercises about **multi-container Pods** in
Kubernetes. The exercises follow a **CKA-style approach**: short tasks,
then a reference solution.

------------------------------------------------------------------------

# Exercise 1 -- Basic Multi‑Container Pod

## Task

Create a Pod named **multi-pod** with two containers:

-   **nginx** using image `nginx`

-   **sidecar** using image `busybox` that runs:

        sh -c "while true; do echo sidecar running; sleep 5; done"

Both containers should run in the same Pod.

## Solution

``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-pod
spec:
  containers:
  - name: nginx
    image: nginx
  - name: sidecar
    image: busybox
    command: ["sh","-c","while true; do echo sidecar running; sleep 5; done"]
```

------------------------------------------------------------------------

# Exercise 2 -- Sidecar Log Collector

## Task

Create a Pod called **log-sidecar-pod**:

Container 1: - name: `app` - image: `busybox` - command writes logs to
`/var/log/app.log`

Container 2: - name: `log-agent` - image: `busybox` - reads the same log
file continuously

Use a **shared emptyDir volume**.

## Solution

``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: log-sidecar-pod
spec:
  volumes:
  - name: shared-logs
    emptyDir: {}

  containers:
  - name: app
    image: busybox
    command: ["sh","-c","while true; do date >> /var/log/app.log; sleep 5; done"]
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log

  - name: log-agent
    image: busybox
    command: ["sh","-c","tail -f /var/log/app.log"]
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log
```

------------------------------------------------------------------------

# Exercise 3 -- Ambassador Pattern

## Task

Create a Pod **ambassador-pod** with two containers:

-   **redis**
-   **ambassador proxy** (socat) forwarding traffic from port 6379

## Solution

``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: ambassador-pod
spec:
  containers:
  - name: redis
    image: redis
    ports:
    - containerPort: 6379

  - name: ambassador
    image: alpine/socat
    args:
    - "tcp-listen:6379,fork,reuseaddr"
    - "tcp-connect:localhost:6379"
```

------------------------------------------------------------------------

# Exercise 4 -- Init Container + App Container

## Task

Create a Pod that:

1.  Uses an **initContainer**
2.  Writes a file `/work/index.html`
3.  The main container (`nginx`) serves it.

## Solution

``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-example
spec:
  volumes:
  - name: workdir
    emptyDir: {}

  initContainers:
  - name: init
    image: busybox
    command: ["sh","-c","echo Hello Kubernetes > /work/index.html"]
    volumeMounts:
    - name: workdir
      mountPath: /work

  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: workdir
      mountPath: /usr/share/nginx/html
```

------------------------------------------------------------------------

# Exercise 5 -- Multi‑Container Pod with Shared Config

## Task

Create a Pod with:

Container 1: - nginx

Container 2: - busybox that modifies a config file every 10 seconds

Use a shared volume `/config`.

## Solution

``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: shared-config-pod
spec:
  volumes:
  - name: config-vol
    emptyDir: {}

  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: config-vol
      mountPath: /etc/config

  - name: updater
    image: busybox
    command: ["sh","-c","while true; do date > /etc/config/time.txt; sleep 10; done"]
    volumeMounts:
    - name: config-vol
      mountPath: /etc/config
```

------------------------------------------------------------------------

# Quick Commands (Exam Tips)

Create pod quickly:

    kubectl run pod --image=nginx --dry-run=client -o yaml > pod.yaml

Describe containers:

    kubectl describe pod PODNAME

Check logs from specific container:

    kubectl logs PODNAME -c container-name

------------------------------------------------------------------------

# Multi‑Container Patterns Summary

  Pattern          Purpose
  ---------------- ------------------------------------------------
  Sidecar          Add helper functionality (logging, monitoring)
  Ambassador       Proxy local traffic
  Adapter          Transform output format
  Init Container   Setup environment before main app
