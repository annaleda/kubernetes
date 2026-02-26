
## 📦 Helm

Helm è il package manager di Kubernetes.
Permette di installare applicazioni nel Cluster/nei Cluster tramite Chart.

### Installazione Helm su Linux (Debian/Ubuntu)

Metodo ufficiale (APT repository)

```bash
curl https://baltocdn.com/helm/signing.asc | sudo apt-key add -
sudo apt-get install apt-transport-https --yes
echo "deb https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```
Verifica installazione

```bash
helm version
```
### Installazione Helm su Windows (PowerShell)

Metodo ufficiale manuale
### Scaricare Helm

Aprire PowerShell e scaricare l’ultima versione:

```bash
Invoke-WebRequest -Uri https://get.helm.sh/helm-v3.14.0-windows-amd64.zip -OutFile helm.zip
```

Estrarre il file ZIP

```bash
Expand-Archive helm.zip -DestinationPath C:\helm
```

Aggiungere Helm al PATH

```bash
$env:Path += ";C:\helm\windows-amd64"
```
Per renderlo permanente:

Apri Variabili d’ambiente

Modifica la variabile PATH

Aggiungi: C:\helm\windows-amd64

Verifica installazione

```bash
helm version
```
Cercare applicazioni

```bash
helm search repo <nome-applicazione>
```
Esempio:

```bash
helm search repo nginx
```

Installare un'applicazione

```bash
helm install <nome-release> <chart>
```
Esempio:
```bash
helm install my-nginx bitnami/nginx
```
Importante
> Ogni installazione di un chart nel cluster è chiamata RELEASE

### Release

È possibile avere:

Più release dello stesso chart

Configurazioni diverse per ogni release

 Installare con valori personalizzati
 ```bash
helm install my-nginx bitnami/nginx --set replicaCount=3
```
Oppure usando un file:
```bash
helm install my-nginx bitnami/nginx -f values.yaml
```
Creare un Chart personalizzato

```bash
helm create <nome-chart>
```
Esempio:

```bash
helm create my-app
```
Verrà creata una directory my-app/.

📂 Struttura di un Chart
```bash
my-app/
├── Chart.yaml
├── values.yaml
├── templates/
└── charts/
📄 Chart.yaml
```
Contiene:

  - Nome chart
  - Versione
  - Descrizione

📄 values.yaml

Contiene i valori di default del chart.
Può essere sovrascritto con --set o -f.

📂 templates/

Contiene i manifest Kubernetes in formato template.

📂 charts/

Directory opzionale che può contenere sub-chart.

Comandi principali per lavorare con Chart

Validare un Chart
> helm lint <percorso-chart>

Esempio:

```bash
helm lint my-app
```

Generare i manifest senza installare

```bash
helm template <percorso-chart>
```
Esempio:

```bash
helm template my-app
```

Molto utile per verificare l’output YAML.

Installare un Chart locale
```bash
helm install <nome-release> <percorso-chart>
```
Esempio:
```bash
helm install my-release my-app
```
Vedere le release installate
```bash
helm list
```
Oppure:

```bash
helm ls --all
```
Aggiornare una release

```bash
helm upgrade <nome-release> <percorso-chart>
```
Esempio:

```bash
helm upgrade my-release my-app
```
Fare rollback

```bash
helm rollback <nome-release> <numero-versione>
```
Esempio:

```bash
helm rollback my-release 1
```
Eliminare una release

```bash
helm uninstall <nome-release>
```
 Nota:
> Il comando helm delete --purge era usato in Helm v2.

> In Helm v3 si usa helm uninstall.

Comandi utili per debug
```bash
helm status <nome-release>

helm get values <nome-release>

helm get manifest <nome-release>

helm history <nome-release>
```
---
