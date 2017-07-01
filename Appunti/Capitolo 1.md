# Capitolo 1

## Definizione di Sistema Operativo

Una definizione comune di sistema operativo è:

> Il sistema operativo è il solo programma che funziona sempre nel calcolatore, viene chiamato *kernel*.

Oltre al kernel sono presenti due tipologie di programmi:

* I programmi di sistema che sono quelli associati al sistema operativo.
* I programmi applicativi che includono tutti i programmi non associati al funzionamento del sistema.

## Interrupt

È importante dire che i programmi applicativi non possono comunicare direttamente con l'HW. C'è infatti un flag che indica se si stanno eseguendo istruzioni utente o istruzioni kernel. Proprio per questo motivo l'utente ha un accesso limitato al set di istruzioni fornite dall'HW, mentre il sistema può accedere a tutte.

Per questo motivo vengono distinte due modalità durante l'esecuzione di un programma:

* Modalità Kernel dove vengono eseguite istruzioni che mantengono in vita il sistema operativo.
* Modalità Utente dove vengono eseguite istruzioni dei programmi applicativi.

Il sistema operativo ha bisogno di introdurre degli interruput ogni tot di tempo per terminare le istruzioni utente ed eseguire quelle kernel in modo da mantenere in vita il sistema.

## Sistemi Batch 

> Un sistema batch è un modo per gestire le operazioni di calcolo in un sistema.

Veniva utilizzato ai tempi delle schede perforate, tutte i dati e le istruzioni dei programmi erano caricati in una scheda che veniva inserita nel sistema, caricata in memoria ed eseguita, una volta finita la procedure ne incominciava un'altra. Non poteva essere interrotta in alcun modo fino alla fine dell'esecuzione. Ogni volta che una procedura finiva si doveva resettare la memoria tramite un un programma di gestione che era caricato in RAM. 

Questo sistema ha diverse pecche tra cui la più importante è che la CPU veniva utilzzata solo durante l'esecuzione della procedura (che risulta essere un tempo molto minore rispetto all'input e all'output). Se voglio ottimizzare questa cosa devo aumentare il *throughput* (numero job / tempo). Una possibile soluzione è mettere in buffer l'input in modo da creare una *job pool* e non avere tempi di attesa in input (visto che sono già caricati in RAM). Questo viene fatto tramite la tecnica di **spooling**.

## Multiprogrammazione

Per ottimizzare ancora di più l'utilizzo della CPU posso eseguire dei jobs mentre un altro è in output. Facendo così ho più jobs in esecuzione che vengono eseguiti a pezzi (solo se sono caricati RAM). Non c'è alcun tipo di interazione.

### Multiprogrammazione - Time Sharing

È un'estensione logica della multiprogrammazione. L'esecuzione dei jobs della CPU viene divisa in quanti temporali permettendo alla CPU di passare da un job all'altro simulando così un parallelismo di esecuzione dei jobs. Il quanto di tempo permette di interrompere l'esecuzione di un job e farne partire un altro. Ovviamente una volta finito il quanto di tempo il job può trovarsi in diversi stati:

* Se ha finito prima del quanto di tempo tante care cose non deve fare nient ma il quanto di tempo viene resettato.
* Se non ha finito viene fatto, prima di essere interrotto, uno snapshot dello stato attuale del job per poi essere ricaricato quando torna in esecuzione in modo da ripartire da dove aveva smesso, questa operazione viene chiamata **Context Switch**.

## Sistemi Paralleli

> Sono sistemi che hanno più CPU.

Abbiamo due tipi: *Tighly Coupled* e *Loosely Coupled*

Nei primi abbiamo più CPU accoppiate a una sola RAM, facendo così avremo dei colli di bottiglia nel caso in cui ci sian molti jobs attivi che, per esempio, vogliono scrivere in memoria, il perché, se non lo capisci, sei fesso. Per ovviare a questo problema vengono quindi in aiuto i sistemi *loosely coupled* dove a ogni CPU è associata una RAM.

## Sistemi Real-Time

> Sistemi operativi specializzati nell'eseguire programmi real-time.

Mi permettono di avere tempistiche ben definite nell'esecuzione dei programmi. Non deve essere necessariamente veloce, l'importante è che mi risponda entro un tempo limite predeterminato. Il sistema deve essere quindi deterministico, nel senso che si possa prevedere il tempismo (nei best/worst cae) di esecuzione di un programma.

## Sistemi Embedded

> Sistemi che vengono programmati insieme all'HW.

Sono strettamente ottimizzati per esso, hanno una singola funzione e non possono essere modificati dall'utente. (LE LAVATRICI)


