
## Service and Networking
--- 
### 5.1 [Services & Networking](../doc/ckad_services_networking.md)
   - (Service,Type of Service,Port,Ingress,Network Policies)
### 5.2 [Network Policies examples](https://github.com/ahmetb/kubernetes-network-policy-recipes)
---

### 5.2 LoadBalancer

---

Un **Service di tipo LoadBalancer** espone un'applicazione Kubernetes verso l’esterno tramite un IP pubblico fornito dal cloud provider.

Funziona creando automaticamente un **Load Balancer esterno** (solo in ambienti cloud come AWS, Azure, GCP).

Esempi:
 - AWS → crea automaticamente un Elastic Load Balancer (ELB)
 - Azure → crea un Azure Load Balancer
 - GCP → crea un Cloud Load Balancer
---
     
[Service+Type_+Load+Balancer.pdf](https://github.com/user-attachments/files/25584395/Service%2BType_%2BLoad%2BBalancer.pdf)

[load_balancer.yml](https://github.com/user-attachments/files/25584404/load_balancer.yml)
