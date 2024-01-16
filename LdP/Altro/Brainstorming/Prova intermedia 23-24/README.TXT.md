# Mid_Proj

## Descrizione del progetto

Il progetto "Mid_Proj" è un'applicazione di gestione di una biblioteca digitale. L'applicazione permette di creare e gestire una collezione di libri.

## Funzionalità

- Creazione di un "scaffale" di libri con una dimensione specificata.
- Aggiunta di libri a un "scaffale".
- Spostamento di libri da un "scaffale" a un altro.
- Copia di un "scaffale" in un altro.
- Creazione e modifica di libri

## Come eseguire il progetto

Il progetto va eseguito utilizzando Cmake e Make dal proprio terminale. Assicurarsi di avere installato sulla propria macchina questi due programmi, altrimenti consultare i siti ufficiali per l'installazione.
1) dal terminale, recarsi nella cartella `build` con il comando `cd /path/to` presente all'interno della cartella `Mid_proj`
2) eseguire il comando `cmake ..`, a seguire `make` 
3) eseguire il programma con il comando `./Book`

## Struttura del codice

Il progetto è composto da diversi file:

- `BookShelf.h`: Questo file contiene la definizione della classe `BookShelf`, che rappresenta un "scaffale" di libri.
- `Book.h`: Questo file contiene la definizione della classe `Book`, che rappresenta un libro.
- `Date.h`: questo file contiene una semplice implementazione della classe Date
- `BookShelf.cpp, Book.cpp, Date.cpp`: questi file contengono le implementazioni delle funzioni scritte nei rispettivi header file.
- `main.cpp`: Questo file contiene il codice per testare le funzionalità del progetto.

Gli header file e i file .cpp sono stati separati rispettivamente nelle cartelle `include` e `src` per rendere migliore l'organizzazione dei file

# Autori

Il progetto è stato sviluppato dagli studenti Vulcu Alessandro, Zolla Gabriele e Prizzon Giorgio del corso di Ingegneria Informatica, a.a. 2023-2024, per il corso di Laboratorio di Programmazione. 