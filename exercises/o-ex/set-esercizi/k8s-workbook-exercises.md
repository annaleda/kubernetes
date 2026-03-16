# Kubernetes Practice Workbook (CKA / CKAD)

## Instructions

Complete the following exercises using Kubernetes manifests or kubectl
commands.

------------------------------------------------------------------------

# Multi-Container Pods

### Exercise 1

Create a Pod `multi-basic` with two containers: - nginx - busybox
printing "hello" every 5 seconds

### Exercise 2

Create a Pod `sidecar-logs`: - app container writes logs to
`/var/log/app.log` - sidecar container reads the log file

### Exercise 3

Create a Pod `shared-volume` where two containers share an `emptyDir`
volume.

### Exercise 4

Create a Pod `init-web`: - init container creates `/data/index.html` -
nginx serves it

### Exercise 5

Create a Pod `metrics-sidecar`: - nginx container - busybox printing
metrics every 10s

### Exercise 6

Create a Pod where a sidecar tails `/var/log/nginx/access.log`.

### Exercise 7

Create a Pod using an `emptyDir` volume shared between two containers
writing timestamps.

### Exercise 8

Create a Pod `ambassador-redis` with: - redis container - proxy
container forwarding port 6379.

### Exercise 9

Create a Pod where a helper container modifies config files in a shared
volume.

### Exercise 10

Create a Pod with 3 containers sharing the same volume.

------------------------------------------------------------------------

# Network Policies

### Exercise 11

Create a default deny ingress policy for namespace `dev`.

### Exercise 12

Create a default deny egress policy for namespace `dev`.

### Exercise 13

Allow ingress only from Pods with label `role=frontend`.

### Exercise 14

Allow traffic only on port 80.

### Exercise 15

Allow traffic from namespace `monitoring`.

### Exercise 16

Allow DNS egress on port 53 TCP/UDP.

### Exercise 17

Allow egress only to namespace `database`.

### Exercise 18

Allow ingress from specific Pod label `app=backend`.

### Exercise 19

Create policy allowing ingress from namespace `logging` on port 9200.

### Exercise 20

Create policy allowing egress to external IP `8.8.8.8`.

------------------------------------------------------------------------

# Storage

### Exercise 21

Create a Pod using `emptyDir`.

### Exercise 22

Create a Pod using `hostPath` `/data`.

### Exercise 23

Create a PersistentVolume with 1Gi capacity.

### Exercise 24

Create a PersistentVolumeClaim requesting 1Gi.

### Exercise 25

Create a Pod mounting the PVC.

### Exercise 26

Create a PV with access mode `ReadWriteMany`.

### Exercise 27

Create a Pod using two volumes.

### Exercise 28

Create an init container preparing a shared volume.

------------------------------------------------------------------------

# RBAC & Security

### Exercise 29

Create a ServiceAccount `app-sa`.

### Exercise 30

Create a Role allowing get/list Pods.

### Exercise 31

Bind the Role to the ServiceAccount.

### Exercise 32

Create a ClusterRole allowing get nodes.

### Exercise 33

Bind ClusterRole to user `dev-user`.

### Exercise 34

Create a Pod running as user 1000.

### Exercise 35

Create a Pod with readOnlyRootFilesystem enabled.

### Exercise 36

Create a Pod with capabilities dropped.

------------------------------------------------------------------------

# Troubleshooting

### Exercise 37

Create a Pod with a failing liveness probe.

### Exercise 38

Create a Pod with readiness probe on port 8080.

### Exercise 39

Create a Service exposing Pod on port 80.

### Exercise 40

Create a Deployment with 3 replicas.

### Exercise 41

Scale Deployment to 5 replicas.

### Exercise 42

Expose Deployment via ClusterIP service.

### Exercise 43

Create ConfigMap and mount it.

### Exercise 44

Create Secret and mount it.

### Exercise 45

Create Pod using env variables from ConfigMap.
