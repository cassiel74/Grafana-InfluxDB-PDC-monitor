# ENERGETIC(A)MBIENTE

Come requisito per questa breve guida supponiamo di avere una installazione di un sistema *unix like* (Raspbian per raspberry ad esempio oppure una qualsiasi distro linux) funzionante ed in rete (locale/LAN o WAN) e relativo indirizzo IP. Nella guida verrà preso ad esempio il raspberry, da qui in avanti indicato con "Raspi".
Le modifiche da apportare sono minimali e pressoché tutte automatiche nel caso si voglia utilizzare un desktop .

Supponendo che il vostro Raspi sia attivo h24 (e debba restarci) è preferibile associargli un indirizzo IP statico anziché lasciarlo in balia del router e fargli assegnare un indirizzo diverso ogni volta che il vostro lease scade perché magari avete lasciato spento il raspi per troppo tempo per un motivo o per un altro.

Io ho un TP-LINK e la pagina di configurazione è la seguente. Il ```raspi-monitor``` è l'ultimo della lista con un ip statico 192.168.1.111.

<pre>
<img src='img/1-static.png' width='400px' />
</pre>

## DNS locale (una semplificazione utile)

Chi si connette da una macchina linux (ma credo anche su winzozz ci sia l'analogo) può andare ad aggiungere, subito dopo la prima riga del file ```/etc/hosts```, questa:

```
192.168.1.111   raspi
```

in modo da avere nel file qualcosa di simile (questo è il file del mio PC)

```
127.0.0.1	localhost localhost.localdomain debian
192.168.1.111   raspi

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

Così facendo possiamo connetterci al nostro Raspi (sia da browser che attraverso ssh) usando 

```
ssh pi@raspi
```

Anziché sgranare ogni volta tutto l'indirizzo Ip.

Chiaramente potete usare un *alias* (in realtà è il DNS locale) qualsiasi a posto di ```raspi``` ;-)

# Note

Tutti i comandi che seguiranno si suppone vengano lanciati usando un utente non privilegiato, che su una installazione Raspberry è comunemente l'utente **pi**.
Le parti in grigetto del codice rappresentano commenti (tutto ciò che segue il simbolo #) e possono pertanto essere copiati ed incollati per intero insieme alla parte dei comandi veri e propri. Non verranno eseguiti.

Le parti in grigetto che contegono comandi (in alcuni commenti sono stati inseriti) possono essere utilizzate come indicato.
