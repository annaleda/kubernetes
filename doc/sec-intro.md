

### Security Intro – TLS Certificates in Kubernetes
- Overview

Kubernetes protegge tutte le comunicazioni interne usando TLS (Transport Layer Security).

Ogni componente del cluster comunica in modo cifrato e autenticato tramite certificati digitali.

---
Il modello è basato su:

 - Certificate Authority (CA)
 - Server Certificates

## Client Certificates

  - Mutual TLS (mTLS)
  - Architettura della Comunicazione Sicura

Al centro di tutto troviamo:

kube-apiserver

È il cuore del cluster.

Tutti i componenti parlano con il kube-apiserver.
Solo il kube-apiserver comunica direttamente con etcd.

| Componente              | Comunica con   | Tipo Certificato |
| ----------------------- | -------------- | ---------------- |
| kube-apiserver          | etcd           | client cert      |
| etcd                    | kube-apiserver | server cert      |
| kube-scheduler          | kube-apiserver | client cert      |
| kube-controller-manager | kube-apiserver | client cert      |
| kubelet                 | kube-apiserver | client cert      |
| kube-proxy              | kube-apiserver | client cert      |


<img width="984" height="621" alt="Immagine 2026-03-02 184857" src="https://github.com/user-attachments/assets/54ded85a-48b2-4f9c-892e-3f165232f8e7" />

- Mutual TLS (mTLS)

Kubernetes utilizza mTLS (mutual TLS).

Significa che:

 - Il server verifica l'identità del client
 - Il client verifica l'identità del server

> Non è solo HTTPS semplice → è autenticazione bidirezionale.
---
- Posizione dei Certificati

Nei cluster creati con kubeadm, i certificati si trovano in:

/etc/kubernetes/pki/

Esempi comuni:

    ca.crt
    ca.key
    apiserver.crt
    apiserver.key
    etcd/ca.crt
    etcd/server.crt
  
- Ruolo della Certificate Authority (CA)

La CA:

- Firma tutti i certificati del cluster
- Garantisce fiducia tra componenti
- Permette l’autenticazione forte

> Se la CA viene compromessa → tutto il cluster è compromesso.
---
- Tipi di Certificati nel Cluster
  1. CA Certificate

  > Firma tutti gli altri certificati.

  2. API Server Certificate

  Usato per:
    - Esporre HTTPS
    - Autenticare kubelet
    - Autenticare kubectl

  3. Client Certificates

  Usati da:
  
    - scheduler
    - controller-manager
    - kubelet
    - admin user

  4. etcd Certificates

  Proteggono:
  
    - Comunicazione interna tra membri etcd
    - Comunicazione tra API server e etcd
---
> Cosa Succede Se i Certificati Scadono?

Possibili sintomi:

  - kubectl non riesce a connettersi
  - kubelet non si registra
  - API server non comunica con etcd
  - Pod non vengono schedulati
---
- Comandi Utili per Troubleshooting

Verificare certificati:
```
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout
```
Controllare data di scadenza:
```
openssl x509 -enddate -noout -in apiserver.crt
```
Verificare stato kubelet:
```
systemctl status kubelet
```
Controllare certificati con kubeadm:
```
kubeadm certs check-expiration
```
---
> Nessun componente parla direttamente con etcd tranne il kube-apiserver.

TLS nel cluster garantisce:

  - Confidenzialità (dati cifrati)
  - Integrità (nessuna manomissione)
  - Autenticazione forte
  - Riduzione attacchi Man-in-the-Middle

- Riassunto Finale

Kubernetes usa TLS ovunque
È basato su mTLS
kube-apiserver è il centro delle comunicazioni
etcd è protetto da certificati dedicati
La CA è il punto di fiducia del cluster
Se vuoi posso aggiungerti anche una sezione:

---

Dimmi tu quanto vuoi spingere in profondità 😈
