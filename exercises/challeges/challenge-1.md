## Challenge 1

Deploy the given architecture diagram for implementing a Jekyll SGG.
---

### Build users info

- Build user information for martin in the default kubeconfig file: User=martin, client-key=/root/martin.key, client-cert=/root/martin.crt
- Create a new context called developer in the default kubeconfig file with user=martin and cluster=kubernetes

```bash
k config set-credentials martin --client-key=/root/martin.key --client-certificate=/root/martin.crt
```
```bash
k config set-context developer --cluster=kubernates --user=martin
```
### Role and Rolebinding

- Create a rolebinding=developer-rolebinding, role=developer-role, namenspace=developer
- rolebinding=rolebinding-developer should be associated with user martin
- set context developer with user=martin and cluster=kubernates as the current context

```
k create rolebinding developer-rolebinding --user=martin --roloe=developer-role -n development

k create role developer-role --verb=list,create,delete --resources=pods,services,persistentvolumeclaims -n development
```
### Pod
Pod: jekyll

- Requisiti
  - Name → jekyll
  - Label → run=jekyll
  - InitContainer:
    - Name → copy-jekyll-site
    - Image → gcr.io/kodekloud/customimage/jekyll
    - Command: rm -rf /site/* && jekyll new /site && cd /site && bundle install
  - Container principale:
    - Name → jekyll
    - Image → gcr.io/kodekloud/customimage/jekyll-serve
    - Command: cd /site && bundle install && bundle exec jekyll serve --host 0.0.0.0 --port 4000
  - Volume name → site
  - PVC → jekyll-site
  - MounthPath → site for all type of containers

```
kubectl create pod jekyll \
--image=gcr.io/kodekloud/customimage/jekyll-serve \
--dry-run=client -o yaml > pod.yaml
```

```
vi pod.yaml
```

Pod YAML
```
apiVersion: v1
kind: Pod
metadata:
  name: jekyll
  labels:
    run: jekyll
spec:
  initContainers:
    - name: copy-jekyll-site
      image: gcr.io/kodekloud/customimage/jekyll
      command:
        - sh
        - -c
        - rm -rf /site/* && jekyll new /site && cd /site && bundle install
      volumeMounts:
        - name: site
          mountPath: /site

  containers:
    - name: jekyll
      image: gcr.io/kodekloud/customimage/jekyll-serve
      command:
        - sh
        - -c
        - cd /site && bundle install && bundle exec jekyll serve --host 0.0.0.0 --port 4000
      volumeMounts:
        - name: site
          mountPath: /site

  volumes:
    - name: site
      persistentVolumeClaim:
        claimName: jekyll-site
```

```
k apply -f pod.yaml
```

### Service

- Service – gop-fs-service

Requisiti:
```
Name → jekyll-node-service
Port → 4000
TargetPort → 4000
nodePort  → 30097
```

```
kubectl create service nodeport jekyll-node-service \
--tcp=4000:4000 \
--node-port=30097 \
--dry-run=client -o yaml > service.yaml
```

```
vi service.yaml
```

Service YAML
```
apiVersion: v1
kind: Service
metadata:
  name: jekyll-node-service
spec:
  type: NodePort
  selector:
    run: jekyll
  ports:
    - port: 4000
      targetPort: 4000
      nodePort: 30097
```

```
k apply -f service.yaml
```

### PVC

- Persistent Volume Claim (PVC)
- Requisiti
  

| Campo           | Valore          |
| --------------- | --------------- |
| Name            | `data-pvc`      |
| Namespace       | `development`   |
| Storage Request | `1Gi`           |
| Access Mode     | `ReadWriteMany` |
| Volume Bound    | `data-pv`       |

```
kubectl create pvc data-pvc \
--namespace=development \
--storage=1Gi \
--access-mode=ReadWriteMany \
--dry-run=client -o yaml > pvc.yaml
```

```
vi pvc.yaml
```
PVC Example

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: data-pvc
  namespace: development
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
  storageClassName: local-storage
  volumeName: data-pv
```

```
k apply -f pvc.yaml
```


### PV

- Persistent Volume (PV)
- Requisiti
  
| Campo          | Valore                                                                 |
| -------------- | ---------------------------------------------------------------------- |
| Name           | `data-pv`                                                              |
| Storage        | `1Gi`                                                                  |
| Access Mode    | `ReadWriteMany`                                                        |
| HostPath       | `/web`                                                                 |
| StorageClass   | `local-storage`                                                        |
| Reclaim Policy | (non strettamente richiesto ma tipicamente `Delete` o default del lab) |

```
kubectl get pv data-pv -o yaml > pv.yaml

kubectl create pv --dry-run=client -o yaml > pv.yaml
```

```
vi pv.yaml
```

PV Example
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: data-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  storageClassName: local-storage
  hostPath:
    path: /web
```

```
k apply -f pv.yaml
```
---
### Verifica finale cluster

Test API server:
```
kubectl get nodes
kubectl get pods -A
```
Se funziona → cluster healthy ✅
