# Monopoly 

Questo progetto è una simulazione del classico gioco da tavolo Monopoly, con qualche semplificazione. È scritto in C++ e offre due modalità di gioco: "computer_only" e "human_and_computer".

# Funzionalità

- **Modalità di gioco "computer_only"**: In questa modalità, quattro bot giocano l'uno contro l'altro in una partita di Monopoly. Ogni bot prende decisioni in modo autonomo, permettendo di simulare una partita completa di Monopoly senza intervento umano.

- **Modalità di gioco "human_and_computer"**: In questa modalità, un giocatore umano compete contro tre bot. Questa modalità offre un'interfaccia interattiva che permette al giocatore umano di prendere decisioni durante il gioco.

- **Registro di gioco**: Tutte le azioni di gioco vengono registrate in un file di log, permettendo di analizzare il gioco dopo la sua conclusione.

# Classi principali

## 🗺️Classe Playground
La classe `Playground` rappresenta il tabellone del gioco del Monopoly. Gestisce le caselle del gioco, che possono essere di vari tipi, come "Economy", "Standard", "Luxury" o "Corner".

La classe `Playground` utilizza un generatore di numeri casuali per determinare il tipo di ciascuna casella durante l'inizializzazione del tabellone del gioco.

## 📦Classe Box
La classe `Box` rappresenta una casella sul tabellone del gioco del Monopoly. Ogni casella ha un tipo, uno stato, un proprietario, una posizione e una serie di prezzi associati.

## 🕹️Classe Player
La classe `Player` rappresenta un giocatore nel gioco del Monopoly. Ogni giocatore ha un indice univoco, un saldo di fiorini (la valuta del gioco), e un flag che indica se il giocatore è stato eliminato o no.

## 👨🏻Classe Human
La classe `Human` estende la classe `Player` e rappresenta un giocatore umano nel gioco del Monopoly. Questa classe implementa il comportamento specifico per un giocatore umano, come prendere decisioni basate sull'input dell'utente.

## 🤖Classe Bot
La classe `Bot` estende la classe `Player` e rappresenta un bot nel gioco del Monopoly. Questa classe implementa il comportamento specifico per un bot, come prendere decisioni in modo autonomo basate su una logica predefinita.

## 📂Classe FileIO
La classe `FileIO` fornisce funzionalità per la gestione di file. Questa classe è utilizzata per aprire un file, scrivere su di esso e cancellarne il contenuto.

# Come eseguire
Per eseguire il simulatore di Monopoly, compila il codice sorgente utilizzando cmake: 

Naviga nella cartella build del progetto
```
cd build
```

Esegui cmake e make:

```
cmake ..
```

```
make
```

Infine, esegui l'eseguibile con uno dei seguenti comandi:

```
./Monopoly computer_only
```

```
./Monopoly human_and_computer
```

Il primo comando avvia una partita "computer_only", mentre il secondo avvia una partita "human_and_computer".

# Come giocare
Nel caso "computer_only", vengono creati quattro bot che giocano tra loro per un massimo di 100 turni. Se un bot esaurisce i suoi fiorini, viene eliminato. Il gioco termina quando rimane un solo bot, che viene dichiarato vincitore.

Nel caso "human_and_computer", tre bot e un giocatore umano partecipano al gioco. Quando arriva il turno del giocatore umano, il gioco richiede all'utente di eseguire le azioni. Per rendere il turno del giocatore umano più interattivo, viene richiesto di premere Enter per lanciare i dadi. Se il giocatore atterra su una casella libera o di sua proprietà, gli viene chiesto se desidera acquistare la casella o effettuare un upgrade a casa o hotel, se la casella è già di sua proprietà.

In entrambi i casi, alla fine di ogni turno, viene chiesto se:
- Mostrare lo stato della partita, ossia il tabellone, il budget e le proprietà di tutti i giocatori
- Andare al prossimo turno cliccando il tasto Enter

Tutte le azioni del gioco vengono registrate in un file di log chiamato "game_log.txt".

# Crediti
Questo progetto è stato sviluppato dagli studenti Prizzon Giorgio, Vulcu Alessandro e Zolla Gabriele, che frequentano il corso di Ingegneria Informatica nell'anno accademico 2023/2024. È stato realizzato come progetto finale per il corso di "Laboratorio di Programmazione".