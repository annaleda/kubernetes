# Bash & Kubernetes -- Esercizi

## 🟢 Facile

### 1. Loop infinito

Scrivi uno script che stampa "hello world" ogni 3 secondi.

### 2. Contatore

Stampa i numeri da 1 a 10 usando un loop.

## 🟡 Medio

### 3. Curl semplice

Fai una chiamata a `localhost:80` ogni 5 secondi.

### 4. Timestamp

Modifica l'esercizio sopra aggiungendo la data prima di ogni output.

### 5. Controllo errore

Scrivi uno script che: - chiama un endpoint - stampa "OK" se funziona -
stampa "ERROR" se fallisce

## 🟠 Difficile

### 6. Log su file

Salva l'output di curl in un file `log.txt` con timestamp.

### 7. Retry limitato

Fai massimo 5 tentativi di chiamata HTTP, poi esci.

## 🔴 Avanzato (CKAD style)

### 8. Health check

Scrivi uno script che: - controlla lo status HTTP - stampa "OK" se 200 -
stampa "ERROR `<codice>`{=html}" altrimenti

### 9. Critical check

Scrivi uno script che: - prova un endpoint - se fallisce 3 volte di fila
→ stampa "CRITICAL" - se funziona → resetta il contatore

### 10. Sidecar simulation

Simula un container sidecar che: - ogni 5 secondi controlla un
servizio - scrive log con timestamp - non si ferma mai
