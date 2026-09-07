# Raspberry Pi Digital Mixer Controller

**Author: Fabio Crivellaro**

Software di controllo e interfaccia grafica realizzato in C++ per utilizzare un Raspberry Pi come interfaccia alternativa ai controlli fisici di un vecchio mixer digitale.

Il progetto è nato per sostituire l'interfaccia di controllo originale di un'apparecchiatura i cui comandi fisici non erano più funzionanti, realizzando una nuova interfaccia software collegata direttamente ai GPIO del Raspberry Pi.

## Caratteristiche

* Interfaccia grafica realizzata con **FLTK**
* 27 funzioni di controllo del mixer
* 52 posizioni di canale/banco
* Memorizzazione dello stato dei controlli per ogni posizione
* Selezione banco/canale tramite GPIO
* Generazione diretta dei segnali **DATA/CLOCK** tramite GPIO
* Salvataggio e caricamento delle configurazioni in file `.mix`
* Gestione concorrente dell'interfaccia grafica e della generazione dei segnali hardware
* Widget FLTK personalizzati realizzati in C++

## Architettura

Il programma è organizzato attorno a due elementi principali dell'interfaccia grafica e alla gestione dei GPIO:

```text
                       Raspberry Pi 400
                              │
                 ┌────────────┴────────────┐
                 │                         │
             FLTK GUI                Stato mixer
                 │                         │
          ┌──────┴──────┐                  │
          │             │                  │
    BottoneMixer   BottoneBanco            │
     27 funzioni    52 posizioni           │
          │             │                  │
          └──────┬──────┘                  │
                 │                         │
                 └──────────┬──────────────┘
                            │
                            ▼
                       GPIO Raspberry
                            │
                 ┌──────────┴──────────┐
                 │                     │
          Selezione banco/canale    DATA / CLOCK
                 │                     │
                 └──────────┬──────────┘
                            ▼
                    Elettronica mixer
```

## Interfaccia grafica

### `BottoneMixer`

`BottoneMixer` è un widget personalizzato derivato da `Fl_Box`.

Ogni istanza rappresenta una delle funzioni del mixer e mantiene un proprio stato ON/OFF.

Lo stato viene rappresentato graficamente:

* **verde** → funzione attiva
* **rosso** → funzione disattiva

Il widget implementa il comportamento di toggle direttamente attraverso la gestione degli eventi FLTK.

### `BottoneBanco`

`BottoneBanco` è un widget derivato da `Fl_Button` utilizzato per selezionare una delle 52 posizioni disponibili.

La posizione selezionata viene evidenziata graficamente e viene utilizzata per determinare i valori dei segnali GPIO relativi a banco e canale.

## Gestione dello stato

Ogni posizione di canale/banco dispone di un proprio insieme di 27 stati.

Concettualmente:

```text
52 posizioni
    │
    ├── posizione 1  → 27 stati
    ├── posizione 2  → 27 stati
    ├── posizione 3  → 27 stati
    │       ...
    └── posizione 52 → 27 stati
```

Quando viene selezionata una nuova posizione:

1. lo stato corrente dei 27 controlli viene memorizzato;
2. viene recuperato lo stato associato alla nuova posizione;
3. i controlli della GUI vengono aggiornati;
4. vengono aggiornati i segnali GPIO relativi a banco e canale.

In questo modo l'interfaccia grafica rappresenta sempre lo stato della posizione attualmente selezionata.

## Interfaccia GPIO

Il Raspberry Pi comunica con l'elettronica del mixer direttamente attraverso i propri GPIO.

Non viene utilizzato un bus di comunicazione standard: il software genera direttamente i livelli logici necessari sui pin del Raspberry Pi.

### Selezione banco

Tre GPIO vengono utilizzati per rappresentare il banco selezionato tramite tre bit:

```text
BANK BIT 1
BANK BIT 2
BANK BIT 3
```

### Selezione canale

Altri tre GPIO rappresentano il canale tramite tre bit:

```text
CHANNEL BIT 1
CHANNEL BIT 2
CHANNEL BIT 3
```

### DATA / CLOCK

La configurazione delle 27 funzioni del mixer viene rappresentata internamente come un word di stato.

Il programma genera quindi direttamente sui GPIO i segnali:

```text
DATA
CLOCK
```

I due segnali vengono prodotti dal software tramite `digitalWrite()` e inviati direttamente all'elettronica di controllo.

## Test hardware

Lo sviluppo è stato effettuato su **Raspberry Pi 400**, utilizzando direttamente i GPIO.

Prima del collegamento all'apparecchiatura reale, il funzionamento è stato verificato tramite un piccolo **testbench elettronico** costruito appositamente per simulare l'elettronica di ingresso.

Il testbench utilizzava due circuiti integrati e LED per visualizzare i segnali generati dal Raspberry Pi.

Questo ha permesso di verificare direttamente a livello hardware:

* selezione di banco;
* selezione di canale;
* generazione del segnale DATA;
* generazione del segnale CLOCK;
* variazione dei dati in funzione dei controlli selezionati nella GUI.

Dopo la verifica sul testbench, l'interfaccia è stata utilizzata per il controllo dell'apparecchiatura reale.

## Salvataggio delle configurazioni

Le configurazioni possono essere salvate e caricate tramite file con estensione `.mix`.

Il formato memorizza lo stato delle 27 funzioni per ciascuna delle 52 posizioni.

Struttura generale:

```text
[mixer data]
000101...
101100...
...
```

La prima riga identifica il tipo di file, mentre le righe successive rappresentano gli stati dei 27 controlli associati alle diverse posizioni.

Il programma effettua inoltre controlli sulla struttura del file durante il caricamento, verificando estensione, intestazione, numero di righe e valori contenuti nei dati.

## Tecnologie

* **C++**
* **FLTK** — interfaccia grafica
* **wiringPi** — accesso ai GPIO
* **Raspberry Pi 400**
* Thread C++ per la generazione continua dei segnali hardware
* File I/O per il salvataggio delle configurazioni
* GPIO digitali per l'interfacciamento con l'elettronica

## Struttura del progetto

```text
.
├── main.cpp
├── bottone_mixer.cpp
├── bottone_mixer.h
├── bottone_banco.cpp
├── bottone_banco.h
└── ...
```

Il progetto originale è stato sviluppato utilizzando **Code::Blocks**, che gestiva la configurazione del progetto e la compilazione.

## Contesto del progetto

Questo progetto non è stato sviluppato come semplice esercizio di programmazione GUI.

L'obiettivo era realizzare una soluzione concreta per continuare a utilizzare un'apparecchiatura esistente nonostante il malfunzionamento dei suoi controlli originali.

Il lavoro ha quindi coinvolto sia lo sviluppo software sia l'interfacciamento diretto con l'elettronica:

**GUI → gestione dello stato → GPIO → segnali logici → elettronica del mixer**

La validazione è stata effettuata inizialmente tramite un testbench elettronico dedicato e successivamente sull'apparecchiatura reale.

## Autore

**Fabio Crivellaro**

Progetto sviluppato in C++ su Raspberry Pi 400.
