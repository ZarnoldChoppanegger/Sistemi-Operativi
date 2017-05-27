# Capitolo 3

# Processi

> Un processo è un programma in esecuzione.

Un processo contiene sempre: 
* Il suo PC (Progam Counter, che contiene la prossima istruzione da eseguire).
* Il suo Stack, dove contiene i dati temporanei (parametri di una procedure, variabili locali ecc...)
* A volte un heap dove è inserita la memoria allocata dinamicamente durante l'esecuzione del programma.

È bene puntualizzare che programma e processo sono due cose completamente diverse. Un programma è un'entità *passiva* (un file su disco) mentre il processo è un'entità *attiva* (con program counter e risorse associate). Possono infatti esistere due processi dello stesso programma.

Un processo può inoltre contenere un'ambiente dove viene eseguito altro codice, Java Virtual Machine ne è un esempio.

## Stato dei processi

Un processo viene etichettato come `new` appena entra nel sistema, successivamente può entrare in diversi stati:

* `new`: Come detto prima è lo stato del processo nonappena esso entra nel sistema.
* `ready`: Il processo viene caricato nella memoria principale e aspetta di essere eseguito.
  * Scheduling: sceglie il processo da eseguire dalla coda.
* `exec`: Il processo entra in esecuzione e gli viene assegnata CPU.
* `wait`: Il processo entra in coda wait quando sta aspettando un evento (input da parte dell'utente). Una volta che può esser eeseguito viene rimesso in coda `ready`.
* `terminated`: Quando il processo finisce, le risorse non vengono liberate immediatamente ma viene fatto nei ritagli di tempo in cui la CPU è ferma (o ha meno richieste).

## Process Control Block (PCB)

Ogni processo, quando entra nel sistema ha un suo PCB che contiene molteplici informazioni:

* **Stato del processo**: se è un processo nuovo, in coda di ready o in coda di wait ecc...
* **Program Counter**: contiene l'indirizzo della prossima istruzione da eseguire.
* **Registri della CPU**: contiene registri generali, stack pointer, accumulatori ecc... Tutte queste informazioni devono essere salvate, insieme al program counter nel caso in cui il processo venisse interrotto, per permettergli di riprendere l'esecuzione in maniera corretta.
* **Informazioni sullo scheduling**: tutte le informazioni necessarie per lo scheduling (priorità del processo, puntatori a code).
* **Informazioni sulla memoria**: tabella delle pagine, tabella dei segmenti (in base a che gestione della memoria è utilizzata).
* **Informazioni di accounting**: utilizzo della CPU, tempo di utilizzo della stessa, limiti di tempo ecc...
* **Informazioni sull'I/O**: dispositivi di I/O assegnati al processo, file aperti ecc...

## Scheduling dei processi

> È un sistema che sceglie i processi da inserire ed eseguire nel sistema

È utile per avere una multiprogrammazione efficiente.

### Code di scheduling

Ogni processo che entra nel sistema viene inserito nella **job queue**. Quando è sceto viene inserito in memoria e spostato nella **ready queue**. Questa coda è generalmente una lista concatenata contenente i puntatori al primo e all'ultimo *PCB*, ogni *PCB* ha a sua volta un campo puntatore che punta al *PCB* del processo successivo. Quando un processo viene eseguito e richiede la disponibilità di un dispositivo I/O viene spostato nella **wait queue**, una volta finito viene rimesso nella *ready queue*.

Quando un processo viene eseguito possono succedere diverse situazioni:

* Il processo attente I/O e viene spostato nella *wait queue*.
* Il processo crea un processo figlio e deve attendere la fine di esso.
* Il processo è rimosso forzatamente dalla CPU a causa di un'interruzione per essere reinserito nella *ready queue*.

Il processo continua questo ciclo fino a quando non termina del tutto la sua esecuzione.

### Scheduler

Un processo può trovarsi, durante la sua vita, in diverse code di scheduling. Lo **scheduler** è colui che si occupa di selezionare i processi da queste cose. Ce ne sono due tipi:

* **Scheduling a lungo termine**: Sceglie i processi dalla memoria secondaria per inserirli in quella principale (quindi nella ready queue).
* **Scheduling a breve termine**: Sceglie i processi nella ready queue per assegnarli CPU.

Questi due scheduler si differenziano principalmente per la frequenza con la quale sono utilizzati. Quello a breve termine viene richiamato molto spesso perché deve scegliere i processi da mettere in esecuzione e un processo ha una durata di millisecondi. Quello a lungo termine invece deve occuparsi del **grado di multiprogrammazione** del sistema, ovvero deve tenere d'occhio che la velocità con io inserisce in ready queue i processi sia uguale alla velocità con cui i processi escono dal sistema, possono passare quindi anche diversi minuti. 

Inoltre possiamo differenziare i processi in:

* **I/O Bound**: Che hanno prevalenza di operazioni I/O durante l'esecuzione.
* **CPU Bound**: Che utilizzano di più la CPU.

Lo scheduler a lungo termine deve assicurarsi che il numero di processi I/O Bound e CPU Bound sia quasi uguale all'interno del sistema in modo da davere esecuzioni più omogenee possibili.

Oltre a questi due scheduler possiamo anche inserirne uno a medio termine che si occupa di togliere i processi dalla memoria principale per ridurre il grado di multiprogrammazione, questa operazione si chiama **swapping** (si fa nel capitolo 8).

### Creazione di un processo

Durante la propria esecuzione, un processo può creare numerosi nuovi processi. Il processo creante si chiama **padre**, il processo creato si chiama **figlio**. I processi figlio possono a loro volta creare altri processi, creando così un **albero** di processi. La maggiorparte dei sistemi operativi identifica i processi tramite un intero identificativo chiamato **PID** (Process Identifier). Il processo `init` è il primo processo ad essere caricato e ha sempre il PID uguale a 1. Da questo processo si ramificano successivamente tutti i processi utente.

Quando viene creato un figlio, esso dovrà attingere risorse. Può farlo in due modi:

* Ottenerle direttamente dal sistema operativo.
* Ottenerle come un sottoinsieme delle risorse del processo padre.

Quest'ultimo è più consigliabile perché permette al genitore di creare un numero limitato di figli e di non attingere da tutte le risorse del sistema.

Quando un processo ne crea uno nuovo, dal lato dell'esecuzione ci sono due possibilità:

1. Il processo genitore esegue in maniera concorrente con il processo figlio.
2. Il processo genitore attende la fine di tutti o di uno dei processi figli.

Ci sono altre due possibilità per quanto riguarda lo spazio degli indirizzi dei processi figli:

1. Il figlio è una copia del processo genitore (stessi dati e programmi del genitore).
2. Al figlio viene caricato un nuovo programma da eseguire.

Un nuovo processo si crea tramite la chiamata di sistema `fork()`. Permette al genitore di comunicare con facilità con i processi figli perché la `fork()` assegna PID = 0 al processo figlio e al processo padre assegna il PID del figlio in modo da distinguere perfettamente i processi padre da quelli figlio, ritorna un valore minore di 0 se ci sono stati errori nella creazione del nuovo processo.

### Terminazione processi

Un processo termina quando finisce l'esecuzione dell'ultima istruzione e richiede di essere cancellato dal sistema tramite la chiamata `exit()`. Se il processo che termina è un processo figlio sposta tutte le sue informazioni nel processo genitore che è pronto ad accettarle se ha eseguito la chiamata `wait()`, una volta fatta questa operazione tutte le risorse del processo vengono eliminate dal sistema operativo. 

> Un processo può essere terminato da un altro processo a patto che l'altro processo sia suo genitore

Un processo che è terminato ma il cui genitore non ha ancora eseguito la chiamata `wait()` viene chiamato processo **zombie** in quanto tutte le risorse di esso sono ancora nel sistema. Un processo rimane, di solito, in questo stato per poco tempo perché i processi genitori chiamano la `wait()` periodicamente rilasciando tutte le risorse di questi processi.

Se un genitore termina la sua esecuzione prima della fine dei processi figli essi rimangono orfani. Su *Linux* e *UNIX* i processi orfani vengono assegnati come figli al processo `init` che chiama la `wait()` periodicamente rilasciando le risorse.

## Comunicazione tra processi

