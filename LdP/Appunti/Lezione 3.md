# 3 .1 Scope e namespace

### Definizione di scope
> Lo scope è una regione del testo di un programma. Si dice che una variabile *In scope* quando è in esso dal punto della dichiarazione fino alla fine dello scope.

L'obiettivo è rendere i nomi locali (BS) ed evitare interferenze (clash)
Gli scope sono di vario tipo:
- Globale: al di fuori di ogni altro scope 
- Di classe: il testo all'interno di una classe
- Locale: all'interno di un blocco o nella lista degli argomenti di una funzione
- Di statement: es. In un for
- Namespace: con nome inserito nello scope globale o in un altro namespace

```c
void f(int x) // f is global; x is local to f 
{
	int z = x+7; // z is local 
} 
	
int g(int x)  // g is global; x is local to g 
{ 
	int f = x+2; // f is local 
	return 2*f; 
 }
```

**MAI** usare scope globale, perché accessibile ovunque nel file sorgente e altri file inclusi, non il massimo della sicurezza.

Gli scope possono essere **annidati** cioè scope dentro scope, come due for loops oppure due funzioni di classi (schifo).

L'uso di namespace può appesantire il codice. Per renderlo più leggibile si possono usare due dichiarazioni:
- **Using declaration**: associa un namespace a un nome
```c
using std::string; 
using std::cout;
```
- **Using directive**: da usare solo per namespace molto noti (std)
```c
using namespace std;
```

# 3.2 Librerie e linking
### File header
è un insieme di dichiarazioni di entità:
- Usabili da chi include il file header
- Es. *math. H* dichiara *double sqrt (*double x)

Gli header di sistema sono inclusi con le parentesi triangolari
```c++
#include <iostream>
```

Gli header definiti dall'utente sono inclusi con le graffe
```c
#include "my_header.h" 
#include "opencv2/core.h"
```

# 3.3 – Funzioni e reference, efficienza
### Passaggio per valore
Passaggio di argomenti per valore, l'argomento viene inizializzato ad ogni chiamata. Si tratta di un valore copiato

è molto comodo efficace ed efficente su pochi dati che non devono essere modificati.

### Passaggio per riferimento
