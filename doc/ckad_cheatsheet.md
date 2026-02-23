# CKAD Cheatsheet / Keymap

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

## Multi-container / Sidecar

```bash
kubectl apply -f pod-sidecar.yaml
kubectl exec -it <nome-pod> -c <nome-container> -- bash
```

---


