# Capitolo 11
# Interfaccia del file system

> Un file è un insieme di informazioni correlate, registrate in memoria seondaria. La struttura di un file dipende dal suo tipo.

### Attributi dei file

Ogni file ha un nome che viene usato come riferimento, inoltre ha diversi attributi:

* **Nome:** nome simbolico del file.
* **Identificatore:** identifica il file nel file system.
* **Tipo**
* **Locazione:** puntatore al file.
* **Dimensione**
* **Protezione:** le informazioni che controllano gli accessi.
* **Ora, data e identificazione dell'utente**

### Operazioni sui file

Un file è un tipo di dato astratto quindi lo identificon in base alle operazione che posso fare su di esso, quelle basilari sono:

* **Creazione di un file**
* **Scrittura di un file**
* **Lettura di un file**
* **Riposizionamento di un file**
* **Cancellazione di un file**
* **Troncamento di un file**

Ogni volta che tento di fare un'operazione sopracitata su un file devo andare a ricercarlo nella sua directory. Per evitare questa continua ricerca ho una **tabella dei file aperti** che indica tutti i file, appunto, aperti in modo da non dover andare a ricercarli per ogni operazione. Ogni qualvolta un file viene chiuso viene rimosso dalla tabella dei file aperti.

Ogni file aperto contiene le seguenti informazioni:

* **Puntatore al file:** tiene traccia dell'ultima operazione di lettura e/o scrittura sottoforma di un puntatore alla posizione corrente del file.
* **Contatore file aperti:** dato che un file può essere aperto contemporaneamente da più processi, ho un contatore che mi dice quante istanze del file sono aperte in modo da togliere il file dalla tabella dei file aperti solo quando il contatore è uguale a zero.
* **Posizione nel disco del file**
* **Diritti di accesso**

I file possono anche avere dei **lock** che possono essere **esclusivi**(per scrittura) o **condivisi**(per lettura).

## Metodi d'accesso
### Accesso sequenziale

> Le informazioni del file vengono elaborate un record dopo l'altro.

### Accesso diretto

> Consente di leggere e/o scrivere nel file senza un ordine particolare. 

Il file è considerato come una sequenza di blocchi. Il blocco che utilizza l'utente per accedere al file è detto **blocco relativo** in quanto è sicuramente diverso dal blocco reale (ogni file inizia da un blocco 0 a un blocco max, non possono esserci più blocchi 0 quindi lo 0 di un file è diverso dallo 0 di un altro file).

## Struttra della directory e del disco

Ogni entità che contiene un file system è denominata **volume**. Ogni volume deve contenere le informazioni dei file che contiene, queste informazioni risiedono nella **directory del dispositivo**

### Directory a un livello

La struttura più semplice, tutti i file sono contenuti in una sola directory. Non posso avere più file con lo stesso nome.

### Directory a due livelli

In questa struttura ogni utente dispone della sua directory chiamata **directory utente**. In questo modo utenti diversi possono avere file con nomi uguali.

C'è un problema nel caso in cui si voglia accedere ai file di un altro utente. Alcuni sistemi non lo permettono mentre altri si, quelli che lo permettono consentono l'accesso ai file di un altro utente specificando il path completo per quell'utente (/utenteB/gta.sa).

È inoltre presente una directory utente che contiene i file di sistema, quando un utente vuole avviare un file di sistema la ricerca avviene prima nella sua directory e poi nella directory che contiene i file di sistema.

### Directory con struttura ad albero

Estensione della struttura a due livelli con la possibilità di creare un numero *n* di sottodirectory generando così una struttura ad albero. Mi permette di avere una struttura più organizzata. 

L'albero ha la directory radice chiamata **root**. Le directory vengono viste come dei file ma hanno un bit che, se settato a 1, le identifica come directory.

Ogni utente dispone di una **directory corrente**. Possiamo fare riferimenti **assoluti** o **relativi** ai file, i primi sono riferimenti che iniziano dalla root dell'albero e i secondi iniziano dalla directory corrente.

È importante capire come gestire la cancellazione di una directory. Alcuni sistemi permettono la cancellazione solo se la directory è vuota, altri anche se è piena con conseguente cancellazione di tutte le sottodirectory e files.

### Directory con struttura a grafo aciclico

Con questa struttura posso permettere la condivisione di file tra gli utenti.

Posso avere due tipi di condivisione ( *link* ):

* **Symbolic link:** sono delle semplici shortcut, quindi dei file che riferiscono a un altro file.
* **Hard link:** duplico le informazioni del file in entrambe le directory che lo contengono, non è una copia e i file sono identici.

Sorge il problema della gestione della cancellazione di file condivisi. 

Per quanto riguarda i symbolic link viene lasciato il file che fungeva da link in *dangling* finché l'utente non si accorge che quel file non punta più a niente. 
Per quanto riguarda gli hard link un file è cancellabile solo quando tutti i suoi riferimenti sono stati cancellati, si implementa con un contatore che conta il numero di riferimenti attivi per quel file, quando il contatore è 0 allora il file è cancellabile.

### Directory con struttura a grafo generale

Quando aggiungo dei collegamenti a una struttura ad albero vengono a crearsi dei grafi generali. Questi grafi ammettono cicli che sono problematici nel caso di una ricerca di un elemento (possono venire a crearsi loop infiniti) oppure quando si vuole cancellare un file condiviso.

Infatti se esistono cicli aggragati a quel file, il contatore dei riferimenti non potrà mai scendere fino a 0 in quanto è possibile che il file si stia autoriferendo. In questo caso è necessario un **garbage collector** che tolga questi autoriferimenti.

## Montaggio di un file system

Deve essere montato un file system per poter accedere a un file. Ovvero devo costruire la directory che conterrà quel file. 

Per montare un file system si fornisce al sistema operativo il nome del dispositivo e la sua locazione (detta **punto di montaggio**). Il tipo del file system può essere necessario specificarlo oppure è autodedotto dal sistema, infine il sistema annota in una tabella che un certo dispositivo è montato con un certo file system. Questa tabella mi permette di poter switchare facilmente tra i dispositivi montati nel sistema.

Alcuni sistemi operativi richiedono che il file system sia montato su una diretory vuota, altri anche su directory non vuote. Nell'ultimo caso i file che erano contenuti nella directory non sono più accedibili finché non viene smontato il file system.

### File system remoti

Posso accedere in maniera remota ad altri file system tramite diversi metodi:

* **File system distribuito**
* **FTP**
* **World Wide Web**
* **Cloud computing**

## Semantica della coerenza

Dice come comportarsi con file modificabili e condivisi. Le transazioni atomiche non sono possibili perché richiedono troppe scritture e letture su disco.

### Semantica UNIX

* Le scritture di un file aperto contemporaneamente da più utenti sono visibili a tutti gli utenti.
* Condividono il puntatore del file.

### Semantica delle sessioni

* Le scritture in un file aperto contemporaneamente sono visibili solo all'utente che modifica e non agli altri.
* Le modifiche apportate al file sono visibili solo agli utenti che lo aprono dopo che la modifica è stata registrata.

### Semantica dei file condivisi immutabili

Come da titolo

## Protezione

Devo proteggere i file da accessi impropri, come faccio?

### Controllo degli accessi

L'approccio più comune è rendere l'acceso dipendente all'utente. Ogni utente avrà regole di accesso al file diverso, queste regole sono riportate nella **lista di controllo degli accessi** (ACL, access control list).

### Controllo degli accessi in UNIX

A ogni sottodirectory (che può anche essere un file) sono associati 3 bit (rwx), chi ha r a 1 può leggere, chi ha w a 1 può scrivere e chi ha x a 1 può eseguire.


