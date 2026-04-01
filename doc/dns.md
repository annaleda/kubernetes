
## DNS & IP — Fondamenti + Kubernetes

---

###  1. Concetti base

Un indirizzo IP identifica un dispositivo in rete.

- IPv4 → 32 bit → es: 192.168.1.10
- IPv6 → 128 bit → es: 2001:db8::1

---

Il DNS traduce:

nome → indirizzo IP

Esempio:

backend.local → 192.168.1.10

---

### Tipi di record DNS

| Tipo | Descrizione |
|------|------------|
| A | Nome → IPv4 |
| AAAA | Nome → IPv6 |

---

## 2. IPv4 — subnet e assegnazione

### Struttura IPv4

192.168.1.10

Diviso in:

[Network] [Host]

---

### CIDR notation

192.168.1.0/24

 significa:

- primi 24 bit = network
- ultimi 8 bit = host

---

### Esempio completo /24

Network: 192.168.1.0/24

| Tipo | IP |
|------|----|
| Network | 192.168.1.0 |
| First host | 192.168.1.1 |
| Last host | 192.168.1.254 |
| Broadcast | 192.168.1.255 |

 Totale IP: 256  
 Usabili: 254

---

### Assegnazione IP

- Statico
- DHCP
- Kubernetes (CNI)

---

##  3. IPv6 — subnet /64

## Struttura IPv6

2001:db8:abcd:12::1

Diviso in:

[Network prefix] [Interface ID]

---

### CIDR IPv6

2001:db8:abcd:12::/64

 significa:

- primi 64 bit = network
- ultimi 64 bit = host

---

### Esempio /64

Network: 2001:db8:abcd:12::/64

Range:

2001:db8:abcd:12:0000:0000:0000:0000
→
2001:db8:abcd:12:ffff:ffff:ffff:ffff

 Numero IP: 2^64

---

### Esempi IP validi

2001:db8:abcd:12::1  
2001:db8:abcd:12::2  
2001:db8:abcd:12::a1  

---

### Interface ID

- manuale
- EUI-64
- random

---

## 4. DNS ↔ IP

DNS traduce:

nome → IP

Esempio:

backend → 10.96.0.10  
backend → 2001:db8::10  

---

## 5. Kubernetes — DNS e IP

### Pod IP

- assegnato dal CNI
- unico nel cluster
- dinamico

Esempio:

Pod → 10.244.1.5

---

### Service IP

Service → 10.96.0.10

---

### DNS Service

backend.prod.svc.cluster.local → 10.96.0.10

---

### Risoluzione DNS

Pod → CoreDNS → Service → Pod

---

### Headless Service

clusterIP: None

DNS:

backend → [Pod IP]

---

### DNS nei Pod

cat /etc/resolv.conf

search default.svc.cluster.local svc.cluster.local cluster.local

---

### DNS e NetworkPolicy

Permettere porta 53 (UDP/TCP)

---

### Riassunto

- IPv4 → /24
- IPv6 → /64
- DNS → nome → IP
- Kubernetes:
  - Pod IP → CNI
  - Service IP → cluster
  - DNS → CoreDNS
 
---
### [ CoreDNS - CNI k8s ](./k8s_dns_cni_cilium.md)
---
