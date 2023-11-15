## Accesso ai membri della classe
Come facciamo ad accedere agli slot del vettore?

Un modo è quello di implementare _get ()_ e _set ()_. Sintatticamente però è meglio rispettare l'uso delle parentesi quadre, quindi meglio vare overloading dell'operatore  _[]_

```c++
class vector { int sz; double* elem; public: double operator[] (int n) { return elem[n]; } // ... };
```
Al momento [] permette lettura ma non scrittura. Quindi una possibile soluzione è usare il puntatore, ma poco elegante e bisognerebbe dereferenziare.

Posso usare le *reference*, 
```c++
class vector { 
	int sz; 
	double* elem; 
	public: 
		double& operator[] (int n) { return elem[n]; } 
		// ... 
};
```

Così è permesso di usarlo come variabile.

Problemi:
- Non è possibile usarlo su *const*
### Overloading su const
È possibile fare l'overloading di una funzione const e non const
```c++
class vector { 
	int sz; double* elem; 
	public: 
		double& operator[] (int n); // per vettori non costanti 
		double operator[] (int n) const; // per vettori costanti 
		// ... 
};
```
La seconda viene chiamata solo su vettori const, ritorna solo la copia e non ci possiamo scrivere. Non è un problema perché è impossibile scriverci.

Quindi ora possiamo scrivere:
```c++
void ff(const vector& cv, vector& v) { 
	double d = cv[1]; // ok: usa const [] 
	cv[1] = 2.0; // errore: usa const [] 
	d = v[1]; // ok: usa non-const [] 
	v[1] = 2.0; // ok: usa non-const [] 
}
```

### Reference a membro privato
La versione di *operator[]* ritorna una reference a un membro privato. Si tratta di una cosa corretta?
Risposta: generalmente non è una buona idea, però dipende dai casi. Vedi:
- https://stackoverflow.com/questions/4706788/why-can-i-expose-private-members-when-i-return-a-reference-from-a-public-member
- https://stackoverflow.com/questions/8005514/is-returning-references-of-member-variables-bad-practice
---
## Progettazione di inizializzazione, copia, spostamento
Si tratta di un recap per noi poveri che non studiamo.

### Operazioni essenziali
Da considerare, sono:
1. Costruttore con uno o più argomenti 
2. Costruttore di default 
3. Costruttore di copia (copy constructor) 
4. Assegnamento di copia (copy assignment) 
5. Costruttore di spostamento (move constructor) 
6. Assegnamento di spostamento (move assignment) 
7. Distruttore

Da oggi in poi per una classe bisogna rispettare questa lista di cose da implementare

### Costruttore di default
Osserviamo che:
```c++
std::vector<double> vi(10); 
std::vector<std::string> vs(10); 
std::vector<std::vector<int>> vvi(10);
```

Workano perchéci sono costruttori di default di:
- Double
- Std:: string 
- Std:: vector (int) e int

### Copia e spostamento
Necessario se acquisiamo risorse (che devono essere liberate) 
- Risorse: 
	- Memoria nel free store (allocata dinamicamente) 
	- File 
	- Lock, thread handles, socket
- Probabilmente è necessario se ci sono membri puntatori

### Copia e spostamento
Se c'è distruttore allora serve anche una funzione di copia e spostamento
- Costruttore di copia
- Assegnamento
- Spostamento
- Assegnamento di spostamento
Ci sono risorse da gestire


### Costruttori e conversioni
Un costruttore che riceve un singolo argomento definisce: una conversione implicita da quell'argomento alla classe

```c++
class complex { public: complex(double); // conversione da double a complex complex(double, double); // ... }; complex z1 = 3.14; // ok: conversione complex z2 = complex{1.2, 3.4};
```

A volte questa cosa ci piace poco, per esempio in vector, c'è un costruttore che accetta un int, usato per costruire vettori di n elementi

```c++
class vector { 
	// ... 
	vector(int); 
	
	// ... 
}; 

vector v = 10; // crea un vector di 10 elementi 
v = 20; // assegna un vector di 20 elementi void f(const vector&); f(10); // chiama f con un vettore di 10 elementi
```

Lo posso eliminare con *explicit*, e impedisco che il compilatore accetti le righe viste nel codice precedente.


### Analizzare costruttori e distruttori
Un costruttore è chiamato quando si crea un oggetto:
- Oggetto inizializzato
- New
- Oggetto copiato

Distruttore chiamato quando:
- Un nome esce dallo scope
- è usato delete

#### Esercizio
Implementare questo codice:
```c++
X glob(2); // variabile globale 

X copy(X a) { return a; } 

X copy2(X a) { X aa = a; return aa; } 

X& ref_to(X& a) { return a; } 

X* make(int i) { X a(i); return new X(a); } 

struct XX { X a; X b; }; // segue
```

```c++
 int main() { X loc {4}; // var locale 
 
 X loc2 {loc}; // costruttore di copia 
 
 loc = X{5}; // assegnamento di copia 
 
 loc2 = copy(loc); // call by value e return 
 
 loc2 = copy2(loc); 
 
 X loc3 {6}; X& r = ref_to(loc); // call by reference e return 
 
 delete make(7); 
 
 delete make(8); 
 
 vector<X> v(4); 
 
 XX loc4; X* p = new X{9}; 
 
 delete p; X* pp = new X[5]; 
 
 delete[] pp; }
```

- In funzione del compilatore usato, alcune copie di copy e copy 2 potrebbero non essere eseguite
- Una copia di un oggetto non utilizzato può non essere eseguita
	- Il compilatore è autorizzato a ritenere che un costruttore di copia esegua solamente una copia, e nient'altro