# Bash & Kubernetes -- Esempi pratici

## Loop base

``` bash
while true; do echo "ciao"; sleep 2; done
```

## Contatore

``` bash
i=1
while [ $i -le 5 ]; do
  echo "numero: $i"
  i=$((i+1))
done
```

## Curl semplice

``` bash
while true; do
  curl -s localhost:80
  sleep 5
done
```

## Curl con timestamp

``` bash
while true; do
  echo "[$(date)]"
  curl -s localhost:80
  sleep 5
done
```

## Gestione errori

``` bash
while true; do
  if curl -s localhost:80 > /dev/null; then
    echo "OK"
  else
    echo "ERROR"
  fi
  sleep 5
done
```

## Log su file

``` bash
while true; do
  echo "[$(date)] $(curl -s localhost:80)" >> log.txt
  sleep 5
done
```

## Retry limitato

``` bash
count=0
while [ $count -lt 5 ]; do
  curl -s localhost:80
  count=$((count+1))
  sleep 2
done
```

## Health check avanzato

``` bash
while true; do
  STATUS=$(curl -s -o /dev/null -w "%{http_code}" localhost:80)

  if [ "$STATUS" -eq 200 ]; then
    echo "OK"
  else
    echo "ERROR: $STATUS"
  fi

  sleep 5
done
```
