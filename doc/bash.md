| Comando                         | Descrizione               | Quando usarlo            |                         |
| ------------------------------- | ------------------------- | ------------------------ | ----------------------- |
| **`ls`**                        | Lista file e cartelle     | Navigare nel filesystem  |                         |
| **`cd`**                        | Cambia directory          | Spostarsi tra cartelle   |                         |
| **`pwd`**                       | Mostra directory corrente | Sapere dove sei          |                         |
| **`mkdir`**                     | Crea cartella             | Organizzare file         |                         |
| **`rm`**                        | Cancella file/cartelle    | Pulizia filesystem       |                         |
| **`cat`**                       | Mostra contenuto file     | Lettura veloce file      |                         |
| **`less`**                      | Lettura file paginata     | Log o file lunghi        |                         |
| **`head`**                      | Prime righe file          | Anteprima file           |                         |
| **`tail`**                      | Ultime righe file         | Log realtime             |                         |
| **`grep`**                      | Cerca testo               | Filtrare output          |                         |
| **`awk`**                       | Estrai colonne            | Parsing dati             |                         |
| **`sed`**                       | Modifica testo            | Replace stringhe         |                         |
| **`while true; do ... done`**   | Loop infinito             | Task background          |                         |
| **`for i in ...; do ... done`** | Ciclo iterativo           | Automazioni              |                         |
| **`ps`**                        | Lista processi            | Monitor sistema          |                         |
| **`top`**                       | Monitor processi live     | Analisi performance      |                         |
| **`kill`**                      | Termina processo          | Gestione processi        |                         |
| **`>`**                         | Sovrascrive output        | Scrittura file           |                         |
| **`>>`**                        | Aggiunge output           | Logging                  |                         |
| **`2>`**                        | Error output redirect     | Debug script             |                         |
| **` \| ` (pipe)**               | Passa output tra comandi  | Filtri e trasformazioni  |                         |

Esempi

```bash
# Filtra le righe che contengono "ERROR", poi ordina e conta le occorrenze
cat log.txt | grep ERROR | sort | uniq -c

# Loop FOR: stampa numeri da 1 a 5
for i in {1..5}; do
  echo "Numero: $i"
done

# Loop WHILE infinito: stampa la data ogni 2 secondi
while true; do
  date
  sleep 2
done

# Loop WHILE infinito: chiama endpoint HTTP ogni 5 secondi (monitoring)
while true; do
  curl http://localhost:80
  sleep 5
done

# Legge un file riga per riga e stampa ogni linea
while read line; do
  echo $line
done < file.txt

# Sovrascrive il file scrivendo "hello" (cancella contenuto precedente)
echo "hello" > file.txt

# Aggiunge una riga senza cancellare il file
echo "second line" >> file.txt

# Conta il numero di righe del file e stampa il risultato
wc -l < file.txt
```
---
 - [guide redirect bash](./exercises/bash_redirect_guide.md)
 - [esempi bash](./exercises/bash_k8s_examples.md)
 - [esercizi bash](./exercises/bash_k8s_exercises.md)
 - [soluzioni bash](./exercises/bash_k8s_solutions.md)

---   
