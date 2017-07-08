# Capitolo 4

## Thread

Un thread è la base d'uso della CPU, contiene un identificatore di thread (ID), un contatore di programma, un insieme di registri e uno stack. Condivide con gli altri thread dello stesso processo tutto lo spazio d'indirizzi (dati e risorse di sistema). Un processo multithread è in grado di svolgere più compiti in maniera concorrente.

### Vantaggi
 
1. **Tempi di risposta:** Rendere un processo multithread mi permette di continuare la sua esecuzione anche se una parte di esso è in attesa di un azione dell'utente o sta facendo un'operazione particolarmente lunga. Utile per le interfacce utente.
2. **Condivisione delle risorse:** I thread per default condividono la memoria del processo a cui appartengono, è utile perché si possono avere molti thread di attività diverse ma tutte nello stesso spazio d'indirizzi.
3. **Economia:** C'è meno overhead nel creare thread e gestirne i cambi di contesto che farlo direttamente con i processi.
4. **Scalabilità:** Perché la programmazione multithreading si adatta ancora meglio nei sistemi multiprocessore rendendo ancora più concorrente l'esecizione dei processi.

### Tipi di thread

Distinguiamo due tipi di thread: a livello *utente* e a livello *kernel*. I primi gestiti sopra il livello del kernel e i secondi gestiti direttamente dal sistema operativo. Deve però esserci una relazione tra i thread a livello utente e kernel, ne abbiamo tre tipologie: **molti a uno**, **uno a uno** e **molti a molti**.

#### Molti a Uno

Fa corrispondere molti thread a livello utente a un solo thread a livello kernel. È efficiente perché i thread sono gestiti da librerie utente. Tuttavia il processo rimane bloccato se un solo thread fa una chiamata di tipo bloccante. Inoltre, essendo associato solo un thread kernel nei sistemi a multicore si annulla il parallelismo.

#### Uno a Uno

Mette in orrispondenza un thread a livello utente a un thread a livello kernel. Offre un grado di concorrenza maggiore perché se un thread fa una chiamata bloccante basta crearne un altro. È supportata anche nei sistemi multiprocessore. L'unico difetto è che la creazione di thread a livello kernel può essere pesante per l'applicazione quindi se ne deve limitare il numero.

#### Molti a Molti

Mette in corrispondenza più thread a livello utente con più thread a livello kernel. Questo modello non ha alcuno dei difetti sopra elencati per gli alrtri modelli. I programmatori possono creare tutti i thread che ritengono necessari.

### Differenza tra thread e processi

I processi sono, di solito, indipendenti tra loro. I thread di un processo invece condividono tutto lo spazio di indirizzi del processo a cui sono associati. Inoltre la creazione di un thread è incredibilmente meno onerosa della creazione di un nuovo processo.
