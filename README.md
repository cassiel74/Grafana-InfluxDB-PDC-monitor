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

```CREATE DATABASE PDC```

altro passaggio, anch'esso comune a tutti i db manager, è quello di istruire influx su quale db stiamo operando. Lo facciamo con il comando ```USE```


```
USE PDC
```

# Creazione tabella (misura) ed inserimento dati

To do this, we must first get a basic understanding of InfluxDB’s datastore.

Data in InfluxDB are sorted by “time series“. These “time series” can contain as many or as little data points as you need. Each of these data points represents a single sample of that metric.

A data point consists of the time, a measurement name such as “temperature”, and at least one field. You can also use tags which are indexed pieces of data that are a string only. Tags are essential for optimizing database lookups.

If you are familiar with the general layout of an SQL table, you can consider “time” to be the primary index, measurement as the table name, and the tags and fields as the column names.

You do not need to specify the timestamp unless you want to specify a specific time and date for the data point.

Below we have included the basic format of an InfluxDB data point.

<measurement>[,<tag-key>=<tag-value>...] <field-key>=<field-value>[,<field2-key>=<field2-value>...] [unix-nano-timestamp]
If you would like to learn more about the InfluxDB line syntax, then you can check out the InfluxDB official documentation.

5. Now that we have a basic understanding of data in InfluxDB, we can now proceed to add our very first data point to our database.

For this example database, we are going to be storing measurements of the “temperature” of various locations around a house.

So for this, we will be inserting data points with a measurement name of “temperature” and a tag key of “location“, and a field key of “value“.

For our first sample point, we will be saying the location is the “living_room“, and the value is “20” .

INSERT temperature,location=living_room value=20
6. To make the data more interesting for showing off “selecting” data in InfluxDB, let’s go ahead and add some more random data.

Enter the following few commands to enter some extra data into our database. These are just variations of the above “INSERT” command but with the value and location adjusted.

INSERT temperature,location=living_room value=10
INSERT temperature,location=bedroom value=34
INSERT temperature,location=bedroom value=23
7. Now that we have some sample data, we can now show you how to query this data using “SELECT“.

To start with, you can retrieve all data from a measurement by using a command like below. This command will grab all fields and keys from the specified measurement.

SELECT * FROM temperature
Using that command with our sample data you should get a result like we have below.

name: temperature
time                location    value
----                --------    -----
1574055049844513350 living_room 20
1574055196564029842 living_room 10
1574055196576516557 bedroom     34
1574055197188117724 bedroom     23
8. Let’s say that you now only wanted to retrieve the temperature of the bedroom. You can do that by making use of the “WHERE” statement alongside a “SELECT” statement.

We also specify the name of the tags/fields that we want to retrieve the values from.

When querying tag fields, you need to remember that all tags are considered to be strings. This means that we must wrap the value we are searching for in single quotes.

SELECT value FROM temperature WHERE location='bedroom'
With that command, you should receive the following data set, showing only the temperature value in the bedroom.

Which in our example data’s case, this should be 34 and 23.

name: temperature
time                location    value
----                --------    -----
1574055049844513350 living_room 20
1574055196564029842 living_room 10
1574055196576516557 bedroom     34
1574055197188117724 bedroom     23
9. At this point, you should now have a basic understanding of InfluxDB and how its data works.

 Adding Authentication to InfluxDB
1. The next step is to add extra authentication to our InfluxDB installation on the Raspberry Pi. Without authentication, anyone could interact with your database.

To get started, we need to first create a user to act as our admin.

To create this user, we must first load up the InfluxDB CLI tool by running the following command.

influx
2. Within this interface, we can create a user that will have full access to the database. This user will act as our admin account.

To create this admin user, run the following command within InfluxDB’s CLI tool.

Make sure that you replace <password> with a secure password of your choice.

CREATE USER admin WITH PASSWORD '<password>' WITH ALL PRIVILEGES
This command will create a new user called “admin” with your chosen password and grant it all privileges.

3. With that done, you can now exit out of InfluxDB by typing in “exit” and pressing ENTER.

4. Our next job is to modify the InfluxDB config file to enable authentication.

We can begin editing the file by using the command below.

sudo nano /etc/influxdb/influxdb.conf
5. Within this file, use CTRL + W to find the [HTTP] section and add the following options underneath that section.

Find

[HTTP]
Add Below

auth-enabled = true
pprof-enabled = true
pprof-auth-enabled = true
ping-auth-enabled = true
6. Once added, save the file by pressing CTRL + X, then Y, followed by ENTER.

7. Now, as we have made changes to InfluxDB’s configuration, we will need to go ahead and restart the service by using the following command.

Restarting the service will ensure that our configuration changes are read in.

sudo systemctl restart influxdb
8. As we have now turned InfluxDB’s authentication on, we will need to enter our username and password before using the InfluxDB CLI tool.

You can use the “auth” command within the tool or pass the username and password in through the command line, as we have below.

influx -username admin -password <password>
Hopefully, at this point, you will now have successfully set up InfluxDB on your Raspberry Pi. You should now have a basic understanding of InfluxDB as well as have authentication mode enabled.

You can easily see how using this database model will make storing data from sensors and other sources very useful.

If you have run into any issues with getting a Raspberry Pi InfluxDB up and  running, then feel free to drop a comment below.

- periferiche</br>

- configurazione influx</br>

- configurazione grafana</br>

- backup</br>
