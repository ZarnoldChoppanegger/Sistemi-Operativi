# Capitolo 10
## Memoria secondaria

### Dischi magnetici

Sono la risorsa principale utilizzata come memoria secondaria, sono composti da:

* **Piatti** che hanno una forma piana e rotonda come i CD.
* **Testina** che legge i dati presenti sul disco.
* **Braccio del disco** che muove le testine in blocco.
* **Tracce** formate dalla divisione dei piatti.
* **Settori** formati dalla divisione delle tracce.
* **Cilindro** che è l'insieme delle tracce corrispondenti a una posizione del braccio.

Vengono considerati due valori: 

* **Velocità di trasferimento** misurata dalla velocità con cui i dati vengono inseriti all'interno del disco. 
* **Tempo di posizionamento** che consiste a sua volta in due componenti:
  * **Tempo di ricerca** che consiste nel tempo che il braccio impiega per spostarsi nel cilindro desiderato.
  * **Latenza di rotazione** il tempo necessario affinché il settore desiderato si porti sotto la testina.
  
### Dischi a stato solido

Sono più veloci dei dischi magnetici, più piccoli, molto meno rumorosi, più costosi (soldi), meno dispendiosi (energia).

## Scheduling del disco

Ogni richiesta di lettura e scrittura viene inserita in una coda, come faccio a scegliere quale richiesta eseguire? Ho diversi metodi.

### Scheduling FCFS (First Come First Served)

È la forma più semplice di sheduling ma non garantisce la massima velocità di servizio. 

> Il braccio si muove in base all'ordine delle richieste (se ho una richiesta al settore 2 e una al settore 170 faccio un botto di spostamenti ed è poco efficiente).

### Scheduling SSTF (Shortest Seek Time First)

> Sceglie la richiesta che da il minimo tempo di ricerca.

È una forma di scheduling per brevità (come SJF) quindi soffre di *starvation*.

### Scheduling SCAN

> Il braccio  si muove da un estremo all'altro soddisfacendo tutte le richieste nel tragitto.

Viene anche chiamata **algoritmo dell'ascensore**.

### Scheduling C-SCAN

> Uguale a SCAN ma quando arriva a un estremo torna all'altro senza soddisfare richieste nel tragitto.

Si comporta come una lista circolare.

### Scheduling LOOK

> Il braccio si muove in una direzione ma se non ci sono più richieste in quella direzione allora cambio direzione.

Esiste anche la variante C-LOOK.

### Blocco d'avviamento

Affinché un calcolatore possa funzionare ha bisogno di un programma di avviamento chiamato **bootstrap** che inizializza tutti i registri ecc... in modo da avviare in sicurezza il sistema operativo. Inizialmente veniva inserito in un **ROM**(Read Only Memory) in modo da permettere un caricamento veloce ed evitare possibli virus, il problema era che non poteva essere cambiato una volta inserito quindi si è pensato di inserire nella ROM un **bootstrap loader** che esegue il bootstrap che risiede nel disco.

## Strutture RAID

Mi permette di far lavorare in parallelo i dischi in modo da ottenere prestazioni e affidabilità migliore, ho diverse tecniche note con il nome di **RAID** (Redundant Array of Indipendent Disks).

### Mirroring

Posso aumentare l'affidabilità dei dati attraverso una tecnica chiamata **mirroring** che memorizza i dati due volte sul disco in modo che se uno si rompe ho comunque i dati salvaguardati dall'altro.

### Striping

Con il mirroring ho più affidabilità ma la velocità di accesso rimane uguale a quella di un disco singolo, per ovviare a questa pecca posso utilizzare la tecnica dello **striping** che si divide in:

* **Striping a livello bit:** distribuisco i bit di un byte nei dischi inseriti (se ho otto dischi allora ho un bit per disco).
* **Striping a libello di blocco:** invece dei bit distribuisco blocchi (ma va).

Con questa tecnica ho più velocità di accesso ma meno stabilità.

### Livelli RAID

Mi permette di mischiare le tecniche di mirroring e di striping per ottenere sia performance che affidabilità. Ci sono diversi livelli:

* **RAID 0:** Tecnica dello striping a blocchi.
* **RAID 1:** Tecnica del mirroring.
* **RAID 0 + 1:** Il livello 0 mi da prestazioni il livello 1 stabilità, l'unica pecca è che serve il doppio dello spazio (visto che nel mirroring devo creare copie).
* **RAID 1 + 0:** Prima divido in blocchi e poi faccio mirroring. Se si guasta un singolo disco nel RAID 0 + 1 tutto il settore diventa inaccessibile, nel RAID 1 + 0 diventa inaccessibile tutto il disco però la copia è ancora nell'altro disco ed è accedibile.
