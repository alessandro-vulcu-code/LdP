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
