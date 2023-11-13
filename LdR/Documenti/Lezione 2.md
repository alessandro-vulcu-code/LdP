# 2.1 Variabili e type safety

### Tipo, oggetto, variabile, valore
- Il **tipo** definisce il tipo del contenuto, un range di valori e un insieme di operazioni per un oggetto
- L'**oggetto** è una regione di memoria (RAM) con un tipo che specifica quale tipo di dato può essere inserito
- La **variabile** è un oggetto con un nome 
- Il **valore** è l'elemento posto dentro alla variabile

### Tipo di una variabile
- Esso determina le operazioni che possono essere effettuate e il loro significato
- Ogni tipo ha il suo formato **literal**
```c++
39 // int 
3.5 // double ù
'.' // char 
"Annemarie" // string 
true // bool
```

# Scrivere su una variabile
### Assegnamento e inizlializzazione
- L'uguale è usato per assegnare un valore ad una variabile
- Inizializzazione, assegnamento e assegnamento composto

### Type safety
Il tipo di una variabile condiziona
- La natura dei dati e contenuti
- La dimensione in byte
Passare dati tra variabili dello stesso tipo è semplice, ma farlo tra variabili di diverso tipo porta a perdita di dati

Alcune operazioni non sono sicure di default, ad esempio usare una variabile non inizializzata.
```c++
double x; // valore di x indefinito 
double y = x; // valore di y indefinito
```

Le conversioni sicure sono quelle che mantengono il valore, essenzialmente sono sicure le conversioni verso un tipo con maggiore capacità.
- Bool → char 
- Bool → int 
- Bool → double 
- Char → int 
- Char → double 
- Int → double

Le conversioni da int a double permettono di usare espressioni che contengono entrambi i tipi, quindi se sommo 2.3 a 2 viene automaticamente convertito in 2.0

### Conversioni non sicure
- Problema del Narrowing
- Il compilatore raramente rileva questo problema
- Per sapere la dimensione in byte di un tipo si usa `sizeof ('tipo')`

### Controllo su inizializzazione
- Conversioni non sicure sono accettate per eredità storiche da C
- C++ 11 introduce un modo di inizializzazione cheevita le narrowing conversions usando le graffe
```c++
double x {2.7}; // OK 
int y {x}; // errore: narrowing conversion 
int a {1000}; // OK 
char b {a}; // errore: narrowing conversion 
char b1 {1000}; // errore: valore fuori limiti 
char b2 {48}; // OK 
```

Ci sono due modi per convertire:
- `Tipo {valore}` - con controllo di narrowing
- `Tipo {valore}` - senza controllo

```c++
double d = 2.5; 
int i = 2; 

double d2 = d/i; // d2 == 1.25 
int i2 = d/i; // i2 == 1 
int i3 {d/i}; // errore: double -> int, narrowing
```

## Dare nomi alle variabili

Importante usare nomi comprensibili, cioè non acronimi o nomi strani.
Evitare anche nomi lunghi, per nomi brevi sono per convenzioni o usati in ambito molto ristretto.

Meglio scegliere una convenzione e rispettarla, perché rende molto più semplice la lettura di codice. Una convenzione suggerita è quella di Google:
https://google.github.io/styleguide/cppguide.html#Naming


# 2 .2 Espressioni e operatori

## Computazione
Il programmatore deve esprimere la computazione in modo
- Corretto
- Semplice
- Efficente
è spesso gestita dividendo componenti complessi in un numero maggiore di componenti semplici.

## Astrazione
Concetto fondamentale in programmazione
- Nascondere dettagli implementativi che non servono per usare una risorsa

Interfaccia: definizione di come usare una risorsa o una funzione


## Espressioni
- Parte di codice che da un certo numero di operandi calcola un output
- Il più piccolo elemento usato per esprimere la computazione
- Possono essere espressioni i Literal o i nomi delle variabili. Più espressioni semplici insieme ne creano di più complesse

### Lvalue e rvalue
Lvalue è ciò che sta alla sinistra di un operatore di assegnamento.
Il nome di una variabile NON è sempre lvalue. In questo codice: 
```c++
int length; 
length = 99;
length /= 2; 
int width; 
width = length * 2;
```
Length si riferisce: 
- Come lvalue: a un oggetto di tipo int che contiene il valore 99 – length è il box (l'oggetto) 
- Come rvalue: il riferimento al valore contenuto nell'oggetto

Ci sono espressioni costanti, C++ permette di esprimere una costante simbolica. Sono meglio dei Magic Numbers, quindi vanno usate sennò in laboratorio viene contato come errore.

```c++
constexpr int max = 100; 
void use (int n) { 
	constexpr int c1 = max + 7; // OK: da costanti 
	constexpr int c2 = n + 7; // errore: non conosciamo il 
							// valore a tempo di compilazione
	//...;
	}
```

Ci sono due modi per esprimere una costante:
- Constexpr: il valore è noto a tempo di compilazione
- Const: il valore è noto solo a tempo di esecuzione ed è assegnato in inizializzazione.


## Side effect e valutazione delle espressioni

Operatori side effect modificano gli operanti su cui operano e ne forniscono un risultato.
- Es. ++, --, +=, -=, ...

In che ordine sono valutate le espressioni?
```c++
int perimeter = (length + width) * 2;
```
Operazioni:
- Lettura di lenght
- Lettura di width
- Somma
- Prodotto
- Assegnamento

L'ordine delle prime due operazioni è gestito dal compiler, quindi cerca di ottimizzare le operazioni. 

L'ordine di valutazione delle espressioni non è definito, quindi è sbagliato usare la stessa variabile più di una volta se su essa sono applicati operatori con side effect.

## Tipi che influenzano operatori

### Operatori << e >>
- Operatore << inserimento in uno stream
- Operatore complementare >> estrae da uno stream
- Fanno una trasformazione dalla forma  testuale alla rappresentazione interna delle variabili
	- Sono sensibili al tipo
- In sostanza se la variabile richiesta è `int` non posso mettere una stringa, o meglio se succede prende ogni lettera della stringa e la converte in intero.

# 2 .3 Istruzioni, dichiarazioni, definizioni
## Statements

Lo statement o istruzione è una parte di codice C++ che specifica un'azione, che termina con ;. Non sono istruzioni le direttive del preprocessore.

Alcuni tipi di istruzione sono:
- Expression statements
- Dichiarazioni
- Selezione (if, switch)
- Iterazione (while, for)

## Dichiarazione
Istruzione che introduce un nome in uno scope, cioè specifica del nome, tipo ed eventualmente un'inizializzazione. 
Un nome deve essere dichiarato per essere usato (vale per funzioni variabili ec..)

Una dichiarazione ha effetto sulla memoria? NO, perché vado solo a definire un nome in modo che il codice capisca a cosa mi sto riferendo.

## Definizione
Una dichiarazione che specifica completamente l'entità dichiarata
Una definizione è anche una dichiarazione. 
```c++
int a = 7; vector<double> v; 
double sqrt (double d) { /* ... */ }
```

Alcune dichiarazioni *non* sono definizioni
```c++
double sqrt (double); 
extern int a;
```

In alcuni contesti dichiarazioni e definizioni sono mutuamente escusivi.

### Dichiarazioni vs definizioni

Per le variabili:
- Dichiarazione fornisce il nome della variabile e il tipo (niente sulla memoria)
- Definizione fornisce un oggetto riservato in memoria con un determinato nome e tipo

Per le funzioni:
- Dichiarazione fornisce il tipo (degli argomenti e del valore ritornato)
- Definizione fornisce il corpo (occupa spazio in memoria)

### Inizializzazione
Bisogna sempre inizializzare le variabili, soprattutto se sono dichiarati const, perché il compiler dà errore
Usare la graffa per inizializzare è più bello ma è poco diffuso, è uguale all'uguale.

Cosa succede se non inizializziamo una variabile? Ad esempio se confrontiamo un valore x non inizializzato con un valore z, non dà errore. In x ci sarà il valore precedentemente presente in memoria.