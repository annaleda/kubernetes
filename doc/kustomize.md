## 🧩 Kustomize

Kustomize è uno strumento integrato in `kubectl` che permette di personalizzare i manifest Kubernetes **senza usare template**.

A differenza di Helm:
- Non usa variabili
- Non usa motore di templating
- Lavora tramite patch e overlay

È incluso in kubectl (non serve installarlo separatamente nelle versioni moderne).

---

### Verifica Kustomize

Controlla che sia disponibile:

```bash
kubectl version --client
