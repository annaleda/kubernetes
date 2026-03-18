# Bash -- Redirect, stdout, stderr

## 📦 Concetti base

In Linux ogni comando ha 3 stream:

  Stream   Numero   Descrizione
  -------- -------- ----------------
  stdin    0        input
  stdout   1        output normale
  stderr   2        errori

------------------------------------------------------------------------

## 🔁 Redirect base

### ➤ stdout (output normale)

``` bash
echo "ciao" > file.txt
```

✔ scrive "ciao" nel file

``` bash
echo "ciao" >> file.txt
```

✔ aggiunge (append)

------------------------------------------------------------------------

### ➤ stderr (errori)

``` bash
ls file_inesistente 2> error.txt
```

✔ salva solo gli errori

------------------------------------------------------------------------

## 🔀 Unire stdout e stderr

``` bash
comando > file.txt 2>&1
```

✔ tutto (output + errori) nello stesso file

------------------------------------------------------------------------

## 🕳️ /dev/null

### ➤ Ignorare output

``` bash
comando > /dev/null
```

✔ butta via stdout

------------------------------------------------------------------------

### ➤ Ignorare errori

``` bash
comando 2> /dev/null
```

✔ butta via stderr

------------------------------------------------------------------------

### ➤ Ignorare tutto

``` bash
comando > /dev/null 2>&1
```

✔ silenzio totale

------------------------------------------------------------------------

## 🧪 Esempi pratici

### ✔ Nascondere output ma controllare esito

``` bash
if curl -s localhost:80 > /dev/null; then
  echo "OK"
else
  echo "ERROR"
fi
```

------------------------------------------------------------------------

### ✔ Salvare solo errori

``` bash
curl localhost:9999 2> error.log
```

------------------------------------------------------------------------

### ✔ Salvare tutto

``` bash
curl localhost:80 > output.log 2>&1
```

------------------------------------------------------------------------

## 🧠 TL;DR

-   `>` = redirect stdout
-   `>>` = append stdout
-   `2>` = redirect stderr
-   `2>&1` = unisci stderr a stdout
-   `/dev/null` = cestino

------------------------------------------------------------------------

## 💥 Mental model

``` text
stdout (1) ----> output
stderr (2) ----> errori

> file        = salva output
2> file       = salva errori
> /dev/null   = ignora output
2> /dev/null  = ignora errori
```
