# Capitolo 3

# Processi

> Un processo è un programma in esecuzione.

Un processo contiene sempre: 
* Il suo PC (Progam Counter, che contiene la prossima istruzione da eseguire).
* Il suo Stack, dove contiene i dati temporanei (parametri di una procedure, variabili locali ecc...)
* A volte un heap dove è inserita la memoria allocata dinamicamente durante l'esecuzione del programma.

È bene puntualizzare che programma e processo sono due cose completamente diverse. Un programma è un'entità *passiva* (un file su disco) mentre il processo è un'entità *attiva* (con program counter e risorse associate). Possono infatti esistere due processi dello stesso programma.

Un processo può inoltre contenere un'ambiente dove viene eseguito altro codice, Java Virtual Machine ne è un esempio.
 emoria virtu
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

I processi che vengono eseguiti concorrentemente nel sistema possono essere *indipendenti* o *cooperanti*. 

* I primi sono tutti quei processi che non devono condividere alcun dato con altri processi e che non possono influenzarli o esserne influenzati. 
* I secondi sono quelli che devono condividere dati e che possono influenzare o essere influezati da altri processi.

Ci sono diversi vantaggi nell'avere processi che cooperano:

* **Condivisione d'informazioni:** In quanto più utenti vogliono accedere, per esempio, allo stesso file contemporaneamente e quindi non aspettare la fine di alcuni processi per poterci accedere.
* **Velocizzazione del calcolo:** Con la condivisione dei dati i processi possono essere suddivisi in sotto attività che vengono assegnate a processi diversi, in modo da velocizzarne il calcolo.
* **Convenienza:** Perché anche un solo utente ha la necessità di compiere più attività contemporaneamente.
* **Modularità:** Stessa cosa della velocizzazione del calcolo. Suddiviso i processi i moduli.

Per ottenere la cooperazione dei processi è necessario un meccanismo di *comunicazione tra processi* (**IPC**, *interprocess communication*). Esistono due possibilità: a **memoria condivisa** e tramite **scambio di messaggi**. 

Nel modello a memoria condivisa viene ritagliato un pezzo di memoria a cui possono accedere tutti processi permettendogli di comunicare scrivendo e leggendo da questa zona di memoria. Questo modello può essere più veloce del modello a scambio di messaggi perché necessita del kernel molto sporadicamente rispetto all'altro modello.

Nel modello a scambio di messaggi le informazioni vengono condivise tramide dei messaggi, appunto. È utile per piccole quantità di informazione ma è più lento perché i messaggi vengono generati tramite chiamate di sistema che necessitano l'intromissione del kernel impegnandolo.

### Sistemi a memoria condivisa - Produttore/Consumatore

I processi comunicanti allocano una memoria condivida nella quale scrivono e leggono dati. Questo sistema viene illustrato tramite il problema del **produttore/consumatore** dove un processo produce e un altro consuma. Per risolvere questo problema con la memoria condivisa viene creato un buffer condiviso tra i due processi, essi devono essere perfettamente sincronizzati in modo da non permettere al consumatore di consumare un prodotto non ancora creato. 

Con un buffer **limitato** il consumatore deve attendere se il buffer è vuoto e il produttore deve attendere se il buffer è pieno. Esempio in C:

``` C
shared int Buffer[n];
shared int in = 0, out = 0;

// Produttore
while(1){
  while((in + 1) % n == out) { }
  // Se il buffer è pieno il produttore
  // in questo loop e aspetta che il
  // consumatore lo svuoti
  Buffer[in] = Dato(); // Qui produce
  in = (in + 1) % n;
}

// Consumatore
while(1){
  while(in == out) { }  // Se il buffer è vuoto il consumatore aspetta
  Dato = Buffer[out];
  out = (out + 1) % n;
}

```

Questo metodo ammette un massimo di n - 1 elementi contemporaneamente presenti nel buffer.

### Sistemi a scambio messaggi - Send/Receive/Mailbox

Lo scambio di messaggi permette a due o più processi di comunicare tra di loro senza la necessità di avere memoria condivisa. Questo meccanismo richiede due chiamate primitive ovvero `send()` e `receive()`. I messaggi possono avere lunghezza fissa o variabile, nel primo caso si avrà un'implementazione più facile ma sarà più difficile programmarvici. Nel secondo caso avremo un'implementazione più complicata ma la programmazione sarà più facile.

Per permettere a due processi di comunicare serve un **canale di comunicazione** che è realizzabile in molteplici modi:

* Comunicazione diretta e indiretta.
* Comunicazione sincrona e asincrona.
* Gestione automatica o esplicita del buffer.

#### Naming

**Comunicazione diretta**

Per comunicare i processi devono conosce il nome del ricevente e del mittente. Le operazioni `send()` e `receive()` saranno implementate così:

``` C
send(P, message)    // Invia al processo P il messaggio
receive(Q, message) // Riceve dal processo Q il messaggio
```
Con questo schema si hanno queste caratteristiche:

* Tra una coppia di processi può esserci solo un canale di comunicazione e i processi devono conoscere la reciproca idendità.
* Un canale è associato a due soli processi.
* Esiste esattamente un canale tra ciascuna coppia di processi.

Questo schema è *simmetrico* nell'indirizzamento. Esiste anche quello *asimmetrico* dove solamente il mittende deve conoscere l'identità del processo a cui manda il messaggio mentre il ricevente avrà un id che verrà sostituito rispetto al processo che sta mandando il messaggio.

È un implementazione molto sconveniente visto che, se viene cambiato il nome di un processo, esso dovrà essere modificato in tutto il programma.

**Comunicazione indiretta**

Con la comunicazione indiretta i messaggi vengono inviati a una **mailbox** dalla quale i processi potranno, appunto, inviare e ricevere messaggi. Con questa implementazione non è necessario conosce il nome di alcun processo ma solo il nome della mailbox, la `send()` e la `receive()` sono implementate così:

``` C
send(A, message)    // Stessa cosa di sotto
receive(A, message) // Dove A è il nome della mailbox
```

Questo sistema ha queste caratteristiche:

* Tra una coppia di processi si stabilisce un canale solo se condividono una mailbox.
* Un canale può essere associato a più processi.
* Tra ogni coppia di processi possono esserci molteplici canali, ognuno corrispondente a una mailbox diversa.

Il problema adesso è: se tre processi P1, P2 e P3 condividono una mailbox, P1 gli manda un messaggio e P2 e P3 vogliono leggerlo, quale dei due processi leggerà il messaggio? Ci sono diverse soluzioni a questo problema:

* Si fa in modo che un canale sia associato a soli due processi.
* Si cerca di eseguire la `receive()` solo da un processo per volta.
* Si lascia scegliere al sistema operativo (tipicamente tramite un algoritmo *round robin* che assegnerà il messaggio ai processi tramite un turno prestabilito).

---

Una mailbox può essere associata a un processo o al sistema operativo. Se è associata al processo, esso è il proprietario e può solo ricevere messaggi mentre gli utenti possono solo mandarli. Facendo così non si ha il problema esposto sopra ma se il processo termina la mailbox viene elminata insieme ai messaggi che contiene.

Se invece la mailbox è associata al sistema operativo questo problema non si pone in quanto ha vita autonoma. Con questo metodo mi è permesso anche di creare nuove mailbox, inviare e ricevere tramite la mailbox e rimuovere una mailbox.

---

#### Sincronizzazione

La `send()` e la `receive()` possono essere sincrone (bloccanti) o asincrone (non bloccanti):

* `send()` sincrona quando il processo che invia il messaggio si blocca nell'attesa che sia ricevuto.
* `send()` asincrona se, una volta mandato il messaggio, il processo continua la sua esecuzione.
* `receive()` sincrona quando il processo che riceve sta fermo fino a quando non riceve un messaggio.
* `receive()` asincrona permette di ricevere messaggi o valori nulli (non sta mai fermo).

Tramite la `send()` e la `receive()` bloccanti viene risolto facilmente il problema del produttore/consumatore.

#### Code di messaggi

Tutti i messaggi scambiati tra i processi risiedono in code temporanee, ne esistono tre tipi con capacità diverse:

* **Capacità zero:** La coda ha capacità zero quindi non può contenere alcun messaggio in attesa. Se un messaggio è inviato dal mittente esso si ferma in attesa che venga letto.
* **Capacità finita:** La coda ha una lunghezza n predefinita quindi può contenere n messaggi in attesa. Se la coda non è vuota e un messaggio è inviato dal trasmittente esso può continuare a inviarne altri finchè la coda non è piena, in quel caso si ferma.
* **Capacità infinita:** Il trasmittente non si ferma mai (povero).
