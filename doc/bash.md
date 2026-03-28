- [ Home ](../readme.md)   | [ Teoria ](../arguments.md)   
--- 

| Comando                         | Descrizione                | Quando usarlo            | Note utili                       |          |   |
| ------------------------------- | -------------------------- | ------------------------ | -------------------------------- | -------- | - |
| **`ls`**                        | Lista file e cartelle      | Navigare nel filesystem  | `-l`, `-a` per dettagli          |          |   |
| **`cd`**                        | Cambia directory           | Spostarsi tra cartelle   | `cd ..` torna indietro           |          |   |
| **`pwd`**                       | Mostra directory corrente  | Sapere dove sei          |                                  |          |   |
| **`mkdir`**                     | Crea cartella              | Organizzare file         | `-p` crea path completi          |          |   |
| **`rm`**                        | Cancella file/cartelle     | Pulizia filesystem       | `-r` ricorsivo                   |          |   |
| **`cat`**                       | Mostra contenuto file      | Lettura veloce file      |                                  |          |   |
| **`less`**                      | Lettura file paginata      | Log o file lunghi        | `q` per uscire                   |          |   |
| **`head`**                      | Prime righe file           | Anteprima file           | `-n 10`                          |          |   |
| **`tail`**                      | Ultime righe file          | Log realtime             | `-f` per live                    |          |   |
| **`grep`**                      | Cerca testo                | Filtrare output          | `-i`, `-r`                       |          |   |
| **`awk`**                       | Estrai colonne             | Parsing dati             | molto potente                    |          |   |
| **`sed`**                       | Modifica testo             | Replace stringhe         | editing veloce                   |          |   |
| **`while true; do ... done`**   | Loop infinito              | Task background          | attenzione CPU                   |          |   |
| **`for i in ...; do ... done`** | Ciclo iterativo            | Automazioni              |                                  |          |   |
| **`ps`**                        | Lista processi             | Monitor sistema          |                                  |          |   |
| **`top`**                       | Monitor processi live      | Analisi performance      |                                  |          |   |
| **`kill`**                      | Termina processo           | Gestione processi        | `kill -9` forza                  |          |   |
| **`>`**                         | Sovrascrive output         | Scrittura file           | crea file se non esiste          |          |   |
| **`>>`**                        | Aggiunge output            | Logging                  | append                           |          |   |
| **`2>`**                        | Redireziona errori         | Debug script             | stderr                           |          |   |
| **`<`**                         | Input da file              | Leggere file nei comandi | es: `done < file.txt`            |          |   |
| **`<<`**                        | Here document              | Input multilinea         | script/bash inline               |          |   |
| **`<<<`**                       | Here string                | Input rapido             | es: `grep ciao <<< "ciao mondo"` |          |   |
| **`\|` (pipe)**                 | Passa output tra comandi   | Filtri e trasformazioni  | base della shell                 |          |   |
| **`&&`**                        | Esegue se precedente OK    | Controllo flusso         | chaining                         |          |   |
| **`\|\| `**                     | Esegue se precedente fallisce | Fallback              |                                  |
| **`&`**                         | Esegue in background       | Task asincroni           |                                  |          |   |
| **`wait`**                      | Aspetta processi           | Sincronizzazione         | script avanzati                  |          |   |
| **`xargs`**                     | Passa input come argomenti | Pipeline avanzate        | spesso con `find`                |          |   |
| **`find`**                      | Cerca file                 | Ricerca avanzata         | super usato                      |          |   |
| **`chmod`**                     | Cambia permessi            | Sicurezza file           | `+x` per eseguire                |          |   |
| **`chown`**                     | Cambia proprietario        | Gestione utenti          |                                  |          |   |
| **`apk`**                       | Package manager di Alpine Linux | Installare pacchetti | usato in immagini Docker Alpine |
| **`wc`** | Conta righe/parole/caratteri | Analisi file o output | spesso usato con pipe `\|` |


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
