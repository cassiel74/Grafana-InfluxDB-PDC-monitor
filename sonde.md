# ENERGETIC(A)MBIENTE


## Digitemp

Per il collegamento delle sonde al Raspi è possibile utilizzare questa guida, molto semplice e *low cost*
https://martybugs.net/electronics/tempsensor/usb.cgi

Nella guida ci sono già molte informazioni per l'utilizzo del programma *digitemp* che consente di leggere dati di sensori di temperatura (ma in generale di dispositivi *1-wire*) dalla porta USB con un adattatore seriale.

**IMPORTANTE** : per utilizzare gli adattatori seriale => USB è necessario che l'utente *pi* (o altro utente che si utilizza per leggere i dati) faccia parte del gruppo *dialout* quindi verifichiamo se il nostro utente è in questo gruppo con il comando *groups*

```sh
$ groups
```

l'output deve contenere la stringa *dialout*, ad esempio
```
pi adm dialout cdrom sudo audio video plugdev games users input netdev gpio i2c spi
```
se non la contiene allora dobbiamo aggiungere l'utente che andrà a leggere i dati dalle sonde al gruppo *dialout*.  
Ad esempio nel caso  dell'utente *pi* utilizzeremo

```sh
$ usermod -a -G sudo pi
```

controlliamo che l'utente sia stato inserito nel gruppo.

```sh
$ su - $USER  # questo ci assicura che le nuove impostazioni vengano "rilette" senza effettuare il logout/login (o il reboot)
$ groups
```

Procediamo adesso con l'installazione di *digitemp* e alla scrittura dei file di configurazione per le due porte USB (le sonde possono essere collegate in parallelo ma non è questo il mio caso :) )

```sh
$ sudo apt-get install digitemp

# i comandi che seguono servono per interrogare le porte usb e creare i file di 
# configurazione per le successive letture dei dati dalle sonde
$ sudo digitemp_DS9097 -i -s /dev/ttyUSB0 -c .digitemp0
$ sudo digitemp_DS9097 -i -s /dev/ttyUSB1 -c .digitemp1
```
(i nomi utilizzati per i file di configurazione sono arbitrari e si può usare quel che si vuole)
A questo punto i due file sono stati creati (*.digitemp0* e *.digitemp1*) e possiamo interrogare le sonde.

```sh
$ digitemp_DS9097 -a -q -o "%.2C" -c ~/.digitemp0
17.44
```

*digitemp* ha diverse opzioni per restituire le letture, per chi volesse approfondire o conoscere il significato di quelle usate: `digitemp_DS9097 --help` oppure più in generale `man digitemp`

## Script per la scrittura dati su influx

Creiamo una directory *bin* ed un file al suo interno, ad es. *ds.sh*

```sh
$ mkdir bin
$ cd bin
$ touch ds.sh
```
Aprite il file con il vostro editor preferito ed inserite questo contenuto

```sh
#!/bin/bash 

ds0=$( digitemp_DS9097 -a -q -o "%.2C" -c ~/.digitemp0 )
# sleep 5
ds1=$( digitemp_DS9097 -a -q -o "%.2C" -c ~/.digitemp1 )

delta=`expr "${ds0} - ${ds1}" |bc` # calcolo del delta, non essenziale perché possiamo farlo direttamente in Grafana

curl -i -XPOST 'http://localhost:8086/write?db=pdc' --data-binary "pdc,variable=mandata value=$ds1
pdc,variable=ritorno value=$ds0
pdc,variable=delta value=$delta"

```

## Crontab

A questo punto non resta che schedulare l'inserimento dei dati nel db, per farlo utilizziamo *cron*

Aggiungendo questa riga il nostro script verrà eseguito ogni due minuti: per tutti i giorni del mese (dom), tutti i mesi (mon) e tutti i giorni della settimana (dow).
```bash
# m h  dom mon dow   command
*/2 * * * * /home/pi/bin/ds.sh  >/dev/null 2>&1
```

Una alternativa può essere la seguente, che limita la scrittura ogni 2 minuti soltanto nei mesi invernali e aumenta a 4 minuti nei mesi estivi.

```bash
# m h  dom mon dow   command
*/2 * * 10,11,12,1,2,3 * /home/pi/bin/ds.sh  >/dev/null 2>&1
*/4 * * 4-9 * /home/pi/bin/ds.sh  >/dev/null 2>&1
```


