# Note nell'esecuzione del laboratorio

## Infrastruttura
Si usano container per maggior risparmio di risorse per i nodi, mentre la VM di cumulus per lo switch.

## Attivazione dei links
All'inizio, solo le porte `swp1` e `swp2` di Cumulus erano attive. Perciò:
```
net add bridge bridge ports swp1, swp2, swp3
net commit
```
È necessario poi dare un indirizzo ip diverso ad ognuno dei vari host:
```
ip add add 10.0.0.10X/24 dev eth0
```
dove `X` varia tra 0 e 2 nei diversi *end points*. 

Dopo aver fatto questo, tutte le macchine riescono a *pingarsi*.

## Aggiunta dei filtri
Su Cumulus, si prova ad usare i seguenti comandi per inserire delle regole per filtrare i pacchetti:
```
sudo ebtables -A FORWARD --in-interface swp1 -s ! 0a:b0:d6:49:7a:8d -j DROP
sudo ebtables -A INPUT --in-interface swp1 -s ! 0a:b0:d6:49:7a:8d -j DROP
sudo ebtables -A FORWARD --in-interface swp2 -s ! e6:2f:e9:63:20:61 -j DROP
sudo ebtables -A INPUT --in-interface swp2 -s ! e6:2f:e9:63:20:61 -j DROP
sudo ebtables -A FORWARD --in-interface swp3 -s ! d2:02:f1:aa:66:52 -j DROP
sudo ebtables -A INPUT --in-interface swp3 -s ! d2:02:f1:aa:66:52 -j DROP
```

anche qui la `X` va sostituita con il numero dell'interfaccia e `__MAC_X__` con l'indirizzo MAC della macchina collegata a quell'interfaccia.

Nonostante gli indirizzi MAC siano giusti, le macchine riescono ancora a pingarsi: nessun filtro viene applicato. Il comportamento atteso dovrebbe essere che i pacchetti in arrivo sulla macchina 2 dovrebbero essere scartati.

### Soluzione
Dopo il `-s` si specifica il source Mac. Se si lascia il `!`, si sta dicendo di mandare in `DROP` tutti i pacchetti con indirizzo diverso da quello specificato. Si vuole quindi che i pacchetti mandati in `DROP` siano quelli con quell'indirizzo, e allora bisogna rimuovere il `!`. La giusta sequenza di comandi è, quindi:
```
sudo ebtables -A FORWARD --in-interface swp1 -s ! 0a:b0:d6:49:7a:8d -j DROP
sudo ebtables -A INPUT --in-interface swp1 -s ! 0a:b0:d6:49:7a:8d -j DROP
sudo ebtables -A FORWARD --in-interface swp2 -s ! e6:2f:e9:63:20:61 -j DROP
sudo ebtables -A INPUT --in-interface swp2 -s ! e6:2f:e9:63:20:61 -j DROP
sudo ebtables -A FORWARD --in-interface swp3 -s d2:02:f1:aa:66:52 -j DROP
sudo ebtables -A INPUT --in-interface swp3 -s d2:02:f1:aa:66:52 -j DROP
```

## Punto oscuro
Se si esegue solo il comando 
```
sudo ebtables -A FORWARD --in-interface swp3 -s d2:02:f1:aa:66:52 -j DROP
```
si ottiene lo stesso effetto desiderato, mentre eseguendo solo 
```
sudo ebtables -A INPUT --in-interface swp3 -s d2:02:f1:aa:66:52 -j DROP
```
la macchina pinga ed è pingabile.

Controllare con wireshark cosa succede sul link nel secondo caso.
