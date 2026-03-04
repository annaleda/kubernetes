
###  Kube-apiserver (6 esercizi)

## KA-1 — Identificare Endpoint API Server

- Recuperare endpoint del cluster

---
<details>
<summary>Soluzione</summary>
  
```
k cluster-info

# cerca https://

```
</details>

---

## KA-2 — Verificare Certificati

- Esaminare certificati nel kubeconfig
- Obiettivo
  - Identificare campo certificate-authority-data

---
<details>
<summary>Soluzione</summary>
  
```
k config view

# cerca certificate-auth
```
</details>

---

## KA-3 — Abilitare Audit Log (Concettuale)

- Descrivere come si abilita audit logging
nel kube-apiserver

- Obiettivo
  - Identificare flag `--audit-policy-file`
---
<details>
<summary>Soluzione</summary>
  
```
# creare ausit-policy.yaml
apiVersion: audit.k8s.io/v1
kind: Policy
rules:
- level: Metadata


# Avviare kube-apiserver con flag 
# Nel manifest o service configuration del control plane:
# --audit-policy-file=/etc/kubernetes/audit-policy.yaml
 
```
</details>

---

### KA-4 — Admission Controllers

- Identificare admission controller attivi
- Obiettivo
  - Comprendere ruolo di:
    - NamespaceLifecycle
    - LimitRanger
    - ResourceQuota
- Validazione
  - Descrizione teorica o flag API server

---
<details>
<summary>Soluzione</summary>
  
```
kubectl api-versions
cat /etc/kubernetes/manifests/kube-apiserver.yaml

# swe hai accesso al controlplane node
ps aux | grep kube-apiserver
# kube-apiserver -h | grep admission
```
</details>

---

### KA-5 — API Resources

- Elencare tutte le risorse API disponibili

---
<details>
<summary>Soluzione</summary>
  
```
k api-resources
```
</details>

---

### KA-6 — Versione API

- Verificare versione cluster

---
<details>
<summary>Soluzione</summary>
  
```
kubectl version --short  
```
</details>

---
