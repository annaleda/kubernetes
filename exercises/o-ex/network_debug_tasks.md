- [ Home ](../../readme.md)   | [ Teoria ](../../arguments.md)   | [ Info Exam ](../../doc/ckad_exam_strategy.md)   | [ Home Other Exercises ](../o_exercises.md)

---

### CKAD Network Debug Tasks (20 esercizi)

---

## NET-1 — Pod temporaneo BusyBox

SECTION: APPLICATION OBSERVABILITY AND MAINTENANCE

- Obiettivo
  - Creare un Pod temporaneo BusyBox

---

<details>
<summary>Soluzione</summary>

```sh
kubectl run tmp --rm -it --restart=Never --image=busybox:1.36 -- sh
```

</details>

---

## NET-2 — Pod temporaneo Alpine

- Obiettivo
  - Avviare un Pod Alpine interattivo

---

<details>
<summary>Soluzione</summary>

```sh
kubectl run alpine --rm -it --restart=Never --image=alpine -- sh
```

</details>

---

## NET-3 — Verificare DNS con nslookup

- Service: nginx

---

<details>
<summary>Soluzione</summary>

```sh
kubectl run dns-test --rm -it --restart=Never --image=busybox:1.36 -- nslookup nginx
```

</details>

---

## NET-4 — nslookup FQDN

- Namespace: default
- Service: nginx

---

<details>
<summary>Soluzione</summary>

```sh
kubectl run dns-test --rm -it --restart=Never --image=busybox:1.36 -- \
nslookup nginx.default.svc.cluster.local
```

</details>

---

## NET-5 — wget verso Service

---

<details>
<summary>Soluzione</summary>

```sh
kubectl run wget --rm -it --restart=Never --image=busybox:1.36 -- \
wget -qO- http://nginx
```

</details>

---

## NET-6 — wget verso ClusterIP

---

<details>
<summary>Soluzione</summary>

```sh
kubectl run wget --rm -it --restart=Never --image=busybox:1.36 -- \
wget -qO- http://10.96.0.10
```

</details>

---

## NET-7 — wget su NodePort

---

<details>
<summary>Soluzione</summary>

```sh
kubectl run wget --rm -it --restart=Never --image=busybox:1.36 -- \
wget -qO- http://NODE_IP:NODEPORT
```

</details>

---

## NET-8 — Test connessione HTTPS

---

<details>
<summary>Soluzione</summary>

```sh
kubectl run wget --rm -it --restart=Never --image=busybox:1.36 -- \
wget https://kubernetes.io
```

</details>

---

## NET-9 — Test DNS esterno

---

<details>
<summary>Soluzione</summary>

```sh
kubectl run dns --rm -it --restart=Never --image=busybox:1.36 -- \
nslookup google.com
```

</details>

---

## NET-10 — Ping Pod

---

<details>
<summary>Soluzione</summary>

```sh
kubectl run test --rm -it --restart=Never --image=busybox:1.36 -- \
ping POD_IP
```

</details>

---

## NET-11 — shell in Pod temporaneo

<details>
<summary>Soluzione</summary>

```sh
kubectl run debug --rm -it --restart=Never --image=nicolaka/netshoot -- bash
```

</details>

---

## NET-12 — Test endpoint HTTP

<details>
<summary>Soluzione</summary>

```sh
kubectl run curl --rm -it --restart=Never --image=curlimages/curl -- \
curl http://nginx
```

</details>

---

## NET-13 — Recuperare pagina index

<details>
<summary>Soluzione</summary>

```sh
kubectl run wget --rm -it --restart=Never --image=busybox -- \
wget -O- http://nginx
```

</details>

---

## NET-14 — Verificare risoluzione DNS Pod

<details>
<summary>Soluzione</summary>

```sh
kubectl exec nginx -- nslookup kubernetes.default
```

</details>

---

## NET-15 — Verificare variabili DNS

<details>
<summary>Soluzione</summary>

```sh
kubectl exec nginx -- cat /etc/resolv.conf
```

</details>

---

## NET-16 — Pod che rimane in esecuzione

<details>
<summary>Soluzione</summary>

```sh
kubectl run debug --image=busybox:1.36 -- sleep 3600
```

</details>

---

## NET-17 — Eliminare Pod temporaneo

<details>
<summary>Soluzione</summary>

```sh
kubectl delete pod debug
```

</details>

---

## NET-18 — Verificare Endpoint Service

<details>
<summary>Soluzione</summary>

```sh
kubectl get endpoints nginx
```

</details>

---

## NET-19 — Test connessione TCP

<details>
<summary>Soluzione</summary>

```sh
kubectl run netshoot --rm -it --restart=Never --image=nicolaka/netshoot -- \
nc -zv nginx 80
```

</details>

---

## NET-20 — Pod temporaneo con override command

<details>
<summary>Soluzione</summary>

```sh
kubectl run toolbox \
--rm -it \
--restart=Never \
--image=busybox:1.36 \
-- sh
```

</details>

---
