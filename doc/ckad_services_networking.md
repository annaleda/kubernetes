- [ Home ](../readme.md)   | [ Teoria ](../arguments.md)   | [ Info Exam ](./ckad_exam_strategy.md)   | [ Teoria Services and Networking](../arg/fifth_arg.md)   |
---
### Services & Networking

---

## Service

Un Service espone uno o più Pod tramite **label selector**.

Permette:
- Comunicazione stabile tra Pod
- Accesso interno o esterno al cluster
- Load balancing automatico tra Pod

> Quando viene specificato il selector, il service crea in autonomia l'Endpoints, se il selector non viene specificato Endpoints va creato manualmente.

<img width="648" height="198" alt="Immagine 2026-02-26 125012" src="https://github.com/user-attachments/assets/bd0865a9-370e-4dd6-ad59-c6b79372f513" />


Il Service crea un IP virtuale stabile (ClusterIP).

---

## Tipi di Service

| Tipo         | Espone             | IP | Uso                     |
| ------------ | ------------------ | -- | ----------------------- |
| ClusterIP    | Interno            | ✅  | Comunicazione interna   |
| NodePort     | Esterno (via nodo) | ✅  | Test / accesso semplice |
| LoadBalancer | Esterno (cloud)    | ✅  | Produzione              |
| Headless     | DNS diretto        | ❌  | Stateful                |
| ExternalName | Esterno (DNS)      | ❌  | Alias                   |


<img width="1159" height="577" alt="Immagine 2026-02-24 122325" src="https://github.com/user-attachments/assets/74725b98-3bd5-49da-b0a0-6fa81c2a0e45" />

### ClusterIP (default)

- Accessibile solo **all’interno del cluster**
- Usato per comunicazione interna tra microservizi
- Non esposto all’esterno

Esempio:
```yaml
spec:
  type: ClusterIP
```
<img width="1214" height="498" alt="Immagine 2026-02-24 123447" src="https://github.com/user-attachments/assets/7a272354-8bfc-4d97-a209-b29dc30acf18" />

---

### NodePort

- Espone il servizio su una porta del nodo
- Accessibile dall’esterno tramite:

```
<NodeIP>:<NodePort>
```

- Range porte default: 30000–32767

<img width="1248" height="577" alt="Immagine 2026-02-24 122659" src="https://github.com/user-attachments/assets/e0fdd392-a7b3-48b8-a947-5c43e33fe149" />

Esempio:
```yaml
spec:
  type: NodePort
```

---

### LoadBalancer

- Espone il servizio esternamente
- Crea un Load Balancer nel cloud provider
- Non funziona nativamente su cluster locali (Kind/Minikube) senza configurazioni extra

Esempio:
```yaml
spec:
  type: LoadBalancer
```

---

### Headless Service

- `clusterIP: None`
- Non crea IP virtuale
- Usato con StatefulSet o per DNS diretto ai Pod

Esempio:
```yaml
spec:
  clusterIP: None
```

---

### ExternalName

- Non espone Pod
- Non crea ClusterIP
- Non crea Endpoints
- Funziona come **alias DNS**

Serve per puntare a un servizio **esterno al cluster**.

---


Kubernetes crea un record DNS:

<service-name> → externalName

---

Esempio

```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-db
spec:
  type: ExternalName
  externalName: mydatabase.example.com
```
## Service Discovery & DNS Interno

Kubernetes fornisce DNS interno automatico.

Formato DNS:

```
<service-name>.<namespace>.svc.cluster.local
```

Esempio:
```
web.default.svc.cluster.local
```

I Pod possono comunicare semplicemente usando:

```
http://web
```

(se nello stesso namespace)

---

## Port vs TargetPort vs NodePort

- `port` → porta esposta dal Service
- `targetPort` → porta del container
- `nodePort` → porta esposta sul nodo (solo NodePort)

Esempio:

```yaml
ports:
  - port: 80
    targetPort: 8080
    nodePort: 30007
```
---

## Comunicazione tra Pod

- Pod nello stesso namespace → comunicano tramite Service
- Container nello stesso Pod → comunicano via `localhost`
- Namespace diversi → usare FQDN completo

---

## Ingress(entrata) and Egress(uscita)

Ci sono due tipi di traffico: in entrata e in uscita.

Le NetworkPolicies permettono:
  - Ingress rules
  - Egress rules


<img width="722" height="524" alt="Immagine 2026-02-24 151606" src="https://github.com/user-attachments/assets/d2c0f223-a871-44eb-9246-92268c09e755" />

---

## Network Policies

Permettono di controllare il traffico tra Pod.

- Agiscono a livello di Pod
- Basate su label selector
- Permettono:
  - Ingress rules
  - Egress rules

Se esiste una NetworkPolicy, il traffico non permesso viene bloccato.

<img width="551" height="612" alt="Immagine 2026-02-24 151906" src="https://github.com/user-attachments/assets/32ccc5ce-8222-456a-844b-1ee6822b379a" />


<img width="1078" height="605" alt="Immagine 2026-02-24 152156" src="https://github.com/user-attachments/assets/afd85c25-31ea-49dd-88df-35990a86b79b" />


<img width="1059" height="558" alt="Immagine 2026-02-24 152327" src="https://github.com/user-attachments/assets/1e3038a6-2307-4dfc-aeab-48a893a6da1d" />

---

### [Network Policy Patterns](./networkpolicy_cheatsheet.md)

---

## Ingress

Permette routing HTTP/HTTPS basato su:

- Hostname
- Path

Richiede un **Ingress Controller** (non incluso di default). [Esempio: NGINX]

Esempio di regola:

esempio service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
  type: ClusterIP
```
esempio ingress
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress

spec:
  rules:
    - host: myapp.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: nginx-service # service creato in precedenza
                port:
                  number: 80
```
<img width="1012" height="657" alt="Immagine 2026-02-24 152804" src="https://github.com/user-attachments/assets/b425ffa4-9258-46f3-8a2b-d7b83a718e08" />

Flusso Ingress

```yaml
Internet
   ↓
Ingress Controller (Proxy) [Esempio: NGINX]
   ↓
Ingress Resource (Rules)
   ↓
Service
   ↓
Pod
```

---

## Ingress vs Service

| Service | Ingress |
|----------|----------|
| Layer 4 (TCP/UDP) | Layer 7 (HTTP/HTTPS) |
| Espone singolo servizio | Routing multiplo |
| Tipo ClusterIP/NodePort/LoadBalancer | Richiede Ingress Controller |

---

| Feature             | Ingress           | NetworkPolicy    |
| ------------------- | ----------------- | ---------------- |
| Livello             | Application Layer | Network Layer    |
| Uso                 | Traffico esterno  | Traffico interno |
| Routing HTTP        | ✅                 | ❌                |
| Security Pod-to-Pod | ❌                 | ✅                |
| Richiede Controller | ✅                 | Optional         |
| Blocca traffico     | No                | Sì               |

---

## Esercizi

1. Esporre un Deployment con ClusterIP.
2. Esporre un Deployment con NodePort.
3. Convertire un Service da ClusterIP a NodePort.
4. Creare un Headless Service.
5. Installare Ingress controller su Kind.
6. Creare regole di routing basate su host.
7. Creare regole di routing basate su path.
8. Scrivere una NetworkPolicy che permetta traffico solo da Pod con label specifica.

---


