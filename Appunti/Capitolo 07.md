# Capitolo 7
# Situazioni di stallo

### Condizioni necessarie

Si può avere, in un sistema, una situazione di stallo solo se si verificano contemporaneamente queste quattro condizioni:

* **Mutua esclusione:** almeno una risorsa deve essere non condivisibile e quindi utilizzabile da un solo processo per volta.
* **Possesso e attesa:** un processo deve essere in possesso di una risorsa e, simultaneamente, essere in richiesta per un'altra.
* **Assenza di prelazione:** si presume che un processo non possa essere interrotto e quindi debba rilasciare la risorsa volontariamente.
* **Attesa circolare:** dato un insieme [P1,---,Pn] di processi, ogni processo deve attendere una risorsa che è in uso dal processo successivo a lui.

### Grafo di assegnazione delle risorse

Le situazioni di stallo possono essere facilmente descritte avvalendosi del **grafo di assegnazione delle risorse**. 

Vi è un insieme [P1,---,Pn] e un altro [R1,---,Rn] che indicano rispettivamente l'insieme dei processi e quello delle risorse.

* Pi -> Ri è un **grafo di richiesta**.
* Ri -> Pi è un **grafo di assegnazione**.

Si può dimostrare che se nel grafo non è presente alcun ciclo allora siamo sicuri che non è presente alcuno stallo, viceversa se c'è un ciclo **potrebbe** esserci uno stallo. Quest'ultima affermazione dipende da quante istanze ha una risorsa.

![alt text](http://i.imgur.com/iRbyWjd.png "Grafo circolare con e senza stallo")


## Metodi per la gestione degli stalli

Questo problema può essere risolto tramite tre approcci:

* Posso usare un protocollo che previene il verificarsi di almeno una delle condizioni sopra elencate. (**Prevenire**)
* Non mi preoccupo di avere stalli bensì controllo se ce ne sono stati e se ce ne sono stati impiego un algoritmio per recuperare il sistema. (**Evitare** e **Rilevare**)
* Me ne fotto degli stalli e non faccio niente. (**Don't care**)

Quest'ultimo approccio è quello più usato nei sistemi operativi moderni (Linux, Windows) in quanto il verificarsi degli stalli è una cosa molto rara quindi per motivi economici e di efficienza non impiego alcun metodo per la risoluzione di essi.

Per attuare il secondo approccio c'è bisogno che il sistema conosca in anticipo tutte le informazioni riguardanti risorse e processi in modo da accorgersi di eventuali stalli.

## Prevenire le situazioni di stallo

Descrivo dei protocolli che possono evitare una delle condizioni di stallo

### Mutua esclusione

Non possono esserci procotolli che mi evitano la mutua esclusione in quanto deve esistere almeno una risorsa che non sia condivisa.

### Possesso e attesa

Ho due protocolli per evitare *possesso e attesa*:

* Impongo che un processo inizi la sua esecuzione solo quando ha tutte le risorse assegnate.
* Permetto a un processo di richiedere risorse solo quando non ne possiede.

Questi protocolli hanno due svantaggi, il primo è che può verificarsi starving nel processo in quanto è possibile che non riceva mai tutte le risorse, il secondo è un problema di efficienza un quanto vengono assegnate risorse che, magari, non saranno usate subito ma dopo molto tempo.

### Assenza di prelazione

Anche qui ho due protocolli:

* Se un processo che possiede delle risorse ne richiede altre ma queste altre sono utilizzate da altri processi allora faccio la prelazione su tutte le risorse del processo richiedente e gli permetto di avviarsi solo quando potrà accedere sia alle vecchie che alle nuove risorse.
* Protocollo simile al primo ma quando viene fatta la richiesta per le risorse si verifica se sono disponibili, se non lo sono si controlla se sono assegnate a un processo che è in attesa di altre risorse, in questo caso si tolgono da quel processo e si assegnano al primo.

Questi protocolli sono utilizzati spesso per risorse che permettono il salvataggio del loro stato.

### Attesa circolare

Devo imporre che un processo richieda le risorse in maniera crescente.

## Evitare le situazioni di stallo

Un metodo alternativo per evitare la prevenzione delle situazioni di stallo è quello di evitarle e per fare ciò si richiedono più informazione da parte dei processi quando entrano nel sistema. In particolare si richiedono il *numero massimo* delle risorse di ciascun tipo necessarie al processo.

### Stato sicuro

>Quanto un sistema è in *stato sicuro* significa che non potrà mai entrare in stallo, se è invece in *stato non sicuro* c'è la possibilità che possa entrare in stallo (non la sicurezza).

Un sistema è in *stato sicuro* quando è presente una *sequenza sicura* di processi. Considerati [P1,..,Pn] processi, si ha una *sequenza sicura* se per ogni Pi le richieste che può ancora fare si possono soddisfare solo se le risorse richieste appartengono a un Pj con j < i.

> Un sistema può passare da stato sicuro a stato non sicuro.

### Algoritmo con grafo di assegnazione delle risorse

Posso usarlo quando ho una sola istanza per ogni tipo di risorsa. I grafi sono uguali a quelli studiati precedentemente ma hanno in può un **arco di rivendicazione** Pi --> Rj che è denotato da una linea tratteggiata piuttosto che continua e significa che il processo Pi può richiedere la risorsa Rj in qualunque momento rimanendo in stato sicuro.

### Algoritmo del banchiere

Posso usarlo quando ho più istanze per ogni tipo di risorsa.

Quando un nuovo processo entra nel sistema deve dichiarare il numero e il tipo di risorse di cui potrà avere bisogno (non devono essere maggiori delle risorse possedute dal sistema). Le risorse vengono assegnate a un processo che le richiede solo se c'è la sicurezza che dopo l'assegnazione il sistema rimanga in uno stato sicuro se no si fa aspettare il processo.

Per implementare questo algoritmo abbiamo bisogno di qualche struttura dati, considerato con *n* il numero di processi e con *m* il numero dei tipi di risorsa:

* **Disponibili:** Un vettore lungo *m* che indica il numero di istanze per ogni tipo di risorsa.
* **Massimo:** Una matrice *n* x *m* che definisce la richiesta massima di ciascun processo. Massimo[i,j] = k significa che un processo Pi può richiedere k istanze del tipo di risorsa Rj.
* **Assegnate:** Una matrice *n* x *m* che definisce il numero di istanze di ciascun tipo di risorsa assegnate a ogni processo. Assegnate[i,j] = k significa che al processo Pi sono assegnate k istanze del tipo di risorsa Rj
* **Necessità:** Una matrice *n* x *m* che indica la necessità residua di risorse relativa a ogni processo. Necessità[i,j] = k significa che il processo Pi può necessitare di altre k istanze del tipo di risorsa Rj per terminare l'esecuzione. Necessità[i,j] = Massimo[i,j] - Assegnate[i,j]


