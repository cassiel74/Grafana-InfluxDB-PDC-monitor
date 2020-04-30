## Connessione ad InfluxDB 

Per connetterci ad Influx dobbiamo usare l'interfaccia a riga di comando (CLI da ora in poi) e visto che siamo sul nostro Raspi è sufficiente lanciare il comando che segue, senza preoccuparci di specificare *host*, *utente* (ed eventualmente porta, se diversa da quella di default). 

Quindi 
```
influx
```

siamo dentro.

## Primo database

Per creare un database in influx è sufficente lo standard SQL quindi basta semplicemente usare la sintassi:

<pre>CREATE DATABASE <DBNAME></pre>

Iniziamo a chiamare le cose con il proprio nome quindi il nostro primo database si chiamerà PDC

```sql
CREATE DATABASE pdc
```

altro passaggio, anch'esso comune a tutti i db manager, è quello di istruire influx su quale db stiamo operando. Lo facciamo con il comando ```USE```


```sql
USE pdc
```

## Breve intro su InfluxDB

InfluxDB organizza i dati in serie storiche (time series o misure) in cui ogni punto della serie rappresenta una un campione.

Un punto (data point) consiste di: 

- tempo 
- nome della misura
- uno o più valori

Si possono utilizzare anche dei *tags* che sono sostanzialmente stringhe di testo che vengono utilizzate come indici per ottimizzare le ricerche.

Il tempo rappresenta la cosidetta chiave primaria (primary key), la misura la tabella ed i tags e campi sono le colonne.

In sede di inserimento dati non è necessario specificare il tempo (timestamp) a meno che non si abbia la necessità di usarne uno diverso da quello di sistema (ad esempio per caricare dati storici).

Il formato di un data point è quindi

```
<measurement>[,<tag-key>=<tag-value>...] <field-key>=<field-value>[,<field2-key>=<field2-value>...] [unix-nano-timestamp]
```

Per approfondimenti si rimanda alla [documentazione ufficiale](https://docs.influxdata.com/influxdb/v1.7/).

## Creazione tabella (misura) ed inserimento dati

Per lo scopo di questo tutorial avremo bisogno di inserire dati di temperatura per mandata, ritorno e temperatura esterna. Quindi per quanto detto in precedenza useremo come nome della misura "temperature", un tag "sorgente" e come nome di campo "value".

Quindi

```sql
INSERT temperature,sorgente=mandata value=20
```

Quindi per inserire ulteriori punti

```sql
INSERT temperature,sorgente=mandata value=35
INSERT temperature,sorgente=ritorno value=33
INSERT temperature,sorgente=esterna value=18
```


## Utilizzo della clausola "SELECT"

To start with, you can retrieve all data from a measurement by using a command like below. This command will grab all fields and keys from the specified measurement.

```sql
SELECT * FROM temperature
```

Using that command with our sample data you should get a result like we have below.

```
SELECT * FROM temperature
name: temperature
time                sorgente    value
----                --------    -----
1586749433336403281 mandata     20
1586749435859176782 mandata     35
1586749438114881978 ritorno     33
1586749440223591274 esterna     18
```

## Utilizzo della clausola "WHERE"

Per selezionare soltanto la temperatura di mandata basta procedere nel modo canonico usana una SELECT associata alla condizione WHERE specificando i tag o campi di cui vogliamo il valore.

Come detto in precedenza i *tag* sono considerati come stringhe pertanto quando vengono utilizzati come termini di ricerca devono essere racchiusi tra apici singoli.


```sql
SELECT value FROM temperature WHERE sorgente='mandata'
```

```
name: temperature
time                value
----                -----
1586749433336403281 20
1586749435859176782 35
```

Abbiamo ottenuto le sole temperature di mandata.


## Autenticazione

Utilizzare credenziali di autenticazione è sempre una buona prassi, anche per installazioni locali. 

Come primo passo creiamo un utente amministratore quindi utilizziamo Influx dalla riga di comando (CLI) 

```sh
influx
```

e lanciamo questo comando, sostituendo a ```<password>``` una stringa complicata "quanto necessario". Per installazioni in rete locale privata si può usare una stringa semplice, se un futuro il servizio verrà esposto all'esterno allora sarà necessario aumentare il livello di complessità.

```sql
CREATE USER admin WITH PASSWORD '<password>' WITH ALL PRIVILEGES
```

Il nome dell'utente amministratore è arbitrario e non deve essere necessariamente ```admin```.

Creiamo adesso un utente per Grafana che abbia i privilegi di sola lettura (sempre per buona prassi)
```sql
CREATE USER grafana WITH PASSWORD '<password-super-segreta>'
GRANT READ ON pdc TO grafana;
```

