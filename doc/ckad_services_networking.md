# CKAD Services & Networking

## Teoria

---

## Service

Un Service espone uno o più Pod tramite **label selector**.

Permette:
- Comunicazione stabile tra Pod
- Accesso interno o esterno al cluster
- Load balancing automatico tra Pod

Il Service crea un IP virtuale stabile (ClusterIP).

---

## Tipi di Service

### ClusterIP (default)

- Accessibile solo **all’interno del cluster**
- Usato per comunicazione interna tra microservizi
- Non esposto all’esterno

Esempio:
```yaml
spec:
  type: ClusterIP
```

---

### NodePort

- Espone il servizio su una porta del nodo
- Accessibile dall’esterno tramite:

```
<NodeIP>:<NodePort>
```

- Range porte default: 30000–32767

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

## Ingress

Permette routing HTTP/HTTPS basato su:

- Hostname
- Path

Richiede un **Ingress Controller** (non incluso di default).

Esempio di regola:

```yaml
spec:
  rules:
    - host: myapp.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: web
                port:
                  number: 80
```

---

## Ingress vs Service

| Service | Ingress |
|----------|----------|
| Layer 4 (TCP/UDP) | Layer 7 (HTTP/HTTPS) |
| Espone singolo servizio | Routing multiplo |
| Tipo ClusterIP/NodePort/LoadBalancer | Richiede Ingress Controller |

---

## Network Policies

Permettono di controllare il traffico tra Pod.

- Agiscono a livello di Pod
- Basate su label selector
- Permettono:
  - Ingress rules
  - Egress rules

Se esiste una NetworkPolicy, il traffico non permesso viene bloccato.

---

## Comunicazione tra Pod

- Pod nello stesso namespace → comunicano tramite Service
- Container nello stesso Pod → comunicano via `localhost`
- Namespace diversi → usare FQDN completo

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


