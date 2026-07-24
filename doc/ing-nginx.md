# Ingress NGINX: path, rewrite e annotation

## 1. API Kubernetes e comportamento del controller

Un manifest `Ingress` contiene due tipi di configurazione:

```text
Campi standard Kubernetes
├── ingressClassName
├── rules
├── host
├── path
├── pathType
├── backend
└── tls

Configurazioni specifiche del controller
└── metadata.annotations
```

Campi come `rules`, `path`, `pathType`, `backend` e `ingressClassName` appartengono all’API Kubernetes.

Annotation come:

```yaml
nginx.ingress.kubernetes.io/rewrite-target
nginx.ingress.kubernetes.io/use-regex
nginx.ingress.kubernetes.io/ssl-redirect
```

non fanno parte del comportamento generico di Kubernetes: sono interpretate specificamente dal controller **ingress-nginx**. Un altro Ingress Controller potrebbe ignorarle oppure utilizzare annotation differenti.

---

## 2. Struttura base di un Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: applications-ingress
  namespace: green-space
spec:
  ingressClassName: nginx
  rules:
    - host: apps.example.com
      http:
        paths:
          - path: /app1
            pathType: Prefix
            backend:
              service:
                name: app-wear-service
                port:
                  number: 8080

          - path: /app2
            pathType: Prefix
            backend:
              service:
                name: app-video-service
                port:
                  number: 8080
```

Il controller selezionato tramite:

```yaml
ingressClassName: nginx
```

deve essere associato a una `IngressClass` compatibile con il controller ingress-nginx. `ingressClassName` è il metodo moderno che sostituisce la vecchia annotation `kubernetes.io/ingress.class`.

Flusso:

```text
GET /app1
    ↓
Ingress rule /app1
    ↓
app-wear-service:8080
    ↓
EndpointSlice
    ↓
Pod app-wear
```

---

## 3. `pathType`

Ogni path deve avere un `pathType`.

I valori principali sono:

### `Exact`

Il path deve coincidere esattamente.

```yaml
path: /app1
pathType: Exact
```

Corrisponde a:

```text
/app1 ✅
/app1/ ❌
/app1/test ❌
```

---

### `Prefix`

Il path viene interpretato come prefisso, separato per elementi della URL.

```yaml
path: /app1
pathType: Prefix
```

Corrisponde normalmente a:

```text
/app1 ✅
/app1/ ✅
/app1/test ✅
```

Non va interpretato semplicemente come una ricerca testuale del prefisso.

---

### `ImplementationSpecific`

Il significato viene deciso dall’Ingress Controller.

```yaml
pathType: ImplementationSpecific
```

Viene spesso usato con ingress-nginx quando il path contiene espressioni regolari.

```yaml
path: /app1(/|$)(.*)
pathType: ImplementationSpecific
```

---

## 4. Routing senza rewrite

Consideriamo:

```yaml
- path: /app1
  pathType: Prefix
  backend:
    service:
      name: app-wear-service
      port:
        number: 8080
```

Senza annotation di rewrite:

```text
Richiesta del client:
GET /app1/products

Richiesta ricevuta dal backend:
GET /app1/products
```

NGINX sceglie il backend corretto, ma normalmente conserva il path originale.

Questo funziona solo se l’applicazione backend sa gestire URL che iniziano con `/app1`.

---

## 5. `rewrite-target`

L’annotation:

```yaml
nginx.ingress.kubernetes.io/rewrite-target: /
```

indica a ingress-nginx di modificare il path prima di inviare la richiesta al backend.

Esempio:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: applications-ingress
  namespace: green-space
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
    - http:
        paths:
          - path: /app1
            pathType: Prefix
            backend:
              service:
                name: app-wear-service
                port:
                  number: 8080

          - path: /app2
            pathType: Prefix
            backend:
              service:
                name: app-video-service
                port:
                  number: 8080
```

Concettualmente:

```text
GET /app1
    ↓ rewrite
GET /
    ↓
app-wear-service
```

e:

```text
GET /app2
    ↓ rewrite
GET /
    ↓
app-video-service
```

L’annotation si trova in `metadata.annotations`, quindi appartiene all’intero oggetto Ingress, non a un singolo elemento della lista `paths`.

---

## 6. Attenzione a `rewrite-target: /`

Con:

```yaml
nginx.ingress.kubernetes.io/rewrite-target: /
```

una richiesta più lunga può perdere la parte successiva del percorso.

Esempio concettuale:

```text
Client:
GET /app1/products

Backend:
GET /
```

Questa configurazione è adeguata quando si vuole semplicemente esporre la root di due applicazioni tramite prefissi diversi.

Può invece essere sbagliata se si vuole conservare:

```text
/products
```

In quel caso servono regex e capture group.

---

## 7. Rewrite con regex e capture group

Per eliminare soltanto il prefisso `/app1` e conservare il resto del path:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app1-ingress
  namespace: green-space
  annotations:
    nginx.ingress.kubernetes.io/use-regex: "true"
    nginx.ingress.kubernetes.io/rewrite-target: /$2
spec:
  ingressClassName: nginx
  rules:
    - host: apps.example.com
      http:
        paths:
          - path: /app1(/|$)(.*)
            pathType: ImplementationSpecific
            backend:
              service:
                name: app-wear-service
                port:
                  number: 8080
```

Il path contiene due gruppi:

```text
/app1(/|$)(.*)
     └─$1 └─$2
```

* `$1` intercetta `/` oppure la fine della stringa;
* `$2` intercetta tutto quello che viene dopo `/app1`.

Esempi:

```text
GET /app1
→ backend GET /

GET /app1/
→ backend GET /

GET /app1/products
→ backend GET /products

GET /app1/products/42
→ backend GET /products/42
```

`use-regex: "true"` abilita l’interpretazione regex dei path da parte di ingress-nginx.

---

## 8. Effetto delle regex sull’intero host

Con ingress-nginx bisogna prestare attenzione: l’uso di `use-regex` può influire sulla modalità con cui vengono trattati tutti i path relativi allo stesso host, anche se sono distribuiti su oggetti Ingress differenti.

Per esempio:

```yaml
metadata:
  annotations:
    nginx.ingress.kubernetes.io/use-regex: "true"
```

su un Ingress relativo a:

```text
apps.example.com
```

può portare ingress-nginx a trattare come regex anche altri path dello stesso host.

Per questo conviene evitare di mescolare senza necessità:

```text
path Exact
path Prefix
path regex
```

sullo stesso host e su più Ingress differenti.

---

## 9. `ssl-redirect`

Annotation:

```yaml
nginx.ingress.kubernetes.io/ssl-redirect: "false"
```

disabilita il redirect automatico da HTTP a HTTPS per quell’Ingress.

Esempio:

```yaml
metadata:
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
```

Con questa configurazione:

```text
http://apps.example.com/app1
```

rimane HTTP.

Senza questa disabilitazione, quando l’Ingress contiene una configurazione TLS, ingress-nginx normalmente forza il redirect verso HTTPS.

---

## 10. Ingress con TLS

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: applications-ingress
  namespace: green-space
spec:
  ingressClassName: nginx

  tls:
    - hosts:
        - apps.example.com
      secretName: apps-tls

  rules:
    - host: apps.example.com
      http:
        paths:
          - path: /app1
            pathType: Prefix
            backend:
              service:
                name: app-wear-service
                port:
                  number: 8080
```

Il Secret deve contenere certificato e chiave TLS.

Concettualmente:

```text
Client HTTPS
    ↓
Ingress NGINX termina TLS
    ↓
Traffico HTTP interno
    ↓
Service applicativo
```

La terminazione TLS sul controller non implica automaticamente che anche il backend utilizzi HTTPS. Normalmente NGINX riceve HTTPS dal client e comunica in HTTP con il Service applicativo.

---

## 11. `force-ssl-redirect`

Annotation:

```yaml
nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
```

serve a forzare il redirect HTTPS anche in scenari in cui l’Ingress non contiene direttamente un blocco TLS.

Differenza concettuale:

```text
ssl-redirect
└── controlla il redirect associato alla configurazione TLS dell’Ingress

force-ssl-redirect
└── forza HTTPS anche senza un normale blocco TLS sull’Ingress
```

Il comportamento esatto può dipendere anche dalla ConfigMap globale del controller.

---

## 12. Annotation e valore booleano

I valori delle annotation sono stringhe.

Corretto:

```yaml
annotations:
  nginx.ingress.kubernetes.io/ssl-redirect: "false"
  nginx.ingress.kubernetes.io/use-regex: "true"
```

È preferibile usare le virgolette:

```yaml
"true"
"false"
```

perché `metadata.annotations` accetta coppie chiave-valore testuali.

---

## 13. Stesso host e stesso path

Due Ingress gestiti dallo stesso ingress-nginx non dovrebbero dichiarare la stessa combinazione:

```text
IngressClass + host + path
```

Esempio conflittuale:

```text
Ingress A:
host: apps.example.com
path: /app1

Ingress B:
host: apps.example.com
path: /app1
```

Il validating webhook può rifiutare il secondo oggetto perché la stessa route sarebbe assegnata a due backend differenti.

Sono invece normalmente distinguibili:

```text
Stesso host, path diversi:

apps.example.com/app1
apps.example.com/app2
```

oppure:

```text
Host diversi, stesso path:

wear.example.com/
video.example.com/
```

Le annotation non permettono di evitare il conflitto: prima viene determinata la route tramite host e path, poi vengono applicati comportamenti come rewrite e redirect.

---

## 14. Rewrite e conflitto tra path

Questi due Ingress rimangono in conflitto:

```text
Ingress A:
host: apps.example.com
path: /app1
rewrite-target: /

Ingress B:
host: apps.example.com
path: /app1
rewrite-target: /products
```

Anche se il `rewrite-target` è differente, la route in ingresso è sempre:

```text
apps.example.com/app1
```

Il rewrite riguarda il path inviato al backend, non il criterio iniziale con cui NGINX sceglie la regola.

Ordine logico:

```text
1. Ricezione richiesta
2. Selezione tramite host e path
3. Selezione del backend
4. Applicazione del rewrite
5. Proxy verso il Service
```

---

## 15. Esempio completo con due applicazioni

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: applications-ingress
  namespace: green-space
  annotations:
    nginx.ingress.kubernetes.io/use-regex: "true"
    nginx.ingress.kubernetes.io/rewrite-target: /$2
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  ingressClassName: nginx

  rules:
    - host: apps.example.com
      http:
        paths:
          - path: /app1(/|$)(.*)
            pathType: ImplementationSpecific
            backend:
              service:
                name: app-wear-service
                port:
                  number: 8080

          - path: /app2(/|$)(.*)
            pathType: ImplementationSpecific
            backend:
              service:
                name: app-video-service
                port:
                  number: 8080
```

Comportamento:

```text
GET /app1
→ app-wear-service
→ backend riceve /

GET /app1/catalog
→ app-wear-service
→ backend riceve /catalog

GET /app2
→ app-video-service
→ backend riceve /

GET /app2/videos/10
→ app-video-service
→ backend riceve /videos/10
```

---

## 16. Namespace dei backend

Un Ingress instrada normalmente verso Service presenti nel suo stesso namespace.

Esempio:

```text
Ingress:
green-space/applications-ingress

Backend:
green-space/app-wear-service
```

Nel backend si specifica solo:

```yaml
service:
  name: app-wear-service
```

Non:

```yaml
service:
  name: green-space/app-wear-service
```

Questo è diverso dall’argomento del controller:

```text
--default-backend-service=green-space/default-backend-service
```

che usa esplicitamente il formato:

```text
namespace/service
```

---

## 17. Comandi di verifica

Visualizzare le annotation:

```bash
kubectl get ingress applications-ingress \
  -n green-space -o yaml
```

Estrarre solo le annotation:

```bash
kubectl get ingress applications-ingress \
  -n green-space \
  -o jsonpath='{.metadata.annotations}{"\n"}'
```

Descrivere l’Ingress:

```bash
kubectl describe ingress applications-ingress \
  -n green-space
```

Testare senza DNS, impostando l’header `Host`:

```bash
curl -H 'Host: apps.example.com' \
  http://<INGRESS-IP>/app1/products
```

Verificare redirect e header:

```bash
curl -I -H 'Host: apps.example.com' \
  http://<INGRESS-IP>/app1
```

Seguire un eventuale redirect:

```bash
curl -L -H 'Host: apps.example.com' \
  http://<INGRESS-IP>/app1
```

Controllare i log del controller:

```bash
kubectl logs -n ingress-nginx \
  -l app.kubernetes.io/component=controller
```

---

# Schema mentale finale

```text
Host + path
    ↓
Scelta della regola Ingress
    ↓
Scelta del Service backend
    ↓
Annotation ingress-nginx
    ├── rewrite del path
    ├── redirect HTTPS
    ├── regex
    └── altre opzioni NGINX
    ↓
Service
    ↓
EndpointSlice
    ↓
Pod
```

Regola da ricordare:

```text
path
└── decide quale backend usare

rewrite-target
└── decide quale URI riceve quel backend
```

Il rewrite non cambia il Service selezionato e non elimina i conflitti tra due Ingress che dichiarano lo stesso host e lo stesso path.
