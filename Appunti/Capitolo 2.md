# Capitolo 2

## Interprete dei comandi

L'interprete dei comandi (che in alcuni sistemi è integrato nel kernel) è un programma speciale che si avvia non appena un utente effettua il login. Se il sistema permette una scelta dell'interprete dei comandi esso viene definito *shell*. La funzionalità principale dell'interprete dei comandi è quello di interpretare i comandi inseriti dall'utente. Questi ultimi possono essere implementati in due modi:

* Come in MS-DOS. Lo stesso interprete comandi contiene il codice da eseguire e che invocherà le idonee chiamate di sistema. In questo caso il numero di comandi disponibili ha un impatto sulla dimensione dell'interprete.
* Come in UNIX. Ogno comando si riferisce a un programma di sistema quindi quando viene eseguito, l'interprete cercherà il programma associato e lo eseguirà con i parametri inseriti. Esempio:
  
  Il comando `rm file.txt` cerca un file chiamato rm, lo carica in memoria e lo esegue con il parametro `file.txt`.
  
## Sistemi Monolitici

Sono una massa informe di system call, per organizzarle meglio diciamo che una system call dell'utente viene eseguita tramite system call di servizio che a sua volta richiamano system call di utilità. Con questa organizzazione:

* Non c'è modularità.
* Il codice è complesso e difficile da mantenere.
* Non ha stabilità nel codice.
* Ho problemi di protezione (è facile intercettare le procedure del Sistema Operativo).
  
## Sistemi a livelli

Per organizzare la massa informe del sistema monolitico divido in livelli, per esempio:

* Livello 0: HW
* Livelli random
* Livello N-1: Utente

Ogni livello è un sottoinsieme di quello sopra. Facendo così:

* Ho modularità.
* È inserito il concetto di blackbox.
* Ho però problemi di efficienza, in quanto per arrivare da un livello all'altro devo scorrere quelli che stanno in mezzo e ciò può portare via tempo.

## Sistema modulare

Pensato tramite i principi della programmazione Object Oriented. Kernel modulare:

1. Oggetti.
2. Interfacce.
3. Collegamenti dinamici fra gli oggetti.

--- 

Regà qui fa robe che spiega meglio dopo non ci perdo manco tempo a scriverle.
