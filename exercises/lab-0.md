# Kubernetes Labs -- Exercises

## Exercise 3.1 -- Deploy a New Application

### Install Python

``` bash
sudo apt-get -y install python3
```

### Check Python binary

``` bash
which python3
```

### Create working directory

``` bash
mkdir ~/app1
cd ~/app1
ls -l
```

### Create Python script

``` python
#!/usr/bin/python3
import time
import socket

while True:
    now = time.strftime("%Y-%m-%d %H:%M:%S")
    print(now, socket.gethostname())
    with open("date.out", "a") as f:
        f.write(now + "\n")
    time.sleep(5)
```

Run:

``` bash
chmod +x simple.py
./simple.py
cat date.out
```

### Dockerfile

``` dockerfile
FROM docker.io/library/python:3
ADD simple.py /
CMD ["python", "./simple.py"]
```

### Build image

``` bash
sudo podman build -t simpleapp .
sudo podman images
sudo podman run simpleapp
```

------------------------------------------------------------------------

## Exercise 3.2 -- Configure Local Registry

Create registry resources:

``` bash
kubectl create -f registry.yaml
kubectl get svc | grep registry
curl <registry-ip>:5000/v2/_catalog
```

Configure local repo:

``` bash
cp /home/student/LFS258/SOLUTIONS/03/local-repo-setup.sh $HOME
chmod +x local-repo-setup.sh
sudo ./local-repo-setup.sh
```

Test image push:

``` bash
sudo podman pull docker.io/library/alpine
sudo podman tag alpine $repo/tagtest
sudo podman push $repo/tagtest
```

------------------------------------------------------------------------

## Deploy Application

``` bash
kubectl create deployment try1 --image=$repo/simpleapp
kubectl scale deployment try1 --replicas=6
kubectl get pod -o wide
```

------------------------------------------------------------------------

## Exercise 3.3 -- Configure Probes

Edit deployment:

``` bash
vim simpleapp.yaml
```

Add readiness probe:

``` yaml
readinessProbe:
  exec:
    command:
      - cat
      - /tmp/healthy
```

Recreate deployment:

``` bash
kubectl delete deployment try1
kubectl create -f simpleapp.yaml
kubectl get pods
```

Create health file:

``` bash
kubectl exec -it <pod> -- /bin/bash
touch /tmp/healthy
exit
```

------------------------------------------------------------------------

## Exercise 4.2 -- Create a Job

``` yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: sleepy
spec:
  template:
    spec:
      containers:
      - name: resting
        image: busybox
        command: ["/bin/sleep"]
        args: ["3"]
      restartPolicy: Never
```

``` bash
kubectl create -f job.yaml
kubectl get job
kubectl delete job sleepy
```

------------------------------------------------------------------------

## Exercise 4.3 -- Create a CronJob

``` yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: sleepy
spec:
  schedule: "*/2 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: resting
            image: busybox
            command: ["/bin/sleep"]
            args: ["30"]
          restartPolicy: Never
```

``` bash
kubectl create -f cronjob.yaml
kubectl get cronjobs
```

------------------------------------------------------------------------

## Exercise 5.1 -- ConfigMaps

Create files:

``` bash
echo -n primary > primary
echo -n secondary > secondary
```

Create ConfigMap:

``` bash
kubectl create configmap colors --from-file=primary --from-file=secondary
```

View ConfigMap:

``` bash
kubectl get configmap colors
kubectl get configmap colors -o yaml
```

------------------------------------------------------------------------

## Exercise 5.2 -- Persistent Volumes

``` yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pvvol-1
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  nfs:
    path: /opt/sfw
    server: 10.0.0.1
```

``` bash
kubectl create -f pv.yaml
kubectl get pv
```

### PersistentVolumeClaim

``` yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pv-claim
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 200Mi
```

``` bash
kubectl create -f pvc.yaml
kubectl get pvc
```

------------------------------------------------------------------------

## Exercise 5.4 -- Rolling Updates

``` bash
cd ~/app1
vim simple.py
sudo podman build -t simpleapp:v2 .
sudo podman push $repo/simpleapp:v2
```

Update deployment:

``` bash
kubectl set image deployment/try1 simpleapp=$repo/simpleapp:v2
kubectl rollout status deployment try1
kubectl rollout undo deployment try1
```
