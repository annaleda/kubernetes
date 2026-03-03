
###  Services (6 esercizi)

## SVC-1 — ClusterIP

- Creare Deployment `web-svc`  
  - Image: nginx
  - Replicas: 3
- Creare Service `web-clusterip`
  - Type: ClusterIP
  - Port: 80
- Validazione
  - Service ha endpoint
  - kubectl get endpoints
---
<details>
<summary>Soluzione</summary>
  
```
k create deploy web-svc --image=nginx --replicas=3
k expose deploy web-svc --name web-clusterip --type=ClusterIP --port=80
```
</details>

---

## SVC-2 — NodePort

- Creare Service `web-nodeport`
- Specifiche
  - Type: NodePort
  - Porta interna: 81
  - NodePort manuale: 30067
- Validazione
  - kubectl get svc mostra NodePort

---
<details>
<summary>Soluzione</summary>
  
```
k expose deploy web-svc --name web-nodeport --type=NodePort --port=81 --dry-run=client -o yaml > svc2.yaml

vi svc2.yaml

apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: web-svc
  name: web-nodeport
spec:
  ports:
  - port: 81
    nodePort: 30067
    targetPort: 81
  selector:
    app: web-svc
  type: NodePort
status:
  loadBalancer: {}
```
</details>

---

## SVC-3 — LoadBalancer

- Creare Service `web-lb`
  - Type: LoadBalancer
- Validazione
  - External IP assegnato (se supportato)
---
<details>
<summary>Soluzione</summary>
  
```
apiVersion: v1
kind: Service
metadata:
  name: web-lb
spec:
  type: LoadBalancer
  selector:
    env: prod
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80



```
</details>

---

### SVC-4 — Headless Service

- Creare Service `headless-svc`
- Specifiche
  - ClusterIP: None
- Collegare a StatefulSet
- Validazione
  - DNS restituisce IP dei singoli pod

---
<details>
<summary>Soluzione</summary>
  
```  
```
</details>

---

### SVC-5 — Service Selector Mismatch

- Creare Service con selector errato
- Obiettivo
  - Debug e correzione
- Validazione
  - Endpoints presenti dopo fix
---
<details>
<summary>Soluzione</summary>
  
```  
```
</details>

---

### SVC-6 — Port Mapping

- Deployment `multi-port-app`
  - Container espone 8080
- Service
  - Port: 80
  - TargetPort: 8080
- Validazione
  - Mapping corretto
---
<details>
<summary>Soluzione</summary>
  
```  
```
</details>

---
