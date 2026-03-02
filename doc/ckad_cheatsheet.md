# CKAD Cheatsheet / Keymap

## Kubernetes Context & Auth 

 Se kubectl non funziona → controlla prima questo.
```
kubectl config view
kubectl config current-context
kubectl config use-context developer
kubectl config get-contexts
User creation
kubectl config set-credentials martin \
--client-certificate=martin.crt \
--client-key=martin.key
Context creation
kubectl config set-context developer \
--cluster=kubernetes \
--user=martin
```
 Verifica kubeconfig path
```
export KUBECONFIG=/root/.kube/config
```
## Control Plane Troubleshooting 

Se cluster sembra morto:

Check API Server container
```
crictl ps | grep kube-apiserver
```
Logs API server:
```
crictl logs <container-id>
```
## Comandi rapidi kubectl

```bash
kubectl get nodes
kubectl get pods
kubectl get svc
kubectl get deployments
kubectl describe pod <nome-pod>
kubectl logs <nome-pod>
kubectl exec -it <nome-pod> -- bash
kubectl apply -f <file.yaml>
kubectl delete -f <file.yaml>
kubectl scale deployment <nome> --replicas=N
kubectl rollout status deployment <nome>
kubectl rollout undo deployment <nome>
```

## Namespace

```bash
kubectl get ns
kubectl create ns dev
kubectl config set-context --current --namespace=dev
```

## ConfigMap / Secret

```bash
kubectl create configmap <nome> --from-literal=key=value
kubectl create secret generic <nome> --from-literal=key=value
kubectl describe cm <nome>
kubectl describe secret <nome>
```

## Pod / Deployment

```bash
kubectl run nginx --image=nginx
kubectl expose pod nginx --type=NodePort --port=80
kubectl get pods -o wide
kubectl get deployments -o wide
```

## Observability / Debug

```bash
kubectl describe pod <nome>
kubectl logs <nome-pod>
kubectl top pod
kubectl get events
```

## Storage

```bash
kubectl get pv
kubectl get pvc
kubectl apply -f pvc.yaml
```

## Metrics (se Metrics Server installato)
```
kubectl top pod
kubectl top node
```
## Health Probes 

- Liveness Probe

  - Restart container se fallisce.
```
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 80
Readiness Probe
```

> Remove Pod from Service.

```
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
Startup Probe
```

> App lente all’avvio.

```
startupProbe:
  httpGet:
    path: /
    port: 8080
```
## Storage Pattern 
 - PV Check

```
kubectl get pv
kubectl get pvc
```
PVC Example
```
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
```
Volume Mount Pod
```
volumeMounts:
  - name: site
    mountPath: /site
```
## Multi-container / Sidecar

```bash
kubectl apply -f pod-sidecar.yaml
kubectl exec -it <nome-pod> -c <nome-container> -- bash
```

## Namespace Management
```
kubectl get ns
kubectl create ns dev
kubectl config set-context --current --namespace=dev
```

## ConfigMap & Secret
- ConfigMap
```
kubectl create configmap app-config --from-literal=ENV=prod
```
- Secret
```
kubectl create secret generic app-secret \
--from-literal=username=admin \
--from-literal=password=123
```

## Helm – Package Manager Kubernetes

Helm serve per gestire chart reusable.

Install chart
```
helm install release-name chart-name
```
List release
```
helm list
```
Upgrade release
```
helm upgrade release-name chart
```
Delete release
```
helm uninstall release-name
```
---
## Kustomize 

Serve per patching YAML senza modificarlo direttamente.

Build manifest
```
kubectl kustomize .
```
Apply kustomize
```
kubectl apply -k .
```
Example structure
```
base/
  deployment.yaml
  service.yaml

overlays/
  dev/
  prod/
```
 Usato per:
   - Change image
   - Patch replica
   - Environment config

## Ingress Controller 

Ingress = routing HTTP cluster-level.

Check ingress
```
kubectl get ingress
```
Example Ingress YAML
```
spec:
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app-service
            port:
              number: 80
```
> Ingress NON espone da solo il service.

> Serve per Ingress controller (nginx, cloud LB, etc)

## NetworkPolicy 

Serve per traffic isolation.

> Default Kubernetes NON blocca traffico.

Example Policy
```
podSelector:
  matchLabels:
    app: frontend

ingress:
- from:
  - podSelector:
      matchLabels:
        app: backend
```
## Workload Types
```
Deployment → Stateless app
kubectl create deployment web --image=nginx
StatefulSet → Database pattern
```
Usato per:

 - Persistent identity

Ordered startup
```
kubectl get sts
```
```
DaemonSet → One pod per node
```
Esempi:

 - Logging agent
 - Monitoring agent
```
kubectl get ds
```
```
Job → Run once
kubectl create job batch-job --image=busybox
CronJob → Scheduled job
```
```
spec:
  schedule: "*/5 * * * *"
```
## RBAC Advanced 

- Role: Namespace scoped.
```
kubectl create role developer-role \
--verb=get,list,create \
--resource=pods
```
- RoleBinding
```
kubectl create rolebinding dev-binding \
--role=developer-role \
--user=martin
```
---

### Useful Diagnostic Chain 

Se qualcosa non funziona:
```
1. kubectl describe pod
2. kubectl logs pod
3. check events
4. check pvc bound
5. check image pull
6. check probe config
```

