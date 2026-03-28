# Kubernetes Multi-Container Pod Exercises (with Solutions)

---

# Exercise 1 -- Basic Multi-Container Pod

## Task

Create a Pod named **multi-pod** with two containers:

- **nginx** using image `nginx`
- **sidecar** using image `busybox` that runs:

```sh
sh -c "while true; do echo sidecar running; sleep 5; done"
```

Both containers should run in the same Pod.

<details>
<summary>Solution</summary>

```yaml
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

</details>

---

# Exercise 2 -- Sidecar Log Collector

## Task

Create a Pod called **log-sidecar-pod**:

Container 1:
- name: `app`
- image: `busybox`
- command writes logs to `/var/log/app.log`

Container 2:
- name: `log-agent`
- image: `busybox`
- reads the same log file continuously

Use a **shared emptyDir volume**.

<details>
<summary>Solution</summary>

```yaml
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
    command: ["sh","-c","touch /var/log/app.log; tail -f /var/log/app.log"]
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log
```

</details>

---

# Exercise 3 -- Ambassador Pattern

## Task

Create a Pod **ambassador-pod** with two containers:

- **redis**
- **ambassador proxy** (`socat`) forwarding traffic from port `6379`

<details>
<summary>Solution</summary>

```yaml
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

</details>

---

# Exercise 4 -- Init Container + App Container

## Task

Create a Pod that:

1. Uses an **initContainer**
2. Writes a file `/work/index.html`
3. The main container (`nginx`) serves it

<details>
<summary>Solution</summary>

```yaml
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

</details>

---

# Exercise 5 -- Multi-Container Pod with Shared Config

## Task

Create a Pod with:

Container 1:
- `nginx`

Container 2:
- `busybox` that modifies a config file every 10 seconds

Use a shared volume `/config`.

<details>
<summary>Solution</summary>

```yaml
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

</details>

---

# Exercise 6 -- Sidecar Filtering ERROR Logs

## Task

Create a Pod named **error-log-pod** with two containers:

- **app**: writes both `INFO` and `ERROR` messages to `/var/log/app.log`
- **sidecar**: reads the same file and shows only lines containing `ERROR`

Use a shared `emptyDir` volume mounted on `/var/log`.

<details>
<summary>Solution</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: error-log-pod
spec:
  volumes:
  - name: log-volume
    emptyDir: {}

  containers:
  - name: app
    image: busybox
    command: ["/bin/sh","-c"]
    args:
    - while true; do echo "$(date) INFO: ok" >> /var/log/app.log; echo "$(date) ERROR: failure" >> /var/log/app.log; sleep 5; done
    volumeMounts:
    - name: log-volume
      mountPath: /var/log

  - name: sidecar
    image: busybox
    command: ["/bin/sh","-c"]
    args:
    - touch /var/log/app.log; tail -f /var/log/app.log | awk '/ERROR/'
    volumeMounts:
    - name: log-volume
      mountPath: /var/log
```

</details>

---

# Exercise 7 -- Init Container Creates Shared Data

## Task

Create a Pod named **shared-data-pod** where:

- an **initContainer** creates a file `/data/message.txt` with text `Welcome from init container`
- the main container uses `busybox` and continuously prints the file content every 10 seconds

Use an `emptyDir` volume.

<details>
<summary>Solution</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: shared-data-pod
spec:
  volumes:
  - name: shared-data
    emptyDir: {}

  initContainers:
  - name: init-data
    image: busybox
    command: ["sh","-c","echo Welcome from init container > /data/message.txt"]
    volumeMounts:
    - name: shared-data
      mountPath: /data

  containers:
  - name: reader
    image: busybox
    command: ["sh","-c","while true; do cat /data/message.txt; sleep 10; done"]
    volumeMounts:
    - name: shared-data
      mountPath: /data
```

</details>

---

# Exercise 8 -- Shared HTML Between Containers

## Task

Create a Pod named **web-content-pod** with two containers:

- **writer** (`busybox`) updates `/html/index.html` every 15 seconds with the current date
- **web** (`nginx`) serves that file

Use a shared `emptyDir` volume.

<details>
<summary>Solution</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-content-pod
spec:
  volumes:
  - name: html-volume
    emptyDir: {}

  containers:
  - name: writer
    image: busybox
    command: ["sh","-c"]
    args:
    - while true; do echo "<h1>$(date)</h1>" > /html/index.html; sleep 15; done
    volumeMounts:
    - name: html-volume
      mountPath: /html

  - name: web
    image: nginx
    volumeMounts:
    - name: html-volume
      mountPath: /usr/share/nginx/html
```

</details>

---

# Exercise 9 -- Adapter Pattern

## Task

Create a Pod named **adapter-pod**:

- **app** container writes JSON lines to `/data/app.log`
- **adapter** container reads the same file and extracts only the `"level"` field

Example written line:

```json
{"level":"INFO","msg":"started"}
```

Use a shared `emptyDir`.

<details>
<summary>Solution</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: adapter-pod
spec:
  volumes:
  - name: shared-data
    emptyDir: {}

  containers:
  - name: app
    image: busybox
    command: ["sh","-c"]
    args:
    - while true; do echo '{"level":"INFO","msg":"started"}' >> /data/app.log; echo '{"level":"ERROR","msg":"failed"}' >> /data/app.log; sleep 5; done
    volumeMounts:
    - name: shared-data
      mountPath: /data

  - name: adapter
    image: busybox
    command: ["sh","-c"]
    args:
    - touch /data/app.log; tail -f /data/app.log | awk -F'"' '{print $4}'
    volumeMounts:
    - name: shared-data
      mountPath: /data
```

</details>

---

# Exercise 10 -- Multi-Container Pod with Readiness Probe

## Task

Create a Pod named **probe-pod** with two containers:

- **web** using `nginx`
- **helper** using `busybox` that runs forever

Add a **readinessProbe** to the `web` container using HTTP on port `80`.

<details>
<summary>Solution</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: probe-pod
spec:
  containers:
  - name: web
    image: nginx
    readinessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 10

  - name: helper
    image: busybox
    command: ["sh","-c","while true; do echo helper running; sleep 10; done"]
```

</details>

---

# Exercise 11 -- Init Container Downloads Config, App Uses It

## Task

Create a Pod named **config-init-pod**:

- an **initContainer** creates `/config/app.conf` with content `color=blue`
- the main container (`busybox`) prints the config every 10 seconds

Use an `emptyDir` volume mounted on `/config`.

<details>
<summary>Solution</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: config-init-pod
spec:
  volumes:
  - name: config-vol
    emptyDir: {}

  initContainers:
  - name: init-config
    image: busybox
    command: ["sh","-c","echo color=blue > /config/app.conf"]
    volumeMounts:
    - name: config-vol
      mountPath: /config

  containers:
  - name: app
    image: busybox
    command: ["sh","-c","while true; do cat /config/app.conf; sleep 10; done"]
    volumeMounts:
    - name: config-vol
      mountPath: /config
```

</details>

---

# Quick Commands 

## Create a pod quickly

```sh
kubectl run pod --image=nginx --dry-run=client -o yaml > pod.yaml
```

## Describe pod

```sh
kubectl describe pod PODNAME
```

## Check logs from a specific container

```sh
kubectl logs PODNAME -c container-name
```

## Open a shell in one container

```sh
kubectl exec -it PODNAME -c container-name -- sh
```

## Delete and recreate a pod

```sh
kubectl delete pod PODNAME --ignore-not-found
kubectl apply -f pod.yaml
```

---

# Multi-Container Patterns Summary

| Pattern | Purpose |
|---|---|
| Sidecar | Add helper functionality such as logging, monitoring, syncing |
| Ambassador | Proxy local traffic to another service |
| Adapter | Transform output format |
| Init Container | Prepare data/config before main app starts |

---

# Practice 

- Always check which container writes and which one reads
- Use `emptyDir` for file sharing inside the same Pod
- Remember that `kubectl logs` shows stdout/stderr, not file contents
- With BusyBox, some command options may be limited
- For multi-container Pods, always specify `-c container-name` in logs/exec
