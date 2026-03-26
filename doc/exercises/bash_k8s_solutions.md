# Bash & Kubernetes -- Soluzioni



### 1. Loop infinito

``` bash
while true; do
  echo "hello world"
  sleep 3
done
```

### 2. Contatore

``` bash
i=1
while [ $i -le 10 ]; do
  echo $i
  i=$((i+1))
done
```


### 3. Curl semplice

``` bash
while true; do
  curl -s localhost:80
  sleep 5
done
```

### 4. Timestamp

``` bash
while true; do
  echo "[$(date)] $(curl -s localhost:80)"
  sleep 5
done
```

### 5. Controllo errore

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


### 6. Log su file

``` bash
while true; do
  echo "[$(date)] $(curl -s localhost:80)" >> log.txt
  sleep 5
done
```

### 7. Retry limitato

``` bash
count=0
while [ $count -lt 5 ]; do
  curl -s localhost:80
  count=$((count+1))
  sleep 2
done

echo "Fine tentativi"
```


### 8. Health check

``` bash
while true; do
  STATUS=$(curl -s -o /dev/null -w "%{http_code}" localhost:80)

  if [ "$STATUS" -eq 200 ]; then
    echo "OK"
  else
    echo "ERROR $STATUS"
  fi

  sleep 5
done
```

### 9. Critical check

``` bash
fail=0

while true; do
  if curl -s localhost:80 > /dev/null; then
    echo "OK"
    fail=0
  else
    fail=$((fail+1))
    echo "FAIL $fail"
  fi

  if [ $fail -ge 3 ]; then
    echo "CRITICAL"
  fi

  sleep 5
done
```

### 10. Sidecar simulation

``` bash
while true; do
  echo "[$(date)] checking service..."
  STATUS=$(curl -s -o /dev/null -w "%{http_code}" localhost:80)
  echo "status: $STATUS" >> sidecar.log
  sleep 5
done
```
