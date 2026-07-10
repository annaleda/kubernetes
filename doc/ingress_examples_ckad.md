- [ Home ](../readme.md)   | [ Teoria ](../arguments.md)   | [ Info Exam ](../doc/ckad_exam_strategy.md) | [ Teoria base DNS, IP + Kubernetes ](../doc/dns.md) | [ CoreDNS - CNI k8s ](./k8s_dns_cni_cilium.md)

---

## Ingress — Esempi progressivi (Host, Path, TLS e Debug)

> Un oggetto `Ingress` definisce regole HTTP/HTTPS verso uno o più Service.
>
> Per funzionare è necessario che nel cluster sia presente un **Ingress Controller**.

---

## 1. Ingress base — Un Service

Instrada tutte le richieste verso `web-service:80`.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
```

Risultato:

- `/` → `web-service:80` ✅
- `/app` → `web-service:80` ✅

---

## 2. Routing tramite host

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: host-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
```

Test:

```sh
curl -H "Host: app.example.com" http://INGRESS_IP
```

---

## 3. Routing per path

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: path-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /frontend
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 8080
```

Risultato:

- `/frontend` → frontend ✅
- `/api` → API ✅

---

## 4. Più host

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: multi-host-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: frontend.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
  - host: api.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 8080
```

---

## 5. Default backend

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: default-backend-ingress
spec:
  ingressClassName: nginx
  defaultBackend:
    service:
      name: default-service
      port:
        number: 80
```

Il `defaultBackend` riceve il traffico che non corrisponde ad altre regole.

---

## 6. pathType Prefix

```yaml
- path: /api
  pathType: Prefix
```

Corrisponde a:

- `/api` ✅
- `/api/users` ✅
- `/apiv2` ❌

---

## 7. pathType Exact

```yaml
- path: /login
  pathType: Exact
```

Corrisponde a:

- `/login` ✅
- `/login/` ❌
- `/login/user` ❌

---

## 8. pathType ImplementationSpecific

```yaml
- path: /api
  pathType: ImplementationSpecific
```

Il comportamento dipende dall'Ingress Controller. Al CKAD è normalmente preferibile usare `Prefix` o `Exact`.

---

## 9. Ingress con TLS

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tls-ingress
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - app.example.com
    secretName: app-tls
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
```

Creazione Secret:

```sh
kubectl create secret tls app-tls --cert=tls.crt --key=tls.key
```

---

## 10. Rewrite con ingress-nginx

> Le annotation sono specifiche del controller.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: rewrite-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
    nginx.ingress.kubernetes.io/use-regex: "true"
spec:
  ingressClassName: nginx
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /something(/|$)(.*)
        pathType: ImplementationSpecific
        backend:
          service:
            name: web-service
            port:
              number: 80
```

---

## 11. Porta Service nominata

```yaml
backend:
  service:
    name: web-service
    port:
      name: http
```

---

## 12. IngressClass

```sh
kubectl get ingressclass
```

```yaml
spec:
  ingressClassName: nginx
```

`ingressClassName` sostituisce il vecchio approccio basato sull'annotation `kubernetes.io/ingress.class`.

---

## 13. Creazione imperativa

```sh
kubectl create ingress web-ingress   --class=nginx   --rule="app.example.com/=web-service:80"
```

Generare YAML:

```sh
kubectl create ingress web-ingress   --class=nginx   --rule="app.example.com/=web-service:80"   --dry-run=client -o yaml > ingress.yaml
```

---

## 14. Verifica Ingress

```sh
kubectl get ingress
kubectl get ing
kubectl get ingress -A
kubectl describe ingress web-ingress
kubectl get ingress web-ingress -o yaml
```

---

## 15. Recuperare ADDRESS

```sh
kubectl get ingress web-ingress   -o jsonpath='{.status.loadBalancer.ingress[0].ip}'
```

Oppure, se è presente un hostname:

```sh
kubectl get ingress web-ingress   -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
```

---

## 16. Test con curl e Pod temporaneo

```sh
curl -H "Host: app.example.com" http://INGRESS_IP/api
```

```sh
kubectl run curl-test   --rm -it   --restart=Never   --image=curlimages/curl   -- curl -H "Host: app.example.com" http://INGRESS_IP/api
```

---

## 17. Debug — Ingress senza ADDRESS

```sh
kubectl get ingress
kubectl get ingressclass
kubectl describe ingress web-ingress
kubectl get pods -A | grep -i ingress
```

Possibili cause:

- controller non installato
- `ingressClassName` errato
- controller non pronto
- ambiente locale senza LoadBalancer esterno

---

## 18. Debug — Errore 404

Controllare:

```sh
kubectl describe ingress web-ingress
kubectl get svc
kubectl get endpoints
kubectl get endpointslices
```

Possibili cause:

- host errato
- path errato
- regola non corrispondente
- IngressClass sbagliata

---

## 19. Debug — Errore 502/503

```sh
kubectl get pods
kubectl get svc
kubectl get endpoints
kubectl describe svc web-service
```

Possibili cause:

- Service senza Endpoint
- selector errato
- Pod non Ready
- porta o `targetPort` errata

---

## 20. Relazione Ingress, Service e Pod

```text
Client
   |
   v
Ingress Controller
   |
   v
Service
   |
   v
Pod
```

L'Ingress punta alla `port` del Service. Il Service inoltra verso la `targetPort` del Pod.

---

## 21. Regole importanti

- Serve un Ingress Controller
- `ingressClassName` seleziona il controller
- Ingress e Service backend devono normalmente essere nello stesso namespace
- `Prefix` include i sotto-path
- `Exact` richiede corrispondenza esatta
- TLS usa un Secret nello stesso namespace
- Le annotation dipendono dal controller

---

## 22. Tabella riassuntiva

| Scenario | Controllo |
|---|---|
| Nessun ADDRESS | Controller e IngressClass |
| 404 | Host, path e regole |
| 502/503 | Service, Endpoint e Pod |
| TLS non funziona | Secret, host e certificato |
| Backend errato | Service port e targetPort |

---

## 23. Esempio completo CKAD

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx:1.27
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 80
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: web.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
```

Verifica:

```sh
kubectl get deploy,pod,svc,ingress
kubectl get endpoints web-service
kubectl describe ingress web-ingress
```

---

## Considerazioni generali

- Ingress gestisce routing HTTP e HTTPS
- non sostituisce il Service
- le funzionalità avanzate dipendono dal controller
- l'API Ingress è stabile ma congelata
- Kubernetes raccomanda Gateway API per nuove funzionalità
- per il CKAD Ingress resta un argomento importante

---
