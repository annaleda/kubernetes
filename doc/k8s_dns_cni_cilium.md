- [ Home ](../readme.md)   | [ Teoria ](../arguments.md)   | [ Info Exam ](./ckad_exam_strategy.md) | [Teoria base DNS, ip + Kubernates](./dns.md)


# 🌐 Kubernetes DNS & Networking — CoreDNS, CNI, Cilium

---

## 🔹 1. CoreDNS — Teoria

CoreDNS è il **DNS server interno di Kubernetes**.

Traduce:

nome → IP

Esempio:

backend.default.svc.cluster.local → 10.96.0.10

---

## Dove gira

Namespace:

kube-system

Comandi:

kubectl get pods -n kube-system  
kubectl get svc -n kube-system  

Servizio:

kube-dns

---

## Come funziona

Flusso:

Pod → CoreDNS → Kubernetes API → Service → Pod

---

## Da dove prende i dati

- Services
- Endpoints / EndpointSlice

---

## Corefile (configurazione)

kubectl get configmap coredns -n kube-system -o yaml

Esempio:

.:53 {
    errors
    health
    kubernetes cluster.local in-addr.arpa ip6.arpa {
        pods insecure
        fallthrough in-addr.arpa ip6.arpa
    }
    forward . /etc/resolv.conf
    cache 30
}

---

## Plugin principali

- kubernetes → risolve servizi/pod
- forward → DNS esterno
- cache → caching
- health → health check
- errors → logging

---

## Tipi di record

Service:

backend → 10.96.x.x

Headless:

backend → [Pod IP]

---

## DNS search

cat /etc/resolv.conf

search default.svc.cluster.local svc.cluster.local cluster.local

---

## DNS esterno

CoreDNS inoltra con:

forward → DNS esterno

---

## Problemi comuni

- DNS non funziona → CoreDNS down o porta 53 bloccata
- nslookup ok ma curl no → rete/service issue

---

## 🔹 2. CNI — Container Network Interface

Il CNI è il componente che gestisce:

- assegnazione IP ai Pod
- routing tra Pod
- networking cluster

---

Quando crei un Pod:

1. Il CNI assegna un IP
2. Configura networking
3. Permette comunicazione tra Pod

---

## Plugin comuni

- Calico
- Cilium
- Flannel
- Weave

---

## 🔹 3. Cilium — Teoria

Cilium è un CNI avanzato basato su:

> eBPF (Extended Berkeley Packet Filter)

---

## Cosa fa Cilium

- Networking (come altri CNI)
- Security avanzata
- NetworkPolicy avanzate
- Observability (monitoring traffico)

---

> Non usa iptables  
> Usa eBPF (kernel Linux)

Vantaggi:

- più veloce
- più efficiente
- più osservabilità

---

## Funzionalità principali

### 🔸 Networking

- assegna IP ai Pod
- routing tra Pod

---

### 🔸 NetworkPolicy

- supporta policy standard Kubernetes
- supporta policy avanzate (L7)

---

### 🔸 Security

- controlli basati su identità (non solo IP)
- micro-segmentation

---

### 🔸 Observability

- Hubble (tool Cilium)
- visibilità traffico

---

## Flusso con Cilium

Pod → eBPF → Pod

(no iptables)

---

## Differenza con altri CNI

| Feature | Cilium | Calico |
|--------|--------|--------|
| Tecnologia | eBPF | iptables |
| Performance | alta | media |
| Observability | avanzata | base |
| L7 policy | sì | limitato |

---

## 🔹 4. DNS + CNI + Kubernetes

---

## Flusso completo

Pod
 ↓ DNS query
CoreDNS
 ↓
Service IP
 ↓
CNI (Cilium / Calico)
 ↓
Pod target

---

## Ruoli

| Componente | Ruolo |
|-----------|------|
| CoreDNS | risolve nomi |
| Service | espone IP |
| CNI | gestisce rete |
| Pod | esegue app |

---

### Riassunto finale

- CoreDNS → DNS interno Kubernetes
- CNI → networking Pod
- Cilium → CNI avanzato (eBPF)
- Service → IP stabile
- DNS → nome → Service IP
