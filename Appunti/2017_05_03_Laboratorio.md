# Shell

### Caratteristiche

* Ogni sgell fornisce un linguaggio di programmazione. I programmi di shell sono denominati shell script
* Uno shell script è un file di testo che contiene comandi che sono eseguiti come se l'utente li immettesse riga per riga
* I comandi che la shell riconosce possono essere interni alla shell stessa (built-it), file eseguibili esterni o shell script
* A parità di nome i comandi interni hanno la precedenza
* Qui faremo riferimento alla *bash*

### Comandi

La sintassi tipica di un comando è:

`` > comando (opzioni) (argomenti) ``

**Esempi:**

``> date``
``> who``
``> uname -a``
``> ps``
``> ps ef``
``\_`` indica un sottoprocesso

### LS

``` bash
drwxr-xr-x   2 root root     4096 mag  3 08:51 bin
drwxr-xr-x   4 root root     4096 mag  3 08:57 boot
drwxrwxr-x   2 root root     4096 mar 15 11:47 cdrom
-rw-------   1 root root 47595520 mar 27 13:30 core
drwxr-xr-x  20 root root     4300 mag  3 08:47 dev
drwxr-xr-x 139 root root    12288 mag  3 08:57 etc
drwxr-xr-x   3 root root     4096 mar 15 11:47 home
```

Le prime lettere indicano diverse cose:

* La prima tripletta indica i permessi dell'utente, se c'è un meno prima significa che non ho permessi
* La seconda tripletta indica i permessi del gruppo dell'utente
* La terza tripletta indica i permessi di tutti gli utenti (utente generico)

**Esempio:** 

rwvr--r--: rwv = 111, r-- = 100, r--100 che tradotti in decimale indica 7, 4, 4. Difatti per assegnare questi permessi si utilizza il comando ``chmod 744`` (r = read, w = write, v = view).

``umask 002`` sottrae i permessi write alla terza tripletta (utente generico), i nuovi file avranno permessi 664 mentre le directory 775.
