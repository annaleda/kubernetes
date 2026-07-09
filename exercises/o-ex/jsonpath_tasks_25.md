-   [Home](../../readme.md) \| [Teoria](../../arguments.md) \| [Info
    Exam](../../doc/ckad_exam_strategy.md) \| [Home Other
    Exercises](../o_exercises.md)

------------------------------------------------------------------------

### CKAD JSONPath Tasks (25 esercizi)

------------------------------------------------------------------------

## JSONPATH-1 --- Pod IP

SECTION: APPLICATION OBSERVABILITY AND MAINTENANCE

-   Obiettivo
    -   Eseguire l'estrazione tramite JSONPath

------------------------------------------------------------------------

```{=html}
<details>
```
```{=html}
<summary>
```
Soluzione
```{=html}
</summary>
```
``` sh
kubectl get pod nginx -o jsonpath='{.status.podIP}' > /root/pod-ip.txt
```

```{=html}
</details>
```

------------------------------------------------------------------------

## JSONPATH-2 --- Node Name

SECTION: APPLICATION OBSERVABILITY AND MAINTENANCE

-   Obiettivo
    -   Eseguire l'estrazione tramite JSONPath

------------------------------------------------------------------------

```{=html}
<details>
```
```{=html}
<summary>
```
Soluzione
```{=html}
</summary>
```
``` sh
kubectl get pod nginx -o jsonpath='{.spec.nodeName}'
```

```{=html}
</details>
```

------------------------------------------------------------------------

## JSONPATH-3 --- Pod Images

SECTION: APPLICATION OBSERVABILITY AND MAINTENANCE

-   Obiettivo
    -   Eseguire l'estrazione tramite JSONPath

------------------------------------------------------------------------

```{=html}
<details>
```
```{=html}
<summary>
```
Soluzione
```{=html}
</summary>
```
``` sh
kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{" "}{.spec.containers[*].image}{"\n"}{end}'
```

```{=html}
</details>
```

------------------------------------------------------------------------

## JSONPATH-4 --- Node InternalIP

SECTION: APPLICATION OBSERVABILITY AND MAINTENANCE

-   Obiettivo
    -   Eseguire l'estrazione tramite JSONPath

------------------------------------------------------------------------

```{=html}
<details>
```
```{=html}
<summary>
```
Soluzione
```{=html}
</summary>
```
``` sh
kubectl get nodes -o jsonpath='{range .items[*]}{.metadata.name}{" "}{.status.addresses[?(@.type=="InternalIP")].address}{"\n"}{end}'
```

```{=html}
</details>
```

------------------------------------------------------------------------

## JSONPATH-5 --- Service Type

SECTION: APPLICATION OBSERVABILITY AND MAINTENANCE

-   Obiettivo
    -   Eseguire l'estrazione tramite JSONPath

------------------------------------------------------------------------

```{=html}
<details>
```
```{=html}
<summary>
```
Soluzione
```{=html}
</summary>
```
``` sh
kubectl get svc web -o jsonpath='{.spec.type}'
```

```{=html}
</details>
```

------------------------------------------------------------------------

## JSONPATH-6 --- NodePort

SECTION: APPLICATION OBSERVABILITY AND MAINTENANCE

-   Obiettivo
    -   Eseguire l'estrazione tramite JSONPath

------------------------------------------------------------------------

```{=html}
<details>
```
```{=html}
<summary>
```
Soluzione
```{=html}
</summary>
```
``` sh
kubectl get svc web -o jsonpath='{.spec.ports[*].nodePort}'
```

```{=html}
</details>
```

------------------------------------------------------------------------

## JSONPATH-7 --- ServiceAccount

SECTION: APPLICATION OBSERVABILITY AND MAINTENANCE

-   Obiettivo
    -   Eseguire l'estrazione tramite JSONPath

------------------------------------------------------------------------

```{=html}
<details>
```
```{=html}
<summary>
```
Soluzione
```{=html}
</summary>
```
``` sh
kubectl get pod nginx -o jsonpath='{.spec.serviceAccountName}'
```

```{=html}
</details>
```

------------------------------------------------------------------------

## JSONPATH-8 --- Namespace

SECTION: APPLICATION OBSERVABILITY AND MAINTENANCE

-   Obiettivo
    -   Eseguire l'estrazione tramite JSONPath

------------------------------------------------------------------------

```{=html}
<details>
```
```{=html}
<summary>
```
Soluzione
```{=html}
</summary>
```
``` sh
kubectl get pod nginx -o jsonpath='{.metadata.namespace}'
```

```{=html}
</details>
```

------------------------------------------------------------------------

## JSONPATH-9 --- Labels

SECTION: APPLICATION OBSERVABILITY AND MAINTENANCE

-   Obiettivo
    -   Eseguire l'estrazione tramite JSONPath

------------------------------------------------------------------------

```{=html}
<details>
```
```{=html}
<summary>
```
Soluzione
```{=html}
</summary>
```
``` sh
kubectl get pod nginx -o jsonpath='{.metadata.labels}'
```

```{=html}
</details>
```

------------------------------------------------------------------------

## JSONPATH-10 --- Annotations

SECTION: APPLICATION OBSERVABILITY AND MAINTENANCE

-   Obiettivo
    -   Eseguire l'estrazione tramite JSONPath

------------------------------------------------------------------------

```{=html}
<details>
```
```{=html}
<summary>
```
Soluzione
```{=html}
</summary>
```
``` sh
kubectl get pod nginx -o jsonpath='{.metadata.annotations}'
```

```{=html}
</details>
```

------------------------------------------------------------------------

## JSONPATH-11 --- Creation Timestamp

SECTION: APPLICATION OBSERVABILITY AND MAINTENANCE

-   Obiettivo
    -   Eseguire l'estrazione tramite JSONPath

------------------------------------------------------------------------

```{=html}
<details>
```
```{=html}
<summary>
```
Soluzione
```{=html}
</summary>
```
``` sh
kubectl get pod nginx -o jsonpath='{.metadata.creationTimestamp}'
```

```{=html}
</details>
```

------------------------------------------------------------------------

## JSONPATH-12 --- Host IP

SECTION: APPLICATION OBSERVABILITY AND MAINTENANCE

-   Obiettivo
    -   Eseguire l'estrazione tramite JSONPath

------------------------------------------------------------------------

```{=html}
<details>
```
```{=html}
<summary>
```
Soluzione
```{=html}
</summary>
```
``` sh
kubectl get pod nginx -o jsonpath='{.status.hostIP}'
```

```{=html}
</details>
```

------------------------------------------------------------------------

## JSONPATH-13 --- Restart Count

SECTION: APPLICATION OBSERVABILITY AND MAINTENANCE

-   Obiettivo
    -   Eseguire l'estrazione tramite JSONPath

------------------------------------------------------------------------

```{=html}
<details>
```
```{=html}
<summary>
```
Soluzione
```{=html}
</summary>
```
``` sh
kubectl get pod nginx -o jsonpath='{.status.containerStatuses[*].restartCount}'
```

```{=html}
</details>
```

------------------------------------------------------------------------

## JSONPATH-14 --- Ready Status

SECTION: APPLICATION OBSERVABILITY AND MAINTENANCE

-   Obiettivo
    -   Eseguire l'estrazione tramite JSONPath

------------------------------------------------------------------------

```{=html}
<details>
```
```{=html}
<summary>
```
Soluzione
```{=html}
</summary>
```
``` sh
kubectl get pod nginx -o jsonpath='{.status.containerStatuses[*].ready}'
```

```{=html}
</details>
```

------------------------------------------------------------------------

## JSONPATH-15 --- CPU Requests

SECTION: APPLICATION OBSERVABILITY AND MAINTENANCE

-   Obiettivo
    -   Eseguire l'estrazione tramite JSONPath

------------------------------------------------------------------------

```{=html}
<details>
```
```{=html}
<summary>
```
Soluzione
```{=html}
</summary>
```
``` sh
kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{" "}{.spec.containers[*].resources.requests.cpu}{"\n"}{end}'
```

```{=html}
</details>
```

------------------------------------------------------------------------

## JSONPATH-16 --- Memory Requests

SECTION: APPLICATION OBSERVABILITY AND MAINTENANCE

-   Obiettivo
    -   Eseguire l'estrazione tramite JSONPath

------------------------------------------------------------------------

```{=html}
<details>
```
```{=html}
<summary>
```
Soluzione
```{=html}
</summary>
```
``` sh
kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{" "}{.spec.containers[*].resources.requests.memory}{"\n"}{end}'
```

```{=html}
</details>
```

------------------------------------------------------------------------

## JSONPATH-17 --- CPU Limits

SECTION: APPLICATION OBSERVABILITY AND MAINTENANCE

-   Obiettivo
    -   Eseguire l'estrazione tramite JSONPath

------------------------------------------------------------------------

```{=html}
<details>
```
```{=html}
<summary>
```
Soluzione
```{=html}
</summary>
```
``` sh
kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{" "}{.spec.containers[*].resources.limits.cpu}{"\n"}{end}'
```

```{=html}
</details>
```

------------------------------------------------------------------------

## JSONPATH-18 --- Memory Limits

SECTION: APPLICATION OBSERVABILITY AND MAINTENANCE

-   Obiettivo
    -   Eseguire l'estrazione tramite JSONPath

------------------------------------------------------------------------

```{=html}
<details>
```
```{=html}
<summary>
```
Soluzione
```{=html}
</summary>
```
``` sh
kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{" "}{.spec.containers[*].resources.limits.memory}{"\n"}{end}'
```

```{=html}
</details>
```

------------------------------------------------------------------------

## JSONPATH-19 --- Deployment Images

SECTION: APPLICATION OBSERVABILITY AND MAINTENANCE

-   Obiettivo
    -   Eseguire l'estrazione tramite JSONPath

------------------------------------------------------------------------

```{=html}
<details>
```
```{=html}
<summary>
```
Soluzione
```{=html}
</summary>
```
``` sh
kubectl get deploy web -o jsonpath='{.spec.template.spec.containers[*].image}'
```

```{=html}
</details>
```

------------------------------------------------------------------------

## JSONPATH-20 --- PVC Names

SECTION: APPLICATION OBSERVABILITY AND MAINTENANCE

-   Obiettivo
    -   Eseguire l'estrazione tramite JSONPath

------------------------------------------------------------------------

```{=html}
<details>
```
```{=html}
<summary>
```
Soluzione
```{=html}
</summary>
```
``` sh
kubectl get pod nginx -o jsonpath='{.spec.volumes[*].persistentVolumeClaim.claimName}'
```

```{=html}
</details>
```

------------------------------------------------------------------------

## JSONPATH-21 --- ConfigMaps

SECTION: APPLICATION OBSERVABILITY AND MAINTENANCE

-   Obiettivo
    -   Eseguire l'estrazione tramite JSONPath

------------------------------------------------------------------------

```{=html}
<details>
```
```{=html}
<summary>
```
Soluzione
```{=html}
</summary>
```
``` sh
kubectl get pod nginx -o jsonpath='{.spec.volumes[*].configMap.name}'
```

```{=html}
</details>
```

------------------------------------------------------------------------

## JSONPATH-22 --- Secrets

SECTION: APPLICATION OBSERVABILITY AND MAINTENANCE

-   Obiettivo
    -   Eseguire l'estrazione tramite JSONPath

------------------------------------------------------------------------

```{=html}
<details>
```
```{=html}
<summary>
```
Soluzione
```{=html}
</summary>
```
``` sh
kubectl get pod nginx -o jsonpath='{.spec.volumes[*].secret.secretName}'
```

```{=html}
</details>
```

------------------------------------------------------------------------

## JSONPATH-23 --- Init Containers

SECTION: APPLICATION OBSERVABILITY AND MAINTENANCE

-   Obiettivo
    -   Eseguire l'estrazione tramite JSONPath

------------------------------------------------------------------------

```{=html}
<details>
```
```{=html}
<summary>
```
Soluzione
```{=html}
</summary>
```
``` sh
kubectl get pod nginx -o jsonpath='{.spec.initContainers[*].name}'
```

```{=html}
</details>
```

------------------------------------------------------------------------

## JSONPATH-24 --- Selectors

SECTION: APPLICATION OBSERVABILITY AND MAINTENANCE

-   Obiettivo
    -   Eseguire l'estrazione tramite JSONPath

------------------------------------------------------------------------

```{=html}
<details>
```
```{=html}
<summary>
```
Soluzione
```{=html}
</summary>
```
``` sh
kubectl get svc web -o jsonpath='{.spec.selector}'
```

```{=html}
</details>
```

------------------------------------------------------------------------

## JSONPATH-25 --- Container Ports

SECTION: APPLICATION OBSERVABILITY AND MAINTENANCE

-   Obiettivo
    -   Eseguire l'estrazione tramite JSONPath

------------------------------------------------------------------------

```{=html}
<details>
```
```{=html}
<summary>
```
Soluzione
```{=html}
</summary>
```
``` sh
kubectl get pod nginx -o jsonpath='{.spec.containers[*].ports[*].containerPort}'
```

```{=html}
</details>
```

------------------------------------------------------------------------
