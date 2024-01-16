# Scope e namespace

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

# Librerie e linking
### File header
è un insieme di dichiarazioni di entità:
- Usabili da chi include il file header
- Es. `math.h` dichiara `double sqrt (*double x)`

Gli header di sistema sono inclusi con le parentesi triangolari
```c++
#include <iostream>
```

Gli header definiti dall'utente sono inclusi con le graffe
```c
#include "my_header.h" 
#include "opencv2/core.h"
```

# Funzioni e reference, efficienza
### Passaggio per valore
Passaggio di argomenti per valore, l'argomento viene inizializzato ad ogni chiamata. Si tratta di un valore copiato

È molto comodo efficace ed efficente su pochi dati che non devono essere modificati.

### Passaggio per riferimento
Il passaggio per riferimento fornisce accesso in lettura e scrittura!
L'accesso in sola lettura è molto importante -> Passaggio per riferimento costante

- Una reference richiede un lvalue, possiamo scriverci
- Una const reference non richiede un lvalue

### Passaggio per riferimento const
```c++
void print(const vector<double>& v) {
  cout << "{";
  for (int i = 0; i < v.size(); ++i) {
    cout << v[i];
    if (i != v.size() – 1) cout << ", ";
  }
  cout << "}\n";
}
```

- Meccanismo efficiente della reference
- Accesso in sola lettura -> *verificato dal compilatore*

## Copia vs reference
Passaggio per valore per oggetti piccoli 
- Passaggio per const-reference per grandi oggetti da non modificare
- Preferibile ritornare un risultato che modificare un oggetto usando una reference 
- Usare una reference se è proprio necessario 
- Manipolare contenitori (es, `std::vector`) e altri oggetti grandi 
- Funzioni che devono produrre più di un output 
- Si suppone che una funzione che accetta una reference modifichi l'oggetto passato

# Check degli argomenti
## Conversioni automatiche
- Utili, ma possono produrre risultati indesiderati
- Difficile dire se questo codice ha un errore o no

## Conversioni esplicite
- Un cast esplicito è più espressivo: un altro programmatore capisce che è intenzionale
```c++
void ggg(double x) {
    int x1 = x;                   // troncamento
    int x2 = int(x);
    int x3 = static_cast<int>(x); // conversione esplicita
    ff(x1);
    ff(x2);
    ff(x3);
    ff(x);
    ff(int(x));
    ff(static_cast<int>(x));      // conversione esplicita
}

```