# Capitolo 8
# Memoria centrale

### Hardware di base

La memoria consiste in un grande vettore di byte, ciascuno con il proprio indirizzo. La CPU ha il compito di prelevare le istruzioni dalla memoria in base a cosa richiede il contatore di programma, queste istruzioni possono a sua volta determinare ulteriori *load* o ulteriori *store*.

La memoria centrale e i registri del processono sono le uniche aree alle quali la CPU ha un accesso diretto. Quindi ogni istruzione che deve essere eseguita deve risiedere in una di queste due memoria, poiché se risiedono in memoria secondaria devono essere prima caricate nella CPU. 

La CPU esegue la lettura nei registri più volte in un ciclo di clock, per quanto riguarda la memoria centrale, impiega molti cicli di clock e ciò è un problema in quanto porterebbe richiedere molto tempo prima che un processo ottenga i suoi dati e quindi entrare necessariamente in **stallo** ( *stall* ). Il rimedio per questo problema è inserire una memoria veloche che si pone esattamente tra la CPU e la memoria centrale. Questa memoria è detta **cache**.

Oltre a fare ciò dobbiamo assicurarci che le operazioni vengano eseguite correttamente e quindi porre delle protezioni hardware che proteggano il sistema operativo dall'accesso dei processi utente.

Bisogna fare in modo che ogni processo utente abbia una sua zona di memoria riservata e che possa accedere solo e soltanto a quella. Per fare ciò basta inserire (hardware) due registri chiamati **registro base** e **registro limite** che indichino rispettivamente l'inizio della memoria riservata al processo e la fine. Se un processo cerca di accedere a memoria che è oltre questi due registri viene lanciata un'eccezione (trap) che il sistema operativo interpreta come un errore fatale.

Solo il sistema operativo può gestire questi due registri, inoltre essendo eseguito in modalità kernel può accedere a tutta la memoria (anche quella allocata per i processi utente). Questo privilegio consente di caricare i processi utente nelle zone di memoria a loro riservate.

### Associazione degli indirizzi

In genere un programma risiede in un disco sottoforma di file binario eseguibile. Per essere eseguito deve essere caricato in memoria, l'insieme dei processi che risiedono sul disco e aspettano di essere caricati in memoria per essere eseguiti forma la **coda d'ingresso** ( *input queue* ). La procedura normale è che viene pescato uno di questi processi, caricato in memoria, eseguito e, quando finisce, viene tolto dalla memoria e si segnala che la memoria che occupata è di nuovo libera.

Nella maggiorparte dei casi un programma utente prima di essere eseguiti deve compiere qualche passo. Durante questi passi gli indirizzi sono rappresentabili in modo diverso. Di solito gli indirizzi che troviamo nel programma sorgente saranno diversi di quelli del programma in esecuzione. L'associazione degli indirizzi iniziali a quelli finali può essere compiuta in uno di questi tre percorsi:

* **Compilazione:** Se nella fase di compilazione si conoscono già gli indirizzi dove il processo risiederà allora posso generare **codice assoluto**.
* **Caricamento:** Se non posso saperlo durante la compilazione dovrò calcolarlo durante il caricamento in memoria, in questo caso il compilatore deve generare **codice rilocabile**.
* **Esecuzione:** Se durante l'esecuzione posso spostare il processo da un indirizzo a un altro allora devo ritardare l'associazione degli indirizzi a tempo di esecuzione. Quasi tutti i sistemi operativi utilizzano questo metodo.

### Spazi di indirizzi logici e fisici

> Un indirizzo generato dalla CPU è chiamato **indirizzo logico**, un indirizzo caricato nel *memory address register* è chiamato **indirizzo fisico**. 

I metodi di associazione di indirizzi nelle fasi di compilazione e caricamento generano indirizzi logici e fisici identici. Tuttavia nella fase di esecuzione gli indirizzi fisici e logici sono diversi. Ho bisogno quindi di una soluzione hardware che mi permette di associare gli indirizzi logici a quelli fisici, questa soluzione è il **MMU** (memory management unit) che permette di convertire gli indirizzi logici in fisici tramite diversi metodi. 

Quello più usato è quello con il limite dove scelgo una base che indica l'inizio della parte di memoria assegnata al processo e un limite è la dimensione massima che può avere il processo. L'indirizzo logico sarà X mentre quello fisico sarà uguale alla base + X, questa somma non deve superare il limite altrimenti si andrà in *segmentation fault*. Questa tecnica ha un problema: se il mio programma ha delle locazioni fisse, se cambio base devo ricalcolare tutti gli indirizzi.

### Caricamento dinamico, linking dinamico e swapping

**TL;DR** 

* Il caricamento dinamico mi permette di caricare una procedura in memoria solo quando mi serve. 
* Il linking dinamico mi permette di cariacre una sola volta le librerie di sistema e renderle condivisibili a tutte le procedure risparmiando molto spazio.
* Lo swapping mi permette di togliere un processo dalla memoria centrale per metterlo in una **memoria ausiliaria** e viceversa in modo da risparmiare spazio.

## Allocazione contigua della memoria

> Tutta l'informazione dei programmi è memorizzata in indirizzi consecutivi (sia logici che fisici)

Uno dei metodi più semplici per l'allocazione della memoria è quello di dividere in più partizioni uguali la memoria e ogni partizione dovrà contenere un solo programma, viene chiamato **metodo a partizione fissa**.

**Vantaggi**
* Il carico di lavoro svolto dal sistema operativo è minore.

**Svantaggi**
* Il numero massimo di processi che possono essere allocati in memoria è prefissato.
* Si ha uno spreco di memoria perché un programma di solito ha dimensioni minori di quelle della partizione e lo spazio avanzato non può però essere utilizzata da un altro programma.
* Il programma può essere di dimensioni maggiori della partizione.

### Metodo a partizione variabile

Con il metodo a partizioni variabili mentre  un programma viene eseguito vengono create le partizioni.

Se si liberano due partizioni vicine possono essere occupate entrambe dallo stesso programma anche se non viene occupato tutto lo spazio.

**Vantaggi**
* Non c’è  un numero fisso di partizioni.
* Ogni partizione ha dimensioni uguali al programma. Dunque non c’è uno spreco di memoria.

Un processo occupa una parte di memoria quando viene caricato e quando finisce lascia quella memoria libera per altri processi, continuando con questo passo avrò la memoria con fette di spazio libero abbastanza piccole da non poter essere utilizzate per caricarci programmi (quindi ho spreco di spazio), questo problema viene chiamato **frammentazione esterna** e si risolve con la **compattazione**.

Per scegliere che processo inserire nello spazio della memoria libero ho diversi criteri:

* **First-fit:** assegna il primo blocco abbastanza grande da contenere il processo.
* **Best-fit:** assegna il blocco più piccolo che può contenere il processo (produce parti di memoria inutilizzare più piccole).
* **Worst-fit:** assegna il blocco libero più grande (produce parti di memoria inutilizzate più grandi quindi è meglio perché le posso riutilizzare se sono abbastanza grandi).

Come detto prima, la *compattazione* mi permette di rendere riusabile i piccoli spazi di memoria rimasti liberi tramite un "reset" della memoria, di solito vengono shiftati tutti i programmi attualmente in memoria alla fine. La compattazione è possibile solo se è possibile la riallocazione dinamica e si effettua nella fase di esecuzione.

## Segmentazione

La memoria è formata da segmenti, ogni segmento ha degli elementi al suo interno che sono identificati dal loro offset (misurato dall'inizio del segmento). Gli indirizzi nel segmento sono rappresentabile come una coppia <numero segmento, offset>. Purtroppo però gli indirizzi in memoria non sono rappresentati da una coppia bensì da un singolo valore. Per ottenere questo valore viene riutilizzata la tecnica della quale ho scritto prima (base + offset).

Viene attuata tramite la **tabella dei segmenti** che contiene, appunto, una coppia di elementi che sono la base del segmento e il suo limite. Da qui ottengo la base a cui aggiugnere l'offset e poss controllare che non vada oltre il limite.

In questo caso la memoria è allocata non contiguamente

## Paginazione

Un altro metodo che mi permette di allocare memoria non contiguamente è quello della paginazione che a differenza della segmentazione evita la frammentazione esterna e diminuisce la possibilità di frammentazione interna.

### Metodo di base per implementare la paginazione

Il metodo principale per implementare la paginazione consiste nel dividere la memoria fisica in blocchi di dimensione fissa detti **frame** e suddividere la memoria logica in blocchi fissi detti **pagine**

Ogni indirizzo generato dalla CPU è composto dal numero di pagine e dal suo offset di pagine. Il numero di pagine funge da indice nella **tabella delle pagine** che contiene tutti gli indirizzi base della memoria fisica di ogni pagina alla quale verrano combinati gli offset per ottenere l'indirizzo logico.

È presente anche una **tabella dei frame** che tiene traccia di tutti i frame allocati (e cosa allocano) e quelli non allocati.

### Implementazione hardware della tabella delle pagine

Posso implementare facilmente la tabella delle pagine tramite dei registri (quindi memorizzarla li), è incredibilmente efficiente ma mi permette di avere un numero di elementi molto basso (~256). Viene quindi inserita nella memoria centrale e si crea un **registro di base della tabella delle pagine** che punta direttamente ad essa, il problema è che ogni accesso in memoria mi richiede 2 accessi in memoria (uno per ritrovare l'elemento nella tabella delle pagine e uno per accedere all'elemento stesso).

Posso risolvere questo problema attraverso il **translation look-aside buffer** (TLB). Consiste in una memoria associativa (quindi coppie chiave->valore) ad alta velocità. Se cerco la pagina P, essa verrà ricerata parallelamente all'interno di questa memoria, se esiste una corrispondenza (hit) la cache da la pagina e ho un solo accesso in memoria, se non esiste corrispondenza (miss) cerco P nella tabella delle pagine e la restituisce (quindi due accessi in memoria). 

Posso calcolare il tempo medio di accesso in memoria tramite questa formula:

T.accesso medio = Hit time + ((1 - Hit ratio) * Tempo accesso in RAM)

### Protezione

Nella tabella delle pagine posso aggiungere dei bit che fanno introspezione sulle pagine:

* **Read-only:** Mi dice se la pagina è di sola lettura.
* **Validità:** Mi dice se la pagina è valida per il programma che sta cercando di accedervi (quindi non appartiene ad altri programmi).

Inoltre se due processi condividono memoria, le tabelle delle pagine dei due processi rimandano alla stessa posizione nella RAM.

### Problema della paginazione

La tabella delle pagine di ciascun processo è da qualche parte in RAM allocata contiguamente, è un problema in quanto voglio averla allocata in maniera non contigua. Allora applico la paginazione alla tabella delle pagine, per farlo devo ricordare gli indirizzi e quindi avere una tabella delle pagine della tabella delle pagine. Applico questo metodo ricorsivamente fino ad avere una tabella delle pagine che occupa un singolo frame in modo da avere tutto allocato non contiguamente. La ricostruzione dell'indirizzo è più lenta ma va bene.



