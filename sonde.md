# ENERGETIC(A)MBIENTE


## Digitemp

```sh
sudo apt-get install digitemp

sudo digitemp_DS9097 -i -s /dev/ttyUSB0 -c .digitemp1
sudo digitemp_DS9097 -i -s /dev/ttyUSB1 -c .digitemp0

```


## Script

Script per la scrittura dati su influx

```sh
#!/bin/bash 

ds0=$( sudo digitemp_DS9097 -a -q -o "%.2C" -c ~/.digitemp0 )
sleep 5
ds1=$( sudo digitemp_DS9097 -a -q -o "%.2C" -c ~/.digitemp1 )

delta=`expr "${ds0} - ${ds1}" |bc`

curl -i -XPOST 'http://localhost:8086/write?db=mydb' --data-binary "pdc,variable=mandata value=$ds1
pdc,variable=ritorno value=$ds0
pdc,variable=delta value=$delta"

```

## Crontab

```bash
# m h  dom mon dow   command
*/2 * * 10,11,12,1,2,3 * /home/pi/bin/ds.sh  >/dev/null 2>&1
*/2 * * 4,5,6,7,8,9 * /home/pi/bin/ds.sh  >/dev/null 2>&1

*/3 * * * * /home/pi/bin/pciv.sh  >/dev/null 2>&1

0 0 * * * influxd backup -portable /home/pi/backup-influx

0 9-12 * * * rsync -avz --exclude-from 'exclude-list.txt' -e ssh /home/pi/ rmorelli@192.168.1.107:/media/disco2/backup-raspi/
```


