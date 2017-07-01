# Capitolo 9

# Memoria virtuale

## Introduzione

La memoria virtuale consente una diversa percezione della memoria fisica da parte dell'utente, permette inoltre di eseguire programmi che si trovano solo parzialmente nella memoria centrale dando così diversi benefici:

* Un programma non è più involato alla memoria fisica disponibile.
* Poichè ogni programma occupa meno memoria possono essere caricati più programmi aumentanto la concorrenza e l'utilizzo della CPU.
* Poichè i programmi non vengono caricati del tutto servono meno operazioni di I/O quindi i programmi sono eseguiti più velocemente.

Ad ogni processo viene assegnata una memoria virtuale dove verrano contenuti dati, stack, heap e il codice. Tra l'heap e lo stack è lasciato deliberatamente dello spazio vuoto per permettere ad entrambi di espandersi (heap quando c'è memoria allocata dinamicamente e stack quando ci sono diverse chiamate di funzione).

La memoria virtuale permette inoltre di poter condividere dati e memoria di diversi processi tramite la condivisione delle pagine.

## Paginazione su richiesta

Se ho un programma che mi mette davanti delle opzioni su come proseguire, è inutile che carico in memoria tutte le opzioni quando in realtà ne utilizzerò solo una. C'è una tecnica che mi permette di caricare solo le pagine del processo che servono, è chiamata **paginazione su richiesta**. 

In questo caso caricherò solo le pagine necessarie per l'esecuzione del programma più l'opzione scelta dall'utente in modo da risparmiare molta memoria.

### Concetti fondamentali

Per poter mettere in atto questa tecnica ho bisogno di un bit di validità in ciascuna pagina che mi dica se è *valida* o *non valida*. Se è valida significa che la pagina è caricata in memoria centrale e può essere eseguita, se non è valida può significare due cose: o che la pagina non appartiene allo spazio di indirizzi di quel programma o che è caricata in memoria secondaria e deve essere caricata in quella centrale per essere eseguita.

Se un programma ha tutte le pagine necessarie per l'esecuzione caricate già in memoria centrale tutto fila liscio, se invece ne manca qualcuna si va incontro al **page fault**. Nel caso di **page fault** il problema è risolto abbastanza (sono 11 punti :cry: ) facilmente come nei seguenti passi:

1. trap per il sistema operativo.
2. salvataggio dei registri utente e dello stato del processo.
3. verifica che l'interruzione sia dovuta o meno a un page fault.
4. ricerca della pagina mancante nella memoria secondaria.
5. lettura dalla memoria secondaria e trasferimento della pagine in un frame libero.
6. durante il quinto passaggio la CPU fa altre robe con altri processi invece che stare ferma.
7. quando l'I/O del quindi passagio finisce viene mandata un interrupt alla CPU.
8. salvataggio dei registri utente e dello stato del processo che è partito nel sesto punto.
9. aggiornamento della tabella delle pagine con riferimento alla nuova pagina.
10. attesa che la CPU sia di nuovo assegnata a quel processo.
11. context switch del processo e ri-esecuzione di esso dal momento in cui si era fermato, stavolta con la pagina che gli serviva.

La paginazione su richiesta può anche essere **pura** ovvero non viene caricata alcuna pagina del processo in memoria e si va di page fault finché il processo non ha tutto ciò che gli serve.

La memoria secondaria dove le pagine non caricate risiedono si tratta, di solito, di una memoria veloce chiamato **dispositivo di swap**, la sezione del disco usata a questo scopo si chiama **area di swap**.

## Copiatura su scrittura

Utilizzando la `fork()` il processo padre viene duplicato in un processo figlio dove entrambi i processi condivisono le stesse pagine. Per evitare problemi quando uno dei due processi deve modificare una pagina, ne viene creata una copia che sarà puntata dal processo che deve modificare. In questo modo l'altro processo punterà alla pagina originale e non modificata.

Quando devo creare la copia devo cercare un frame libero dove inserirla, a questo scopo i sistemi hanno una *pool* di frame liberi che sono inizializzati tramite la tecnica **azzeramento su richiesta** che riempie i frame liberi di zeri in modo da cancellare qualunque cosa ci fosse prima.

## Sostituzione delle pagine

Se un processo mi da **page fault** ma non ho memoria libera per inserire la pagina richiesta sono in un caso di **sovrallocazione**, come si comporta quindi il sistema operativo?

* Posso terminare il processo che ha generato page fault ma così non renderei più trasparente all'utente il fatto che il sistema stia usando la paginazione della memoria. Non è quindi una scelta adatta.
* Posso scaricare dalla memoria un processo in modo da lasciare i frame che utilizzava liberi e diminuire il grado di multiprogrammazione.
* Posso utilizzare il metodo della **sostituzione delle pagine**.

### Sostituzione pagina

Quando un processo mi da page fault e sono in sovrallocazione scelgo un **frame vittima** (un frame è dove inserisco una pagina a livello fisico) la cui pagina verrà sostituita con la pagina necessaria a terminare il page fault. 

Per attuare questa procedura ho bisogno però di **due** accessi alla memoria, uno per salvare lo stato della pagina vittima e uno per inserire la pagina nuova. Posso evitare questi due accessi utilizzando un **bit di modifica** associato ad ogni pagina e che indica se la pagina è stata modificata o meno. Avendo questo bit decido di sceglere come vittima solo processi che hanno questo bit di modifica impostato a 0 in quanto non sono stati modificati e non c'è bisogno di salvarli in memoria visto che ci sono già.

Per realizzare questo metodo ho bisogno di un **algoritmo di sostituzione dei frame**.

### Sostituzione FIFO delle pagine

Ad ogni pagina associo il timestamp di quando è stata caricata in memoria, scelgo quindi di sostituire le pagine che sono da più tempo in memoria. Oltre ad assegnare un timestamp posso anche creare una coda FIFO e in questo caso prendere il primo elemento della coda come **vittima**.

È normale pensare che più frame ho in memoria meno page fault possono avvenire in quanto ho più spazio per inserire pagine, tuttavia questo algoritmo soffre di un problema che va contro quest'ultima affermazione.

Questo problema è detto **anomalia di Belady**. Difatti se ho più frame ho più page fault.

### Sostituzione ottimale delle pagine (OPT)

Per ovviare all'anomalia di Belady sono stati creati altri algoritmi che non ne soffrono, questo è il migliore e consiste nel scegliere come vittima la pagina che non verrà usata per il periodo di tempo più lungo.

È chiamato algoritmo ottimale proprio perché è l'algoritmo che garantisce meno page fault di tutti gli altri algoritmi, purtroppo però è di difficile implementazione in quanto non si conosce la frequenza con la quale una pagina sarà usata, viene quindi utilizzato solo per stuti comparativi :unamused: .

### Sostituzione LRU (Last-recently-used)

Questo algoritmo sostituisce la pagina che non è stata usata per più tempo, è comunque di difficile implementazione ma abbiamo diversi metodi per implementarlo:

* **Contatori:** A ogni elemento della tabella delle pagine si associa un campo *momento di utilizzo* e alla CPU si aggiunge un contatore che si incrementa ogni volta che ci si riferisce a quella pagina. Ogni volta che mi riferisco ad una pagina il contatore viene aumentato e il suo valore viene inserito nel campo *momento di utilizzo* di quella pagina, facendo così riesco facilmente a capire qual è la pagina che non è stata usata per il tempo più lungo. Tuttavia ho bisogno di ricercare tra le pagine quella giusta e una scrittura in memoria per aggiornare *momento di utilizzo* quindi non è un'implementazinoe ottimale.
* **Stack:** Un altro metodo prevere l'implementazione di uno stack che contiene i numeri delle pagine e ogni volta che faccio riferimento a una di quelle pagine essa viene spostata in cima allo stack, facendo così ho la pagina meno usata sempre alla fine dello stack.

Né LRU né OPT soffrono dell'anomalia di Belady e appartengono entrambi alla famiglia degli **algoritmi a stack** ovvero un algoritmo per il quale è possibile mostrare che l'insieme delle pagine in memoria per *n* frame è sempre un *sottoinsieme* dell'insieme delle pagine che dovrebbero essere in memoria per *n* + 1 frame.
