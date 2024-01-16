Reference:
[[L08.1 - Progetti complessi.pdf]]
[[L08.2 - CMake.pdf]]
[[L08.3-Git_introduzione.pdf]]
## Progetti su file multipli
Meglio organizzare il software in più file:
- Organizzazione migliore
- Migliore leggibilità
- Maggiore semplicità organizzativa se più sviluppatori lavorano allo stesso progetto

I *file header* contengono:
- Definizioni di classe
- Dichiarazioni di funzione
- Inclusi ( #include ) ma non compilati direttamente

Il *file sorgente* contiene:
- Definizioni di funzioni e classi
- Il main (uno solo in tutto il progetto, in un solo file sorgente)
- Compilati dal compilatore, ma non inclusi 

Queste due modalità si usano perché:
- Header: inclusi perché dichiarazioni di funzioni e definizioni di classi sono funzionali alla scrittura dei file .cpp 
- I sorgenti .cpp non hanno questa funzione, non devono essere inclusi 
- Le translation unit (file oggetto) sono linkati assieme

### Header multipli
Un sorgente può includere più header

```c++
#include <iostream> 
#include "date.h" 
#include "year.h"
```

Ogni header deve essere protetto dalle *include guard*
```c++
#ifndef RATIONAL_H 
#define RATIONAL_H // ... #

endif // RATIONAL_H
```
Rational_H è la costante che usiamo per proteggere il rational.h

### Errori di linking
Accade quando ci sono tanti file. Molto importante distinguere:
- <span style="color:#ff0000">Errori di compilazione</span>: sintassi, conversione di tipo, ...
- <span style="color:#ff0000">Errori di linking</span>: parti di software mancanti
```c++
double area(double w, double h); 
int main() 
{ 
	int w = 2, h = 4; 
	int a = area(w, h); 
	return 0; 
}
```

Rivedi ste maledette direttive sul coding style: https://google.github.io/styleguide/cppguide.html

---
## Introduzione a CMake
Perché usiamo Cmake? A cosa serve?
è un tool molto utile (da usare per progetto finale)
### Descrizione del progetto
Un progetto software è composto da molti file sorgente, ma progetti complessi ci possono essere tanti eseguibili e tante librerie.
Progetti così grandi **devono** essere suddivisi in progetti più piccoli

I soli sorgenti non bastano. Serve sapere come deve essere organizzata la compilazione. 
Capire in fase di compilazione se:
- Creare un eseguibile
- Creare una libreria
In entrambi i casi:
-  Lista dei .cpp che devono essere compilati assieme
- Lista delle librerie che devono essere linkate

Descrizione tramite comando di compilazione non è il massimo, è poco pratico ed esegue compilazione completa.

Descrizione tramite IDE: dipende dall'IDE in uso, poco generale.

Sarebbe utile avere sistema di building *standardizzato*.
### Sistemi di building
Esistono vari sistemi di building 
- Es: Make, Ninja, ... 
- Spesso dipendenti dalla piattaforma in cui lavorano

### CMake
- Meta-sistema di building standardizzato
- Indipendente da:
	- Compilatore 
	- Piattaforma
- Usato in abbinamento al sistema di building nativo
- Utile per distribuire il codice sorgente con indicazioni di building, precompilazione
- Supportato da molti IDE

Cmake è un sistema alternativo per descrivere un progetto. Definisce:
- Target di compilazione ed elenco sorgenti 
- Eseguibile 
- Libreria 
- Librerie da linkare
Gestito tramite file di testo: CMakeLists.txt
![[processo cmake.png.png]]

## Processo lato utente
Per esempio se vogliamo prendere un progetto opensource.
Per compilare un progetto con Cmake abbiamo due tasti:
- Configure
- Generate

Scaricare un sorgente gestito con CMake permette di avere: 
- I file sorgenti del progetto 
- La descrizione del progetto 
- Un sistema che compila il progetto in autonomia 
- Un sistema per gestire la compilazione dei moduli 
- Parti del progetto possono essere compilate o meno a seconda delle impostazioni
Ad esempio per un progetto possiamo decidere quali moduli compilare oppure no, alleggerendo la compilazione.

Tipica gestione di un progetto CMake:
- Lanciare CMake (es., GUI) 
- Selezionare la directory contenente il sorgente 
- Selezionare la directory che conterrà i prodotti della compilazione 
- Standard: [root sorgente]/build 
- Configure: legge e scrive configurazione standard
- Eventuale selezione/deselezione dei moduli 
	- Se ci sono modifiche: configure 
- Generate

- Generate crea i file per il sistema di building in uso 
- Dopo generate si chiude CMake e si compila con il normale flusso di lavoro

## Processo lato sviluppatore
Creo una descrizione del progetto nello standard Cmake
- Scrivo a mano i CmakeLists.txt
- Uso IDE per generarli automaticamente
	- Più complicati
	- In genere, non devono essere modificati
		- Possono essere sovrascritti in automatico
Capiamo come si scrive a manina:

1. Setto versione minima di Cmake
```c++
cmake_minimum_required(VERSION 3.2 FATAL_ERROR)
```
2. Nome al progetto
```c++
project(<name> VERSION <version> LANGUAGES CXX)
```
3. Seleziona librerie con dipendenze
```c++
find_package(Qt5 REQUIRED COMPONENTS Widgets)
```
4. Target per la creazione di un eseguibile
```c++
add_executable(tool
	main.cpp
	another_file.cpp		  
	)
// qui gli header ci possono stare, ed è corretto
```
5. Target per la creazione di una libreria 
```c++
add_library(foo STATIC
	foo1.cpp	
	foo2.cpp
	)
```

#### Esempio completo

```ruby
ltonin@ltonin-laptop:~/Rational/ 
├── bin 
├── build 
├── CMakeLists.txt 
├── include 
│ 
	└── rational.h 
├── README.txt 
└── src 
	├── main.cpp 
	└── rational.cpp
```

```scss
cmake_minimum_required(VERSION 2.84 FATAL_ERROR) 
set(CMAKE_CXX_STANDARD 11) 
project(rational) 

include_directories(include) 

add_library(${PROJECT_NAME} SHARED 
	src/rational.cpp) 

add_executable(main bin/main.cpp) 

target_link_libraries(main ${PROJECT_NAME})
```

```ruby
ltonin@ltonin-laptop:~/Rational/$ mkdir build 
ltonin@ltonin-laptop:~/Rational/$ cd build 
ltonin@ltonin-laptop:~/Rational/$ cmake .. 
ltonin@ltonin-laptop:~/Rational/$ make
```
