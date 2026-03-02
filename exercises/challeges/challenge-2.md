
## Challenge 2

---

### kube-apiserver non è healthy

Fix kube-apiserver. Make sure it's running and healthy.

Controlla lo stato dei componenti:

```bash
crictl ps -a | grep kube-apiserver
```
Se il pod è in CrashLoopBackOff:
```
crictl logs <container_id>
```
Verifica che esista il file certificato:
```
/etc/kubernetes/pki/ca.crt
```
Se manca, ricrea o copia il certificato corretto.

 - Configurazione kubeconfig
 - Parametri richiesti
```
kubeconfig → /root/.kube/config

User → kubernetes-admin

Cluster server port → 6443
```

### Configurazione

Imposta il cluster:
```
kubectl config set-cluster kubernetes \
--server=https://127.0.0.1:6443
```
Configura user:
```
kubectl config set-credentials kubernetes-admin \
--client-certificate=/etc/kubernetes/pki/apiserver.crt \
--client-key=/etc/kubernetes/pki/apiserver.key
```
Crea e usa context:
```
kubectl config set-context developer \
--cluster=kubernetes \
--user=kubernetes-admin
```
```
kubectl config use-context developer
```
### Errori
- Errore certificati non trovati
- Errore unable to read client-cert /root/.kube/admin.crt

Usa i certificati presenti nel sistema:
```
/etc/kubernetes/pki/
```
E aggiorna kubeconfig user credentials.

---

### Node readiness – node01 schedulable
- Verifica node status

```
kubectl get nodes
```
Deve mostrare:
```
node01   Ready
```

Se node01 è cordoned:
```
kubectl uncordon node01
```
### CoreDNS deployment image
- Richiesto
- registry.k8s.io/coredns/coredns:v1.8.6
```
kubectl -n kube-system set image deployment/coredns \
coredns=registry.k8s.io/coredns/coredns:v1.8.6
```
### Pod
```
Pod – gop-file-server
Name: gop-file-server
Image: kodekloud/fileserver
Mount path: /web
Volume name: data-store
PVC utilizzata: data-pvc
Label: run=gop-file-server
```
Pod YAML
```
apiVersion: v1
kind: Pod
metadata:
  name: gop-file-server
  labels:
    run: gop-file-server
spec:
  containers:
    - name: gop-file-server
      image: kodekloud/fileserver
      volumeMounts:
        - name: data-store
          mountPath: /web
  volumes:
    - name: data-store
      persistentVolumeClaim:
        claimName: data-pvc
```

### Service

- Service – gop-fs-service

Requisiti:
```
Name → gop-fs-service
Port → 8080
TargetPort → 8080
```
Service YAML
```
apiVersion: v1
kind: Service
metadata:
  name: gop-fs-service
spec:
  type: NodePort
  selector:
    run: gop-file-server
  ports:
    - port: 8080
      targetPort: 8080
      nodePort: 30097
```
### PVC

- Persistent Volume Claim (PVC)
- Requisiti
  
```
Name → data-pvc

Namespace → development

Storage request → 1Gi

AccessMode → ReadWriteMany

VolumeName → data-pv
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
  volumeName: data-pv
```
### PV

- Persistent Volume (PV)
- Requisiti
```
Name → data-pv

AccessMode → ReadWriteMany

Storage → 1Gi

HostPath → /web
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
  hostPath:
    path: /web
```
---
### Verifica finale cluster

Test API server:
```
kubectl get nodes
kubectl get pods -A
```
Se funziona → cluster healthy ✅
