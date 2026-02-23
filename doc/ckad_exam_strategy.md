# 📘 CKAD – Esame + Cheatsheet con shortcut

---

## 1️⃣ Info esame

Caratteristiche principali:

* ⏱️ Durata: 2 ore
* 💻 100% pratico (hands-on)
* 🌍 Online e proctored
* 📊 ~15–20 task
* ✅ Passing score: 66%
* 📅 Validità: 2 anni

L’esame si svolge su cluster Kubernetes reali.

---

## 2️⃣ Setup iniziale consigliato

Appena inizi l’esame:
### Linux:

```bash
mkdir ~/my-cka-exam
cd ~/my-cka-exam
```
- Autocompletion
```bash
alias k=kubectl
source <(kubectl completion bash)
```
- Alias/Funzione per velocità
```bash
alias k='kubectl'
alias kp='kubectl get pods -o wide'
alias kd='kubectl get deployment -o wide'
alias ks='kubectl get svc -o wide'
alias kc='kubectl config get-contexts'
alias kdry='--dry-run=client -o yaml'
alias kapply='kubectl apply -f'
alias kctx='kubectl config use-context'
```
### Windows:

```bash
mkdir $HOME\my-cka-exam
cd $HOME\my-cka-exam
```
- Autocompletion
```bash
kubectl completion powershell | Out-String | Invoke-Expression
```
- Alias/Funzione per velocità
```bash
New-Item -ItemType Directory -Path (Split-Path -Parent $PROFILE) -Force
Set-Alias k kubectl
Function kp { kubectl get pods -o wide }
Function kd { kubectl get deployment -o wide }
Function ks { kubectl get svc -o wide }
Function kc { kubectl config get-contexts }
Function kdry { param($cmd) kubectl $cmd --dry-run=client -o yaml }
Function kapply { param($file) kubectl apply -f $file }
notepad $PROFILE
. $PROFILE
```


---

- [GuideLines Before Exam.pdf](https://github.com/user-attachments/files/25387659/GuideLines%2Bto%2BScore%2B100.%2Bin%2BExam%2B.Before%2BExam.pdf)
- [GuideLines During Exam.pdf](https://github.com/user-attachments/files/25387654/GuideLines%2Bto%2BScore%2B100.%2Bin%2BExam%2B.During%2BExam.pdf)

---

# 📘 CKAD – Cheatsheet Rapido 

| Risorsa            | Comando rapido                                                                 | Uso / Note                                | Alias / Shortcut                        | Alias Dry-Run / YAML           |
|-------------------|-------------------------------------------------------------------------------|------------------------------------------|----------------------------------------|-------------------------------|
| **Cluster/Context**| `kubectl config get-contexts` <br> `kubectl config current-context` <br> `kubectl config use-context <ctx>` | Verificare e cambiare contesto / cluster | `kc` <br> `kctx <ctx>`                 | -                             |
| **Namespace**      | `kubectl get ns` <br> `kubectl config set-context --current --namespace=<ns>` <br> `kubectl get pods -n <ns>` | Impostare namespace corrente o usare `-n`| `ns` (short predefinito)              | -                             |
| **Pod**            | `kubectl run nginx --image=nginx` <br> `kubectl describe pod <pod>` <br> `kubectl logs <pod>` <br> `kubectl exec -it <pod> -- sh` | Creazione e debug rapido                 | `kp` <br> `po`                        | `kdry "create pod nginx --image=nginx"` |
| **Deployment**     | `kubectl create deployment web --image=nginx --replicas=3` <br> `kubectl edit deployment web` <br> `kubectl scale deployment web --replicas=5` <br> `kubectl set image deployment/web nginx=nginx:1.25` <br> `kubectl rollout status deployment/web` <br> `kubectl rollout undo deployment/web` | Gestione cicli di vita deployment        | `kd` <br> `deploy`                     | `kdry "create deployment web --image=nginx"` |
| **Service**        | `kubectl expose deployment web --port=80 --type=ClusterIP` <br> `kubectl expose deployment web --port=80 --type=NodePort` <br> `kubectl create service clusterip my-cs --tcp=5678:8080 --dry-run=client -o yaml` | Esposizione e test servizi               | `ks` <br> `svc`                        | `kdry "create service clusterip my-cs --tcp=5678:8080"` |
| **ConfigMap**      | `kubectl create configmap my-config --from-literal=key=value` <br> `kubectl create configmap my-config --from-env-file=file.env` | Configurazioni chiave/valore             | `cm`                                   | `kdry "create configmap my-config --from-literal=key=value"` |
| **Secret**         | `kubectl create secret generic my-secret --from-literal=key=value` <br> `kubectl create secret generic my-secret --from-env-file=file.env` | Gestione segreti                          | `secret`                               | `kdry "create secret generic my-secret --from-literal=key=value"` |
| **Job / CronJob**  | `kubectl create job my-job --image=busybox -- date` <br> `kubectl create cronjob my-job --image=busybox --schedule="*/1 * * * *"` | Job singolo o pianificato                 | `job` / `cj`                           | `kdry "create job my-job --image=busybox -- date"` |
| **ServiceAccount** | `kubectl create sa my-app-sa`                                                 | Account per risorse                       | `sa`                                   | `kdry "create sa my-app-sa"` |
| **Role / ClusterRole** | `kubectl create role pod-reader --verb=get,list,watch --resource=pods` <br> `kubectl create clusterrole pod-reader --verb=get,list,watch --resource=pods` | Gestione permessi                         | `role` / `cr`                          | `kdry "create role pod-reader --verb=get,list,watch --resource=pods"` |
| **RoleBinding / ClusterRoleBinding** | `kubectl create rolebinding admin-binding --clusterrole=admin --user=user1 --user=user2 --group=group1` <br> `kubectl create clusterrolebinding cluster-admin-binding --clusterrole=cluster-admin --user=user1` | Associa permessi utenti/gruppi           | `rb` / `crb`                           | `kdry "create rolebinding admin-binding --clusterrole=admin --user=user1"` |
| **Debug / Info**   | `kubectl get all` <br> `kubectl describe <resource>` <br> `kubectl logs <pod>` <br> `kubectl exec -it <pod> -- sh` <br> `kubectl top pod` <br> `kubectl top node` <br> `kubectl explain <resource>` | Stato cluster, risorse, metriche         | `k`                                    | -                             |
| **Dry-Run / YAML** | `kubectl create <resource> ... --dry-run=client -o yaml > file.yaml` <br> `kubectl apply -f file.yaml` | Genera e applica YAML rapidamente        | `k`                                    | `kdry "<comando>"` <br> `kapply <file>` |


---


> Se puoi farlo in un comando, fallo in un comando.  
> Se serve YAML, genera con `--dry-run=client -o yaml` e applica.

