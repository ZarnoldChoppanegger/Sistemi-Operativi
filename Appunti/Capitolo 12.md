# Capitolo 12
# Realizzazione del file system

## Struttura del file system

I dischi vengono utilizzati come memoria secondaria proprio perché è facile riscrivere localmente ed è efficiente per operazione di lettura e scrittura. Il file system si ocupa di rendere ancora pù efficiente queste operazioni in maniera tale da memorizzare, individuare e recuperare facilmente i dati.

Il file system viene di solito stratificato in diversi livelli:

* **Controllo del'I/O:** contiene i driver del dispositivo e si occupa di mandare e ricevere dati dalla memoria centrale.
* **File system di base:** Serve a leggere e a scrivere nel disco.
* **Modulo di organizzazione dei file:** conosce la disposizione dei file nel disco insieme agli indirizzi logici e fisici, consente infatti la traduzione da inidirizzi logici a fisici. Inoltre contiene la gestione dello spazio libero del disco.
* **File system logico:** Gestisce i metadata dei file, quindi tutto quello che riguarda i file tranne i dati di essi. Mantiene le strutture dei file tramite dei **file control block** (FCB) per ogni file.

## Realizzazione del fil system

Per realizzare un file system si usano parecchie struttre dati sia nei dischi che nella memoria centrale. 

Nei dischi abbiamo:

* **Blocco di controllo di avviamento:** contiene le informazioni per avviare il sistema opertivo, se presente.
* **Blocco di controllo del volume:** contiene le informazioni riguardanti quel volume, come i blocchi presenti.
* **Struttura della directory:** usata per organizzare i file.
* **Blocco di controllo del file:** contiene informazioni sul file.

Nella memoria abbiamo:

* **Tabella di montaggio:** indica i dispositivi montati e il loro file system.
* **Cache della dircetory:** contiene i file più usati in modo da avervi un accesso più veloce.
* **Tabella di sistema dei file aperti:** contiene l'FCB di tutti i file attualmente aperti.
* **Tabella dei file aperti per processo:** contiene i file che un processo ha attualmente aperti, quello che contiene sono dei puntatori all'FCB della tabella di sistema dei file aperti.

Per aprire un file viene utilizzata la chiamata di sistema `open()`. La `open()` si assicura che il file sia già aperto, se lo è inserisce un elemento nella tabella dei file aperti per processo contenente il puntatore all'FCB della tabella di sistema dei file aperti. Se il file non è ancora stato aperto, lo cerca nel disco e quando lo trova inserisce il suo FCB nella tabella di sistema dei file aperti e successivamnte un puntatore a quell'FCB nella tabella dei file aperti per processo. 
La `open()` ritorna un puntatore a quel file, ma possiamo memorizzare in cache il nome effettivo del file in modo da accedervi più in fretta la prossima volta. Il nome dato all'elemento della tabella è detto **file descriptor** in UNIX e **handle file** in Windows.

### File system virtuali

È un modo che consente ai sistemi operitivi di interfacciarsi contemporaneamente con diversi file system.

UNIX utilizza un approccio orientato agli oggetti, creando una gerarchia:

* Nel primo strato c'è l'interfaccia del file system con i metodi `open()`, `read()`, `close()` e `write()`.
* Nel secondo strato c'è il **Virtual File System** che è un'interfaccia astratta che si andrà ad adattare al tipo di file system che si sta usando. Inoltre permette di rappresentare univocamente un singolo file in tutta la rete. Il VFS è basato sulla rappresentazione di un file detta **vnode** che contiene un indicatore numerico unico per tutta la rete di ciascun file.

Il VFS di Ubuntu è così composto

* **Oggeto inode** che rappresenta un singolo file.
* **Oggetto file** che rappresenta un file aperto.
* **Oggetto superblock** che rappresenta l'intero file system.
* **Oggetto dentry** che rappresenta i singoli elementi di una directory.

## Realizzazione delle directory

### Lista lineare

I file sono memorizzati come in una lista lineare. Quando voglio creare un file devo scorrere la lista per vedere se esiste un file con lo stesso nome. Quando voglio cancellare un file devo scorrere la lista alla ricerca di quel file, cancellarlo e lasciare lo spazio che utilizzava libero. Per contrassegnare uno spazio libero ho diversi modi: posso contrassegnare l'elemento come non usato, posso aggiungerlo a una lista di elementi non utilizzati oppure posso copiare l'ultimo file della lista nello spazio appena liberatosi.

Mi consente una ricerca binaria molto veloce e permette di elencare in ordine i file velocemente.

### Tabella hash

Ho comunque una lista ordinata ma fornisco dei puntatori ai file della lista che sono memorizzati in una tabella hash. La ricerca è ancora più veloce e le operazioni di inserimento e cancellazioni sono abbastanza semplici. C'è però il problema della **collisione** ovvero due elementi nella tabella che puntano allo stesso file.

## Metodi di allocazione

Devo gestire lo spazzio che alloco in maniera efficiente, come? Ho diversi metodi.

### Allocazione contigua

Ogni file occupa blocchi contigui, con questo ordinamento ho un accesso diretto e sequenziale efficienti, inoltre l'accesso è molto veloce perché, trattandosi di blocchi contigui, la testina non dovrà quasi mai muoversi.

Questo tipo di allocazione soffre di **frammentazione esterna**.

Un altro problema è determinare lo spazio necessario per l'allocazione di un nuovo file, se gli si da poca memoria si andrebbe contro ad errori, se invece glie ne si da tanta ho il rischio di avere memoria inutilizzata (quindi anche **frammentazione interna**).

Se ho un file che cresce molto lentamente durante il tempo (in termini di anni), quanta memoria devo allocare? Allocarne troppa porta a frammentazione interna quindi utilizzo un metodo che prevede di *estendere* (sempre in maniera contigua) i blocchi allocati al file, questo nuovo spazio allocato è chiamato **estensione**.

### Allocazione concatenata

Risolve tutti i problemi dell'allocazione contigua.

I file sono composti da liste concatenate che puntano a blocchi allocati casualmente nel disco, la directory contiene il puntatore al primo e all'ultimo blocco e ogni blocco contiene un puntatore al blocco successivo. Per leggere un file quindi occorre solo leggere i puntatori da un blocco all'altro e non è necessario dichiarare la dimensione del file al momento della sua creazione.

Presenta comunque alcuni svantaggi in quanto può essere usata efficentemente solo per file ad accesso sequenziale. Se voglio infatti accedere all'*i*esimo blocco di un file devo scorrere *i* puntatori fino a quello che mi serve, e ogni blocco prima di arrivare all'*i*esimo corrisponde a una lettura su disco.

Un altro svantaggio è che i puntatori occupano 4 byte ciascuno quindi ho uno spreco di memoria se un file è composto da tanti blocchi. 

Per risolvere questo problema posso organizzare blocchi contigui in **cluster** e far puntare ad essi piuttosto che ai blocchi (quindi se ho cluster di 4 blocchi ho meno puntatori da utilizzare).

Un'altra variante sarebbe quella dell'uso di una **tabella di allocazione dei file** (*file allocation table*, FAT). È semplice ma efficiente, viene allocata all'inizio di ogni volume, contiene un elemento per blocco ed è indicizzata dal numero del blocco, l'elemento in directory contiene il numero del primo blocco che va a ricercare nella FAT dove a sua volta è contenuto il numero del prossimo blocco e così via fino all'ultimo che è denotato da un valore speciale.

### Allocazione indicizzata

Risolve il problema dell'efficienza in file ad accesso diretto dell'allocazione concatenata. Raggruppa tutti i puntatori in un solo blocco detto **blocco indice**. Il problema è che un blocco indice occupa molto spazio quindi devo cercare di ridurre al massimo la frammentazione interna e creare blocchi esattamente per il numero di puntatori che mi servono, per fare ciò ci sono diversi schemi:

* **Schema concatenato:** Posso avere un blocco indice che contiene informazioni sul file e, nel caso il file si di grandi dimensioni un puntatore a un altro blocco indice che contiene altre informazioni del file.
* **Schema a più livelli:** Un blocco indice che punta a diversi blocchi indice che puntano a loro volta ai dati dei file.
* **Schema combinato:** Utilizzato da UNIX consiste nell'avere diversi livelli.
  * Il primo livello è un blocco indice che contiene 15 puntatori di cui 12 puntano a **blocchi diretti** (quindi ai dati del file) e 3 puntano a **blocchi indiretti**.
    * Il primo è un puntatore a un **blocco indiretto singolo** ovvero un blocco che punta a un altro blocco che punta ai dati.
    * Il secondo è un puntatore a un **blocco indiretto doppio** ovvero un blocco che punta a un blocco che punta a più blocchi che puntano ai dati.
    * il terzo è un **blocco indiretto triplo** è come quelli sopra ma più lungo.

## Gestione dello spazio libero

Devo tenere traccia dello spazio libero tramite una **lista dello spazio libero**, posso realizzarla in diversi modi.

### Vettore di bit

Realizzo un vettore di bit e ne assegno uno ad ogni blocco, i blocchi segnati a 1 sono liberi quelli a 0 sono occupati, efficiente e semplice ma il vettore di bit occupa troppo spazio (per un disco di un 1TB il vettore di bit occupa 256MB).

### Lista concatenata

Collegare tutti gli spazi liberi in una lista concatenata e tenere traccia del primo elemento. Non è efficiente perché se cerco un blocco di una cerca dimensione devo scorrere la lista, però capita raramente quindi va bene.

### Raggruppamento

Raggruppo i puntatori di *n* blocchi liberi nel primo di questi, quindi avrò *n* - 1 blocchi e l'ultimo blocco contiene indirizzi ad altri blocchi che contengono puntatori a blocchi liberi.
