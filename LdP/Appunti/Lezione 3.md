Reference:
[[L03_1-Scope.pdf]]
[[L03_2-Librerie.pdf]]
[[L03_3-Funzioni.pdf]]
[[L03_4-Chiamate_funzioni.pdf]]
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
È un insieme di dichiarazioni di entità:
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

Come compilare un progetto composto da molti file?
- Preprocessore, compilatore, linker!
![[Pasted image 20240116183250.png]]
## Librerie statiche
Quello che abbiamo appena visto è un linking statico (libreria statica)
## Librerie dinamiche
Una sola copia della libreria vale per tutti

Vantaggi e svantaggi:
**Librerie dinamiche**:
- Riduzione dello spazio disco occupato (una sola copia funziona per tutti gli eseguibili)
- Possono essere ricompilate senza toccare gli eseguibili
- Chiamate .so (Shared Object) sotto Linux e .dll (Dynamic Linking Library) sotto Windows

**Librerie statiche**
- Generano eseguibili che non possono essere spezzati in seguito
- Sono self-contained
- Più adatte alla distribuzione di software monolitico

## Come creare librerie statiche
![[Pasted image 20240116183655.png]]

## Come creare librerie dinamiche
![[Pasted image 20240116183713.png]]

---
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


# Chiamate a funzione e memory layout

## Chiamata a funzione
Una chiamata a funzione causa la creazione di una struttura dati chiamata **RA – Record di Attivazione (Function Activation Record)**

La RA contiene:
- una copia dei parametri
- variabili locali

 I RA sono impilati in una zona di memoria chiamata **stack**, una pila, con struttura LIFO (Last In First Out).

La dimensione della RA dipende dal numero (e dal tipo) di variabili locali. Il tempo necessario per inserire il RA nello stack è costante.

## Record di Attivazione | Struttura dati
![[Pasted image 20240116202315.png]]
- *Implementazione*: informazioni che servono per ritornare al chiamante e fornire un risultato (`return`)
- Parametri e variabili locali sono equivalenti in questo schema
- Ogni chiamata a f o g ha il proprio RA

## Meccanismo di chiamata a funzione
![[Pasted image 20240116202733.png]]
In questo stack:
1. Chiamo funzione f con f1 e f2
	1. Esce f
2. Chiamo funzione g con g1
	1. esce g

Casi più complessi possono avere anche RA "mischiati", in base a come vengono chiamate le funzioni, come in questo caso:
![[Pasted image 20240116203020.png]]

# Rappresentazione in memoria di un programma
## Layout di memoria
![[Pasted image 20240116203205.png]]

### Text segment
- AKA code segment / text
- Copiato in memoria dal file oggetto
- Contiene le istruzioni eseguibili
- Spesso read-only
- Spesso condiviso

### Initialized data segment
- AKA data segment
- Contiene variabili globali e variabili statiche
- Read/write - può contenere una parte read-only

### Uninitialized data segment
- AKA BSS segment (block started by symbol, per ragioni storiche)
- Inizializzato a 0 dal SO
- Contiene variabili globali e statiche non inizializzate esplicitamente
```c++
int glb_i; // in BSS 
int main(void) { 
	// … 
	return 0; 
}
```

### Stack
- Contiene i RA delle funzioni, quindi le variabili locali e i parametri

## Heap
- Formalmente è spesso visto come uno spazio che cresce in verso opposto allo stack
- La sua dimensione può essere modificata con le funzioni di sistema brk e sbrk
- Può essere gestito usando blocchi di memoria non contigui (funzione di sistema mmap)
- Gestito tramite `new` e `delete` (allocazione dinamica della memoria)

# Ordine di valutazione di un programma
La valutazione di un programma corrisponde alla sua esecuzione. Quando l’esecuzione raggiunge il codice che identifica la definizione di una variabile:
1. La variabile viene «costruita»
2. La memoria viene assegnata
3. L’oggetto viene inizializzato

Quando la variabile va out-of-scope
- La memoria viene liberata e può essere utilizzata per altro

## Valutazione delle variabili

**Variabili globali**
- Inizializzate prima della prima istruzione del main
- Esistono fino al termine del programma

**Variabili automatiche**
- Di base, tutte le variabili locali sono automatiche
- Esistono solo all’interno dello scope
- Vengono ricreate ogni volta (es., se all’interno di una funzione)

**Variabili statiche (static)**
- Inizializzate solo la prima volta
- Mantengono il valore anche al di fuori del loro scope
- Possono essere globali o locali (se dichiarate dentro a una funzione)

```c++
#include <iostream>

void f(void) {
    int local_auto = 0;
    static int local_static = 0;
    std::cout << "Auto: " << local_auto++ << ", static: " << local_static++ << '\n';
}

int main(void) {
    f(); // "Auto: 1, static: 1"
    f(); // "Auto: 1, static: 2"
    f(); // "Auto: 1, static: 3"

    return 0;
}

```

- I compilatori fanno sì che il codice si comporti come se le azioni descritte prima fossero eseguite
- Varie ottimizzazioni sono possibili
- I compilatori spesso modificano l'implementazione per ottimizzare, ma il comportamento è immutato

**Inizializzazione globale**
- Le variabili globali sono inizializzate prima dell'esecuzione della prima istruzione del main
- Se le variabili di diverse translation unit hanno una dipendenza, non sappiamo in che ordine sono effettuate le inizializzazioni
- Variabili globali hanno la memoria scritta a 0 prima delle inizializzazioni