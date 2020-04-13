ENERGETICAMBIENTE

- Requisiti</br>

Follow these steps first if you have a brand new rPi:

Download the latest lite Raspbian image from raspberrypi.org

Get a reasonable micro SD card - 32GB is the maximum supported size. I use a SanDisk Ultra

Burn the image to your SD card, I recommend using balenaEtcher.

Balena will eject your SD card once it is complete, re-mount your card by physically removing and re-inserting it, then create an empty file called ssh in the root of the SD card - this enables SSH access which we’ll need later.

Insert the SD card into your Pi, connect to your router via ethernet and power on.

Determine the auto-assigned IP address of the Pi by logging in to your router interface (see a guide on finding your router IP address here) and navigating to LAN / DHCP settings - the pi should be recognised as raspberrypi. Now is a good time to assign a static IP for your Pi to make life easier in the future. You’ll need to power cycle your Pi if you change from the auto assigned IP address.

SSH into the Pi using the IP address you have determined / assigned: ssh pi@<yourip>. You should probably update the password for the pi user now by running passwd.

(optional!) put your Pi in a case to keep it safe and cool. I found a cheap (£6 / $8) case with heatsinks and fan on Amazon.

- Installazione pacchetti</br>

```bash
sudo apt update
sudo apt upgrade -y
```

Influx

```bash
wget -qO- https://repos.influxdata.com/influxdb.key | sudo apt-key add -
source /etc/os-release
echo "deb https://repos.influxdata.com/debian $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/influxdb.list

echo "deb https://repos.influxdata.com/debian stretch stable" | sudo tee /etc/apt/sources.list.d/influxdb.list
echo "deb https://repos.influxdata.com/debian buster stable" | sudo tee /etc/apt/sources.list.d/influxdb.list

echo "deb https://repos.influxdata.com/ubuntu bionic stable" | sudo tee /etc/apt/sources.list.d/influxdb.list
echo "deb https://repos.influxdata.com/ubuntu xenial stable" | sudo tee /etc/apt/sources.list.d/influxdb.list

sudo apt update
sudo apt install influxdb
```

```bash
sudo systemctl unmask influxdb
sudo systemctl enable influxdb

sudo systemctl start influxdb
```

# Connesione InfluxDB 

To do this, we will need to launch up Influx’s command-line tool by running the command below.

You don’t have to worry about specifying an address to connect to as the tool will automatically detect the local installation of InfluxDB.

By default, InfluxDB has no users setup. In our next section, we will explore creating an admin user to lock down access to your InfluxDB. For now, however, we will quickly explore InfluxDB.

```
influx
```

# Primo database

Per creare un database in influx è sufficente lo standard SQL quindi basta semplicemente usare la sintassi:

<pre>CREATE DATABASE <DBNAME></pre>

Iniziamo a chiamare le cose con il proprio nome quindi il nostro primo database si chiamerà PDC,

```sql
CREATE DATABASE PDC
```

altro passaggio, anch'esso comune a tutti i db manager, è quello di istruire influx su quale db stiamo operando. Lo facciamo con il comando ```USE```


```sql
USE PDC
```

# Breve intro su InfluxDB

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

# Creazione tabella (misura) ed inserimento dati

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


# Utilizzo della clausola "SELECT"

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

# Utilizzo della clausola "WHERE"

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


# Autenticazione

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
GRANT READ ON PDC TO grafana;
```

# Installazione e configurazione Grafana

Again we need to add the Grafana packages to apt:

```sh
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee /etc/apt/sources.list.d/grafana.list
```

Procediamo adesso all'installazione, previo aggiornamento della lista dei pacchetti della nuova repo

```
sudo apt update && sudo apt install -y grafana
```

Il prossimo step è quello di abilitare il servizio all'avvio 

```sh
sudo systemctl unmask grafana-server.service
sudo systemctl start grafana-server
sudo systemctl enable grafana-server.service
```

A questo punto Grafana dovrebbe essere *up and running* ed in ascolto sulla porta di default, quindi visitate

http://localhost:3000

se siete sulla macchina (o il raspi) dove avete installato Grafana, altrimenti andrete a sostiuire *localhost* con l'indirizzo remoto opportuno (locale o pubblico).

Seguite quindi le istruzioni per il primo accesso con l'utente di default ```username``` e ```password``` sono ```admin```. Poi cambiate la password a vostro piacimento.


# Aggiungere Influx come *data source*

Autenticarsi, se già non lo siete, in Grafana e cercate "Data Sources".  Select "Add new Data Source” and find InfluxDB under "Timeseries Databases”.

<pre>
<img style='float: left; width: 300px;' src='img/1-data-source.png' />
</pre>

<pre>
<img style='float: left; width: 300px;' src='img/2-data-source.png' />
</pre>



- backup</br>
