# Capitolo 5
# Sincronizzazione dei processi

I processi possono essere eseguiti concorrentemente o parallelamente ma in alcuni casi possono venire a crearsi delle situazioni dove ho dei problemi nella gestione dei dati condivisi dei processi. Per esempio scriviamo l'algoritmo del produttore consumatore che ci permette di avere non più `BUFFER_SIZE - 1` elementi bensì `BUFFER_SIZE` aggiungendo un contatore che viene incrementato quando produco e decrementato quando consumo:

**Produttore**

``` c#
while (true){
  // Produco elemento in next_produced
  while (contatore == BUFFER_SIZE)
    ; // Aspetto e non faccio niente
  buffer[in] = next_produced;
  in = (in + 1) % BUFFER_SIZE;
  contatore++;
}
```

**Consumatore**

``` c#
while (true){
  while (contatore == 0)
    ; // Aspetto e non faccio niente
  next_consumed = buffer[out];
  out = (out + 1) % BUFFER_SIZE;
  contatore--;
}
```
Se eseguo questo codice concorrentemente e c'è prelazione durante l'esecuzione di un decremento o incremento del contatore, posso perdere la semantica di queste due esecuzione (es. 5+1-1 = 6) in quanto dipende dall'ordine di esecuzione delle istruzioni. In questo caso vado in contro a una **Race Condition**.

## Problema della sezione critica 

> La sezione critica di un processo è la sua zona di codice dove modifica o accede a variabili condivise con altri processi.

Se un processo è dentro la sua sezione critica gli altri processi non posso entrare nella loro finché esso non ne esce. Ogni processo deve chiedere il permesso di entrare in sezione critica e lo fa attraverso una sequenza di istruzioni che vengono definite come **sezione d'ingresso**. Una soluzione del problema della sezione critica deve soddisfare queste tre proprietà:

* **Mutua esclusione:** Se un processo è in sezione critica nessun altro processo può entrarvici.
* **Progresso:** Se nessun processo è in sezione critica e alcuni chiedono di entrarci solo chi ha fatto la richiesta può partecipare alla decisione di chi dovrà entrarci.
* **Attesa limitata:** Un processo può fare un tot di richieste di entrare in sezione critica (senza successo), dopodichè se fa richiesta di nuovo entrerà in sezione critica per diritto.

Per la gestione della sezione critica ci sono due strategie principali: *kernel con diritto di prelazione* e *kernel senza diritto di prelazione*.

Con l'ultimo non si avranno mai sezioni critiche visto che i processi non possono subire prelazione mentre nel primo si. È comunque preferibile un kernel con diritto di prelazione in quanto ha una maggior prontezza nelle risposte e sono più adatti nella programmazione real-time.

## Soluzione di Peterson

Una classica empsoluzione per il problema della sezione critica è la soluzione di **Peterson**. È limitata a due soli processi, Pi e Pj. Questa soluzione richide che i processi condividano due dati:

* `int turn;`
* `boolean flag[2];`

La variabile `turn` segnala quale dei due processi può avere accesso alla sezione critica, `flag` indica quali dei due processi sono pronti a entrare in sezione critica.

``` c#
// Codice processo Pi

do {
  // Sezione critica
  flag[i] = true;
  turn = j;
  while (flag[j] && turn == j);
  // Sezione non critica
  flag[i] = false;
} while(true);
```

Da questo codice è facile capire che tutte e tre le condizioni di sezione critica sono soddisfatte.

## Hardware per la sincronizzazione

Le soluzioni basate sul software non garantiscono il loro funzionamento su tutte le architetture moderne. Sono infatti state implementate altre soluzione di tipo hardware che permettono di risolvere il problema della sezione critica.

Su sistemi a singolo processore basta disablitare la prelazione quando il processo esegue codice che può portare a *race condition* (quindi codice che modifica o accede a dati condivisi con altri processi).

Tuttavia nei sistemi a multiprocessore questo sistema non è efficiente in quanto la richiesta di disabilitazione della prelazione può essere molto lenta in quanto deve sincronizzarsi con tutti gli altri processori. Per questo motivo sono state inserite delle speciali istruzioni che garantiscono l'atomicità della loro esecuzione: `test_and_set()` e `compare_and_swap()`.

La prima è implementata come segue:

``` c#
boolean test_and_set(boolean *obiettivo)
{
  boolean valore = *obiettivo;
  *obiettivo = true;
  return valore;
}
```

Posso quindi realizzare la mutua esclusione aggiungendo una variabile condivisa `boolean lock = false` e fare una cosa del genere:

``` c#
do
{
  while (test_and_set(&lock));
    /* Sezione critica */
  lock = false;
  /* Sezione non critica */
}while(true);
```

## Lock Mutex

Le soluzioni hardware descritte prima sono difficili, per i programmatori, da utilizzare quindi sono state implementate dai progettisti dei sistemi operativi delle soluzioni più accessibili.

Una di queste è il **lock mutex**. Viene utilizzato per proteggere le regioni critiche quindi prevenire race condition. Quando un processo entra in sezione critica acquisisce il lock e lo rilascia quando ne esce. Queste due operazioni vengono eseguite tramite le due funzioni `acquire()` e `release()`.

Un lock mutex ha una variabile booleana `available` che indica la disponibilità del lock. Se è a true (quindi è disponibile) la chiamata `acquire()` ha successo, se invece non è disponibile la chiamata si blocca e viene rieseguita fin quando non avrà successo. Queste due funzioni sono implementate così:

`acquire()`

``` c#
acquire()
{
  while(!available)
    /* Aspetta */
  available = false;
}
```
`release()`

``` c#
relase()
{
  available = true;
}
```

Il problema di questa soluzione è che è presente l'**attesa attiva** ( *busy waiting* ). Se mentre un processo si trova nella sua sezione critica, ogni altro processo che vuole entrarvici deve ciclare continuamente nella chiamata di `acquire()` sprecando cicli di clock. Questo tipo di lock mutex viene anche chimato **spinlock** (perché gira).

Lo **spinlock** permette di non avere *context switch* quindi è utile se si prevede con quasi certezza che tutti i lock verranno rilasciati in tempi brevi.

## Semafori

> Un Semaforo S è una variabile intera a cui si può accere, escludendo l'inizializzazione, solo tramite due operazioni atomiche wait e signal.

`wait()`

``` c#
wait(S)
{
  while(S <= 0)
   ; // Attendo
  S--;
}
```

`signal()`

``` c#
signal(S)
{
  S++;
}
```
### Uso dei semafori

Possiamo distinguere tra **semafori contatori** e **semafori binari**. 

* Quelli binari si comportano nella stessa maniera dei lock mutex.
* Quelli contatori vengono principalmente utilizzati nel caso si debba condividere una risorsa solo un determinato numero di volte.

Il semafono è inizialmente impostato a zero, se un processo chiede una risorsa viene invocata `wait()` sul semafono che ne decrementa il valore, quando il processo finisce con la risorsa viene chiamata `signal()` sul semaforo che ne aumenta il valore. Quando il semaforo vale 0 significa che tutte le risorse sono occupate quindi se qualcuno prova a richiederla entra in loop nella chiamata della `wait()` finché un processo non chiama `signal()` liberando una risorsa.

### Implementazione dei semafori

Nei semafori abbiamo lo stesso problema che avevamo nei lock mutex ovvero l' *attesa attiva*. Per superare questo ostacolo possiamo modificare le definizioni di `wait()` e `signal()` in modo da poter rispettivamente *bloccare* un processo invece di farlo entrare in loop e *sbloccarlo* quando si libera una risorsa.

Ci sono due chiamate che permettono di fare queste due operazioni e cioè `block()` e `wakeup()`.

* `block()` permette di inserire il processo che chiama la wait in *coda di wait*.
* `wakeup()` permette di pescare un processo che è in attesa di un semafono dalla *coda di wait*.

Le nuove implementazioni di `signal()` e `wait()` sono:

``` c#
typedef struct
{
  int value;
  struct process *list;
} semaphore;

wait(semaphone *S)
{
  S->value--;
  if(S->value < 0)
  {
    // Aggiungo questo processo a S->list;
	block();
  }
}

signal(semaphone *S)
{
  S->value++;
  if(S->value >= 0)
  {
    // Tolgo un processo P da S->list
	wakeup(P);
  }
}
```

### Stallo e attesa indefinita

La realizzazione di un semaforo con coda d'attesa può condurre a situazioni in cui due o più processi sono in **stallo**, ovvero se ciascun processo dell'insieme attende un evento che può essere causato solo da un altro processo dell'insieme.

Può anche verificarsi una situazione di **starvation** nel caso in cui si scelgono i processi da risvegliare tramite un algoritmo **LIFO**, portanto alla possibilità che gli ultimi processi non vengano mai scelti.

## Problemi tipici di sincronizzazione
### Produttore/consumatore con semafori e buffer limitato

``` c#
int n;                 // Quantità della memoria disponibile
semaphore mutex = 1;   // Semafono mutex che mi permette la mutua esclusione degli accessi al buffer
semaphore empty = n;   // Semaforo empty inizializzato a n, mi dice quante posizioni libere ho
semaphore full = 0;    // semaforo full inizializzato a 0, mi dice quante posizione occupate ho

// Produttore

do {
  /* Produce elemento in next_produced */

  wait(empty);
  wait(mutex);

  /* Inserisce next_produced nel buffer */

  signal(mutex);
  signal(full);
 } while(true);

// Consumatore

do {
  wait(full);
  wait(mutex);

  /* rimuove elemento dal buffer e lo inserisce in next_consumed */

  signal(mutex);
  signal(empty);

  /* consume elemento next_consumed */
 } while(true);
```

### Problema dei lettori/scrittori

Ponendo il caso di avere una base di dati in condivisione su diversi processi, possiamo differenziare due tipi di processi: quelli che necessitano solo di leggere informazioni e quelli che necessitano di leggerle ma anche di modificarle. Questi processi sono rispettivamente chiamati processi **lettori** e processi **scrittori**.

Il *problema dei lettori/scrittori* si pone quando uno scrittore ha accesso al database e un altro processo (sia lettore che scrittore, non fa differenza) vuole accedere a sua volta alle informazioni.

Ve ne sono diverse varianti:

1. Nella prima variante nessun lettore attende, se uno scrittore fa richiesta di accedere alla base deve aspettare che tutti i lettori finiscano e che non ce ne siano altri che richiedano di ottenere l'accesso.
2. Nella seconda variante se uno scrittore attende l'accesso al database, nessun altro lettore può iniziare la lettura.

Da subito si nota che queste due varianti portano alla *starvation* gli scrittori nella *prima* e i lettori nella *seconda*. Per evitare questo problema implementiamo la lettura e la scrittura con dei semafori:

``` c#
semaphore rw_mutex = 1;   // Semaforo comune a entrambi i tipi di processi (lettori e scrittori)
semaphore mutex = 1;      // Semaforo che mi vieta la scrittura se qualche altro processo sta scrivendo su read_count
int read_count = 0;       // Numero di processi che leggono

// Scrittore

do {
  wait(rw_mutex);       // Aspetto che nessun altro processo stia scrivendo

  /* Scrivo */

  signal(rw_mutex);     // Incremento rw_mutex per segnalare che ho finito la scrittura
 } while(true);

// Lettore

do {
  wait(mutex);          // Aspetto che nessun processo stia scrivendo su read_count per poi incrementarlo
  read_count++;
  if (read_cont == 1)   // Se sono il primo processo che sta leggendo (o l'unico) 
    wait(rw_mutex);     // blocco le scritture
  signal(mutex);        // Incremento mutex perché ho aggiornato read_count

  /* Leggo */

  wait(mutex);          // Sto riaggiornando read_count quindi entro in sezione critica
  read_count--;
  (read_count == 0)
    signal(rw_mutex);   // Se sono l'ultimo processo allora risblocco le scritture
  signal(mutex);
 } while(true);
```

In questo protocollo può comunque risultare *starvation* poiché se read_count è sempre maggiore di 1 gli scrittori non verrano mai riabilitati. Questo protocollo da quindi precedenza ai lettori.

### Problema dei cinque filosofi

Si considerino cinque filosifi che condividono un tavolo rotondo con cinque sedie. La tavola è apparecchiata con cinque bacchette (ogni filosofo però ne usa due). Quando viene fame a un filosofo esso tenta di prendere le bacchette più vicine alla sua destra e alla sua sinistra. Un filosofo può prendere una bacchetta alla volta e non può prendere una bacchetta che sta già utilizzando un altro filosofo. Una volta terminato il pasto riposa le bacchette e torna a pensare.

Il *problema dei cinque filosofi* rappresenta i problemi caratterizzati dalla necessità di dare varie risorse a diversi processi evitando situazioni di stallo.

Una semplice soluzione è quella di rappresentare ogni bacchetta come un semaforo in questo modo:

``` c#
semaphore chopstick[5];

do {
  wait(chopstick[i]);           // Aspetto che la bacchetta di destra sia disponibile
  wait(chopstick[i + 1 % 5]);   // Aspetto che quella di sinistra si disponibile

  /* Mangia */

  signal(chopstick[i]);         // Poso la bacchetta di destra
  signal(chopstick[i + 1 % 5]); // Poso la bacchetta di sinistra
 } while(true);
```

Con questa soluzione posso comunque incappare in una situazione di **stallo** poiché se tutti i filosofi prendono la bacchetta di destra contemporaneamente nessun altro riuscirà a prendere quella di sinistra.

Per risolvere questo **deadlock** vengono proposte due soluzioni: quella con il filosofo *timido* e quella con il filosofo *mancino*.

**Filosofo timido**

``` c#
do {
  wait(chopstick[i]);
  if(chopstick[i + 1 % 5] == 1)
    signal(chopstick[i])
  wait(chopstick[i + 1 % 5]);
}
```

Il filosofo timido prende la bacchetta di destra, se la bacchetta di sinistra è occupata allora rilascia anche quella di destra. Facendo così risolfo il deadlock ma vado comunque in contro a *starvation* poiché è possibile che questo filosofo non prenda mai tutte e due le bacchette (e more de fame).

**Filosofo mancino**

``` c#
do {
  wait(chopstick[i + 1 % 5]);
  wait(chopstick[i]);

  /* Mangia */
}
```

Il filosofo mancino prende prima la bacchetta di sinistra e poi quella di destra, facendo così risolvo sia la *starvation* del filosofo timido che il *deadlock* della prima soluzione.

## Monitor (sto capitolo è più lungo della trilogia del signore degli anelli versione extended + lo hobbit)

I semafori sono soggetti a molteplici errori di programmazione che sono, inoltre, difficili da vedere poiché si verificano solo in condizioni particolari (che possono anche essere rare). Per ovviare a questo problema si è invatato un costrutto chiamato **monitor**.

Il **monitor** è un *tipo di dato astratto* che incapsula i dati e mette a disposizione funzioni che lavorano su di esse. La peculiarità di questo costrutto è che tutte le funzioni lavorano in mutua esclusione che quindi non deve essere implementata dal programmatore (evitando errori di programmazione). 

Oltre ai monitor, per lavorare bene sui problemi di sincronizzazione sono state create anche le variabili *condition* che permettono ai proprio oggetti di chiamare solo le funzioni `wait()` e `signal()`.

**Problema lettori/scrittori con i monitor**

``` c#
var_cond full, empty;
int cont = 0;

insert(elem) { // Metodo del monitor implementato in automatico in mutua esclusione
  if (cont == n)
    wait(full);

  add(elem);
  cont++;

  if (cont == 1)
    signal(empty);
}

estrai(elem) { // Metodo del monitor implementato in automatico
  if (cont == 0)
    wait(empty);

  elem = remove();
  cont--;

  if (cont == --n)
    signal(full);
}

```

