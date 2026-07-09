- [ Home ](../../readme.md)   | [ Teoria ](../../arguments.md)   | [ Info Exam ](../../doc/ckad_exam_strategy.md)    | [ Home Other Exercises ](../o_exercises.md) |  [ Exercises ](./oci.md)
--- 

## Docker Cheat Sheet - CKAD

> Cheatsheet pratica per l'esame CKAD, con particolare attenzione ai
> comandi Docker più richiesti.

## Informazioni

``` bash
docker version
docker info
docker system info
```

## Images

``` bash
docker search nginx
docker pull nginx
docker pull nginx:1.27
docker images
docker image ls
docker inspect nginx
docker rmi nginx
docker image rm nginx
docker image prune
docker image prune -a
```

## Build

``` bash
docker build -t myapp .
docker build -t myapp:v1 .
docker build -f Dockerfile.prod -t myapp .
```

## Tag

``` bash
docker tag myapp myrepo/myapp:v1
docker tag nginx nginx:test
```

## Push / Pull

``` bash
docker login
docker push myrepo/myapp:v1
docker logout
```

## Run

``` bash
docker run nginx
docker run -it ubuntu bash
docker run -d nginx
docker run --name web nginx
docker run -p 8080:80 nginx
docker run -e ENV=prod nginx
docker run -v /host:/container nginx
docker run --restart always nginx
```

## Container

``` bash
docker ps
docker ps -a
docker start container
docker stop container
docker restart container
docker kill container
docker rm container
docker rm -f container
```

## Logs

``` bash
docker logs container
docker logs -f container
docker logs --tail 100 container
```

## Exec

``` bash
docker exec -it container bash
docker exec -it container sh
docker exec container ls /
```

## Inspect

``` bash
docker inspect container
docker inspect image
docker inspect -f '{{ .NetworkSettings.IPAddress }}' container
```

## Copy

``` bash
docker cp file.txt container:/tmp
docker cp container:/tmp/file .
```

## Stats / Top / Events

``` bash
docker stats
docker stats container
docker top container
docker events
```

## Networks

``` bash
docker network ls
docker network create mynet
docker network inspect mynet
docker network connect mynet container
docker network disconnect mynet container
docker network rm mynet
```

## Volumes

``` bash
docker volume ls
docker volume create data
docker volume inspect data
docker volume rm data
docker volume prune
```

## Save / Load 

``` bash
docker save -o nginx.tar nginx
docker load -i nginx.tar
```

## Export / Import 

``` bash
docker export container > container.tar
docker import container.tar myimage
```

**Differenza**

  save                export
  ------------------- ---------------------
  salva un'immagine   salva un container
  mantiene i layer    perde i layer
  si usa con `load`   si usa con `import`

## History / Diff / Commit

``` bash
docker history nginx
docker diff container
docker commit container myimage:v1
```

## Pulizia

``` bash
docker system prune
docker system prune -a
```

## Dockerfile

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

## Docker Compose

``` bash
docker compose up
docker compose up -d
docker compose down
docker compose ps
docker compose logs
docker compose restart
```

# I 20 comandi da ricordare

``` bash
docker build -t app:v1 .
docker images
docker ps
docker ps -a
docker run
docker run -d
docker exec -it container sh
docker logs container
docker stop container
docker start container
docker restart container
docker rm container
docker rmi image
docker tag app repo/app:v1
docker pull nginx
docker push repo/app:v1
docker save -o app.tar app:v1
docker load -i app.tar
docker export container > container.tar
docker import container.tar newimage
```

## Esempio CKAD

``` bash
docker build -t myapp:1.2 .
docker save -o /tmp/myapp.tar myapp:1.2
```

Oppure con Podman:

``` bash
podman build -t myapp:1.2 .
podman save -o /tmp/myapp.tar myapp:1.2
```
