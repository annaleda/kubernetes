-   [Home](../../readme.md) \| [Teoria](../../arguments.md) \| [Info
    Exam](../../doc/ckad_exam_strategy.md) \| [Home Other
    Exercises](../o_exercises.md) \| [Exercises](./oci.md)

------------------------------------------------------------------------

# Podman Cheat Sheet - CKAD

> Cheatsheet pratica per l'esame CKAD con i comandi Podman più
> utilizzati.

## Informazioni

``` bash
podman version
podman info
podman system info
```

## Images

``` bash
podman search nginx
podman pull nginx
podman pull nginx:1.27
podman images
podman image ls
podman inspect nginx
podman rmi nginx
podman image rm nginx
podman image prune
podman image prune -a
```

## Build

``` bash
podman build -t myapp .
podman build -t myapp:v1 .
podman build -f Containerfile -t myapp .
```

> Podman usa **Containerfile** come nome predefinito ma supporta anche
> **Dockerfile**.

## Tag

``` bash
podman tag myapp myrepo/myapp:v1
podman tag nginx nginx:test
```

## Push / Pull

``` bash
podman login
podman push myrepo/myapp:v1
podman logout
```

## Run

``` bash
podman run nginx
podman run -it ubuntu bash
podman run -d nginx
podman run --name web nginx
podman run -p 8080:80 nginx
podman run -e ENV=prod nginx
podman run -v /host:/container nginx
podman run --restart always nginx
```

## Container

``` bash
podman ps
podman ps -a
podman start container
podman stop container
podman restart container
podman kill container
podman rm container
podman rm -f container
```

## Logs

``` bash
podman logs container
podman logs -f container
podman logs --tail 100 container
```

## Exec

``` bash
podman exec -it container bash
podman exec -it container sh
podman exec container ls /
```

## Inspect

``` bash
podman inspect container
podman inspect image
podman inspect -f '{{ .NetworkSettings.IPAddress }}' container
```

## Copy

``` bash
podman cp file.txt container:/tmp
podman cp container:/tmp/file .
```

## Stats / Top

``` bash
podman stats
podman top container
```

## Networks

``` bash
podman network ls
podman network create mynet
podman network inspect mynet
podman network connect mynet container
podman network disconnect mynet container
podman network rm mynet
```

## Volumes

``` bash
podman volume ls
podman volume create data
podman volume inspect data
podman volume rm data
podman volume prune
```

## Save / Load

``` bash
podman save -o nginx.tar nginx
podman load -i nginx.tar
```

## Export / Import

``` bash
podman export container > container.tar
podman import container.tar myimage
```

## History / Diff / Commit

``` bash
podman history nginx
podman diff container
podman commit container myimage:v1
```

## Pulizia

``` bash
podman system prune
podman system prune -a
```

## Containerfile

``` dockerfile
COPY app.py /app
ADD file.tar.gz /app
WORKDIR /app
ENV APP=prod
CMD ["python","app.py"]
ENTRYPOINT ["python"]
EXPOSE 8080
USER app
VOLUME /data
```

## Podman Pods

``` bash
podman pod create --name webpod
podman pod ps
podman pod inspect webpod
podman pod start webpod
podman pod stop webpod
podman pod rm webpod
```

## Kubernetes

``` bash
podman generate kube web > pod.yaml
podman play kube pod.yaml
```

## Systemd

``` bash
podman generate systemd web
```

## Rootless

``` bash
podman unshare
podman info
```

# I 20 comandi da ricordare

``` bash
podman build -t app:v1 .
podman images
podman ps
podman ps -a
podman run
podman run -d
podman exec -it container sh
podman logs container
podman stop container
podman start container
podman restart container
podman rm container
podman rmi image
podman tag app repo/app:v1
podman pull nginx
podman push repo/app:v1
podman save -o app.tar app:v1
podman load -i app.tar
podman export container > container.tar
podman import container.tar newimage
```

## Differenze Docker vs Podman

  Docker               Podman
  -------------------- ----------------------------
  Daemon (`dockerd`)   Daemonless
  Root di default      Rootless di default
  Dockerfile           Dockerfile o Containerfile
  docker compose       podman compose
  docker run           podman run

## Esempio CKAD

``` bash
podman build -t myapp:1.2 .
podman save -o /tmp/myapp.tar myapp:1.2
```
