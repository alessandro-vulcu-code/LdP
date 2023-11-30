# Allocare la memoria usando il free store 
## Creare una classe std::vector
### Std ::vector
Ha comportamento intelligente
- Cambia dimensione per ospitare gli elementi che inseriamo
- Conosce la propria dimensione
Non è supportato in HW e non può essere implementato con gli array, che hanno una dimensione fissa.

A basso livello gli strumenti a disposizione sono molto meno evoluti:
- Memoria
- Indirizzi di memoria
Il comportamento intelligente di `std:.vector` è implementato nella libreria standard

## Implementare una classe vector
Come possiamo implementare una classe `vector` di double con le stesse caratteristiche si `std::vector`?
### Struttura di un vector
```c++
vector age(4); 
age[0] = 0.33; 
age[1] = 22; 
age[2] = 27.2; 
age[3] = 54.2;
```

Questo è un caso semplice dove creo un vettore di dimensione 4 e ad ogni posizione assegno un valore double.
Quindi il vettore ha size = 4.

- Il vettore è una struttura dinamica
- Per poter realizzare una struttura dinamica ho bisogno di:
	- Un modo per gestire gli insiemi di elementi
	- Crearli, modificarli, eliminarli
	- Un modo per riferirmi a tali insiemi
- Per riferirmi a tali usiamo i *puntatori*

### Vector con i puntatori
Posso scrivere vector in questo modo:
```c++
class vector { 
	int sz; 
	double *elem; 
	
	public: 
		vector(int s); // crea un vettore di s double e fa sì che elem punti ad
						// esso 
		int size() const { return sz; } 
};
```

Il puntatore deve essere usato in modo da gestire un buffer.

### Gestione della memoria dei vector
*Come rendo variabile il numero di elementi?*
- Il sistema operativo una volta che viene avviato il programma riserva spazio per:
	- Codice
	- Variabili globali
	- Chiamate a funzione (stack)
- Il resto è disponibile: **free store**

Si può accedere al free store usando i *puntatori* e *new*.
- *new* usa il free store e ritorna il puntatore al primo elemento dell'array
- Si parla di *allocazione dinamica della memoria*

### Puntatori e Free Store
- *new* ritorna elementi singoli o array
- Un puntatore **non sa** quanti oggetti sono presenti nell'array ritornato da *new*
- *new* ritorna un puntatore anche se allochiamo un solo elemento
- La memoria ritornata da *new* **non ha nome**
	- Memoria gestita tramite il puntatore restituito

```c++
int* pi = new int; 
int* qi = new int[4]; 

double* pd = new double; 
double* qd = new double[n];
```

### Accesso agli elementi
```c++
double* p = new double[4]; 
double x = *p; // primo oggetto puntato 
double y = p[2]; // terzo oggetto
```

Nella gestione della memoria dinamica usiamo spesso:
- Il nesso tra puntatori e array
- L'aritmetica dei puntatori

Ricordiamo che:
- * p e p[0] sono equivalenti
- Applicare [] a un puntatore equivale a vedere la memoria come un array

### Range e range check
A questo livello non esiste controllo sui range!
![[Pasted image 20231126172855.png]]

- Grave problema
- Esecuzioni diverse possono dare sintomi diversi
- Bug molto difficili da gestire!

### Inizializzazione dei puntatori
```c++
double* p0; // grosso problema! 

double* p1 = new double; // double non inizializzato 

double* p2 = new double {5.5}; // double inizializzato 

double* p3 = new double[5]; // array non inizializzato
```

- Memoria allocata con *new* non è inizializzata per i tipi built-in, per ovviare

```c++
double* p4 = new double[5] {0, 1, 2, 3, 4}; // array init 
double* p5 = new double[] {0, 1, 2, 3, 4}; // dimensione 
										  // dedotta
```

### Inizializzazione con UDT
```c++
X* px1 = new X; 
						//Chiamata al costruttore di default
X* px2 = new X[17];
```

Se non esiste:
```c++
Y* py1 = new Y; // errore 

Y* py2 = new Y[17]; // errore 

Y* py3 = new Y{13}; // ok 

Y* py4 = new Y[17] {1, 2, 3, 4, ..., 16}; // ok
```

### Nullpointer
- Molto comodo avere un valore non valido per segnalare un puntatore non inizializzato
```c++
double* p0 = nullptr;
```

### Controllo di validità
```c++
if (p0 != nullptr) {/* ... */} 

if (p0) {/* ... */}
```

Attenzione quando:
- Il puntatore non è mai stato inizializzato
- **Non è inizializzato a nullptr automaticamente!**
- Il puntatore si riferisce a memoria non valida (liberata)

### Spostamento di un puntatore
- Un puntatore può essere spostato da un oggetto a un altro
```c++
double* p = new double; 
double* q = new double[1000]; 

q[700] = 7.7; // ok 
q = p; // attenzione!! 
double d = q[700]; // attenzione!!
```

Questo può generare grossi problemi
```c++
double* p = new double; 
double* q = new double[1000]; 

q = p; // attenzione!!
```

![[Pasted image 20231126173739.png]]

### Garbage
- Un'area di memoria di cui si perdono i riferimenti diventa garbage
- Tale zona di memoria rimane allocata ma è irraggiungibile e inservibile
- Questo fenomeno è noto come **memory leak**
	- Può portare all'esaurimento di memoria
---
