# NGINX Ingress Controller: Deployment e IngressClass

## 1. Differenza tra Ingress e Ingress Controller

Un oggetto `Ingress` contiene solo le regole di routing HTTP/HTTPS.

Esempio:

```yaml
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
```

Da solo, però, l’oggetto Ingress non instrada alcun traffico.

Serve un **Ingress Controller**, ad esempio NGINX, che:

* osserva gli oggetti Ingress;
* legge le relative regole;
* genera la configurazione NGINX;
* riceve il traffico HTTP/HTTPS;
* inoltra le richieste ai Service Kubernetes.

Il flusso è:

```text
Client
  ↓
Service del controller NGINX
  ↓
Pod ingress-nginx-controller
  ↓
Regole dell’Ingress
  ↓
Service applicativo
  ↓
EndpointSlice / Endpoint
  ↓
Pod applicativo
```

---

## 2. IngressClass

L’oggetto `IngressClass` identifica quale controller deve gestire un determinato Ingress.

Esempio:

```yaml
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  name: nginx
spec:
  controller: k8s.io/ingress-nginx
```

L’Ingress richiama questa classe tramite:

```yaml
spec:
  ingressClassName: nginx
```

Quindi:

```text
Ingress ingressClassName: nginx
           ↓
IngressClass metadata.name: nginx
           ↓
IngressClass spec.controller: k8s.io/ingress-nginx
           ↓
Controller avviato con --controller-class=k8s.io/ingress-nginx
```

Questi valori devono essere coerenti.

Comandi utili:

```bash
kubectl get ingressclass
kubectl describe ingressclass nginx
kubectl get ingressclass nginx -o yaml
```

Controllo rapido:

```bash
kubectl get ingressclass nginx \
  -o jsonpath='{.spec.controller}{"\n"}'
```

---

## 3. Deployment del controller NGINX

Il controller viene normalmente eseguito tramite un Deployment:

```text
ingress-nginx-controller
```

Per verificarlo:

```bash
kubectl get deployment -n ingress-nginx
```

Esempio:

```text
NAME                       READY   UP-TO-DATE   AVAILABLE
ingress-nginx-controller   1/1     1            1
```

Il valore atteso è:

```text
READY: 1/1
AVAILABLE: 1
```

Se invece compare:

```text
0/1
```

il controller non è disponibile e gli Ingress non possono funzionare correttamente.

Comandi di diagnostica:

```bash
kubectl describe deployment ingress-nginx-controller \
  -n ingress-nginx
```

```bash
kubectl get deployment ingress-nginx-controller \
  -n ingress-nginx -o yaml
```

---

## 4. Pod del controller

Il Deployment crea uno o più Pod NGINX.

Per visualizzarli:

```bash
kubectl get pods -n ingress-nginx
```

Per selezionare solo il controller:

```bash
kubectl get pods -n ingress-nginx \
  -l app.kubernetes.io/component=controller
```

Per descrivere il Pod:

```bash
kubectl describe pod -n ingress-nginx \
  $(kubectl get pods -n ingress-nginx \
    -l app.kubernetes.io/component=controller \
    -o jsonpath='{.items[0].metadata.name}')
```

Attenzione alla differenza tra:

```bash
-o name
```

e:

```bash
-o jsonpath='{.items[0].metadata.name}'
```

`-o name` restituisce:

```text
pod/ingress-nginx-controller-xxxxx
```

Il `jsonpath` restituisce solo:

```text
ingress-nginx-controller-xxxxx
```

Per questo, con `-o name`, non bisogna ripetere il tipo di risorsa:

```bash
kubectl describe \
  $(kubectl get pod -n ingress-nginx -o name | head -1) \
  -n ingress-nginx
```

---

## 5. Argomenti principali del controller

Nel Pod del controller si trovano gli argomenti di avvio.

Esempio:

```text
/nginx-ingress-controller
--publish-service=$(POD_NAMESPACE)/ingress-nginx-controller
--watch-ingress-without-class=true
--default-backend-service=default/default-backend-service
--controller-class=k8s.io/ingress-nginx
--ingress-class=nginx
--configmap=$(POD_NAMESPACE)/ingress-nginx-controller
```

### `--controller-class`

Esempio:

```text
--controller-class=k8s.io/ingress-nginx
```

Deve corrispondere a:

```yaml
spec:
  controller: k8s.io/ingress-nginx
```

dell’oggetto `IngressClass`.

---

### `--ingress-class`

Esempio:

```text
--ingress-class=nginx
```

Indica il nome della classe storicamente utilizzata dal controller.

Negli oggetti Ingress moderni si usa:

```yaml
spec:
  ingressClassName: nginx
```

---

### `--watch-ingress-without-class`

Esempio:

```text
--watch-ingress-without-class=true
```

Permette al controller di osservare anche gli Ingress che non hanno una classe esplicitamente configurata.

Per il CKAD è comunque preferibile specificare:

```yaml
spec:
  ingressClassName: nginx
```

---

### `--publish-service`

Esempio:

```text
--publish-service=$(POD_NAMESPACE)/ingress-nginx-controller
```

Indica quale Service rappresenta esternamente il controller.

Il controller può usare questo Service per aggiornare il campo `status` degli oggetti Ingress.

---

### `--configmap`

Esempio:

```text
--configmap=$(POD_NAMESPACE)/ingress-nginx-controller
```

Indica la ConfigMap utilizzata per configurare il comportamento globale di NGINX.

Per controllarla:

```bash
kubectl get configmap ingress-nginx-controller \
  -n ingress-nginx -o yaml
```

---

### `--default-backend-service`

Esempio:

```text
--default-backend-service=default/default-backend-service
```

Il formato è:

```text
namespace/nome-service
```

Questo backend viene usato dal controller per richieste che non corrispondono alle regole configurate, a seconda della versione e della configurazione del controller.

Il Service indicato deve esistere realmente.

Verifica:

```bash
kubectl get service default-backend-service -n default
```

Se il Service esiste invece in un altro namespace:

```text
green-space/default-backend-service
```

ma il controller cerca:

```text
default/default-backend-service
```

la configurazione è incoerente.

In alcuni scenari il controller può terminare con errore e andare in:

```text
CrashLoopBackOff
```

La correzione consiste nel modificare il riferimento in modo che punti al namespace corretto.

---

## 6. Default backend del controller e defaultBackend dell’Ingress

Sono due concetti distinti.

### Default backend del controller

Configurato tramite argomento:

```text
--default-backend-service=namespace/service
```

È una configurazione del Deployment del controller NGINX.

---

### Default backend dell’oggetto Ingress

Configurato nel manifest:

```yaml
spec:
  defaultBackend:
    service:
      name: default-backend-service
      port:
        number: 80
```

Questo backend appartiene a uno specifico oggetto Ingress.

Gestisce le richieste che non corrispondono ai path definiti in quell’Ingress.

---

## 7. Service del controller

Il Pod NGINX deve essere esposto tramite un Service.

Verifica:

```bash
kubectl get service -n ingress-nginx
```

Il Service può essere:

* `LoadBalancer`;
* `NodePort`;
* in alcuni laboratori, un tipo configurato specificamente dall’ambiente.

Esempio:

```text
NAME                       TYPE       PORT(S)
ingress-nginx-controller   NodePort   80:30080/TCP,443:30443/TCP
```

Il traffico entra dal Service del controller, non direttamente dal Service applicativo.

Esempio:

```text
curl http://<NODE-IP>:30080/app1
```

NGINX legge `/app1` e inoltra la richiesta al backend configurato nell’Ingress.

---

## 8. Webhook di admission

L’installazione di ingress-nginx può creare alcuni Job, ad esempio:

```text
ingress-nginx-admission-create
ingress-nginx-admission-patch
```

Questi Job configurano certificati e webhook di validazione.

Un Pod con:

```text
Status: Succeeded
Reason: Completed
Exit Code: 0
```

non è un errore.

È normale che un Pod appartenente a un Job completato mostri:

```text
Ready: False
```

Il Pod ha terminato il proprio compito e non deve rimanere attivo.

Non va confuso con il Pod:

```text
ingress-nginx-controller-xxxxx
```

che invece deve essere `Running` e `Ready`.

---

## 9. Secret del validating webhook

Il controller può montare un Secret simile a:

```text
ingress-nginx-admission
```

Esempio nel Pod:

```text
Volumes:
  webhook-cert:
    SecretName: ingress-nginx-admission
```

Se il Secret non esiste, negli eventi può comparire:

```text
FailedMount
secret "ingress-nginx-admission" not found
```

Verifica:

```bash
kubectl get secret ingress-nginx-admission \
  -n ingress-nginx
```

Se successivamente il Secret viene creato e il container parte, il vecchio evento `FailedMount` può rimanere visibile nel `describe`.

Bisogna quindi controllare gli eventi più recenti e lo stato attuale del container.

---

## 10. CrashLoopBackOff del controller

Se il Deployment mostra:

```text
0/1
```

e il Pod mostra:

```text
State: Waiting
Reason: CrashLoopBackOff
```

significa che il container:

1. viene avviato;
2. termina con errore;
3. Kubernetes tenta di riavviarlo;
4. aumenta progressivamente il tempo tra i tentativi.

Controllare:

```bash
kubectl describe pod <nome-pod> -n ingress-nginx
```

In particolare:

```text
State
Last State
Reason
Exit Code
Restart Count
Events
Args
```

Per leggere i log del container attuale:

```bash
kubectl logs <nome-pod> -n ingress-nginx
```

Per leggere i log dell’ultima istanza terminata:

```bash
kubectl logs <nome-pod> \
  -n ingress-nginx \
  --previous
```

`--previous` è particolarmente utile nei casi di `CrashLoopBackOff`.

---

## 11. Checklist di troubleshooting

Quando un Ingress non funziona:

```text
1. Pod applicativi
2. Service applicativi
3. EndpointSlice / Endpoint
4. Ingress
5. IngressClass
6. Deployment del controller
7. Pod del controller
8. Log del controller
9. Service del controller
```

Comandi:

```bash
kubectl get pods -n <namespace>
kubectl get services -n <namespace>
kubectl get endpointslices -n <namespace>
kubectl get ingress -n <namespace>
kubectl describe ingress <nome> -n <namespace>
kubectl get ingressclass
kubectl get deployment -n ingress-nginx
kubectl get pods -n ingress-nginx
kubectl describe pod <controller-pod> -n ingress-nginx
kubectl logs <controller-pod> -n ingress-nginx --previous
kubectl get services -n ingress-nginx
```

---

## 12. Caso pratico

Situazione:

```text
Ingress corretto
Service applicativi corretti
Endpoint presenti
Controller 0/1
```

Nel `describe` del controller compare:

```text
--default-backend-service=default/default-backend-service
```

Ma il Service esiste in:

```text
green-space/default-backend-service
```

Diagnosi:

```text
Il controller cerca il backend nel namespace sbagliato.
```

Risultato:

```text
Controller in CrashLoopBackOff
Ingress non operativo
```

Flusso di troubleshooting:

```text
Ingress non raggiungibile
        ↓
Verifica delle regole Ingress
        ↓
Verifica Service ed Endpoint
        ↓
Verifica Deployment controller
        ↓
Deployment 0/1
        ↓
Describe del Pod controller
        ↓
CrashLoopBackOff
        ↓
Controllo Args e log
        ↓
Riferimento errato al default backend
```

---

## Regola da ricordare

Un Ingress può essere perfettamente corretto, ma non funzionare perché il relativo controller è assente, non Ready o configurato male.

```text
Ingress resource ≠ Ingress Controller
```

L’Ingress descrive le regole.

Il controller le rende operative.
