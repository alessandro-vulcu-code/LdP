Reference:
[[L07.1 - Inizializzazione di oggetti.pdf]]
[[L07.2 - Copia di oggetti.pdf]]
[[L07.3 - Puntatore this.pdf]]
[[L07.4 - Spostamento di oggetti.pdf]]
[[L07.5 - Accesso alle classi.pdf]]
[[L07.6 - Recap sulle classi.pdf]]
# Inizializzazione di oggetti

## Da basso ad alto livello (recap)
A basso livello (es., array)
- Un oggetto ha dimensione fissa
- Un oggetto ha una posizione fissa
- Esistono poche operazioni fondamentali
Ad alto livello (es., vector)
- Comportamento più intelligente
- Qualcuno deve implementarlo!

Il nostro vector:
```c++
class vector {
  int sz;
  double* elem;

 public:
  vector(int s = 0) : sz{s}, elem{new double[s]} {
    if (s == 0) elem = nullptr;
  }
  ~vector() { delete[] elem; }
  void push_back(double d);  //...
};
```

## Inizializzazione con Initalizer list
Una scrittura del tipo:
```c++
vector v1 = { 1.2, 7.89, 12.34 }; // più compatto!
```
utilizza un oggetto della standard library di tipo `initializer_list<T>`

Aggiungiamo quindi un costruttore:
```c++
class vector {  // ...
  vector(initializer_list<double> lst) : sz{lst.size()}, elem{new double[sz]} {
    std::copy(lst.begin(), lst.end(), elem);
  }  // ...
};
```
- std::copy algorithm: copia una sequenza di elementi delimitata dai suoi primi due argomenti a una sequenza puntata dal terzo argomento

Dettaglio:
```c++
vector(initializer_list<double> lst)
```
- È passato per copia!
- `initializer_list` è usato in questo modo, come richiesto dal linguaggio
- `initializer_list` è semplicemente un handle a elementi allocati "altrove"

# Casi di ambiguità
```c++
vector v1 {3}; 
vector v2(3);
```
Due modi diversi di inizializzare un vettore.

1. `vector v1 {3};`: Questa sintassi utilizza la sintassi di inizializzazione diretta con parentesi graffe `{}`. In questo caso, il vettore `v1` viene inizializzato con un singolo elemento di valore 3.

2. `vector v2(3);`: Questa sintassi utilizza la sintassi di costruzione con le parentesi tonde `()`. In questo caso, il vettore `v2` viene inizializzato con tre elementi, ognuno dei quali ha il valore di default per il tipo di elemento del vettore (nel caso di un `vector<int>`, come sembra essere il tuo esempio, il valore di default è 0).

```c++
vector v11 = {1, 2, 3}; 
vector v12 {1, 2, 3}; 
```

In questo caso sono uguali, solo che nel primo caso uso l'assegnazione dei valori con l'uguale, mentre nel secondo caso utilizzo `initializer_list`

---

# Copia di oggetti
Cosa succede se copio un vector?
```c++
void f(int n) {
  vector v(3);    // definisce un vettore di 3 elementi
  v.set(2, 2.2);  // assegna a v[2] il valore di 2.2
  vector v2 = v;  // cosa succede qui? // ...
}
```
Il vettore in questo caso non viene copiato fisicamente in una nuova area di memoria, `v2`condivide gli stessi elementi di `v`.

In uscita da f, sono chiamati i distruttori di v e v2
- È un problema: una volta chiamato il distruttore, la memoria puntatata da `elem`, viene liberata due volte -> errore!

## Costruttore di copia
Vogliamo modificare il comportamento della copia di default. Bisogna definire il costruttore di copia, cioè un costruttore che accetta una reference a oggetto della stessa classe

```c++
class vector {
  int sz;
  double* elem;

 public:
  vector(const vector&);  // costruttore di copia // ...
};
```

Implementazione del costruttore:
```c++
vector::vector(const vector& arg) : sz{arg.sz}, elem{new double[arg.sz]} {
  std::copy(arg.elem, arg.elem + sz, elem);
}
```

Ora il comportamento è diverso, avviene una **deep copy**
```c++
vector v2 = v; 
vector v2 {v}; // equivalente
```

Il costruttore di copia viene chiamato quando cerchiamo di inizializzare un `vector` con un altro `vector`

## Assegnamento di copia
Un problema analogo al precedente si verifica con l’assegnamento:
```c++
void f2(int n) {
  vector v(3);
  v.set(2, 2.2);
  vector v2(4);
  v2 = v;  // Notare che è un assegnamento! }
```
Comportamento di default: copia membro a membro


Problema: 
![[Pasted image 20240116104031.png]]

Stesso problema di prima: i due vettori non sono indipendenti!
- Doppia chiamata al distruttore
- Inoltre, perdita riferimento alla memoria allocata da v2

Implementazione tipo dell'assegnamento di copia:
```c++
vector& vector::operator=(const vector& a) {
  double* p = new double[a.sz];    // 1. alloca nuovo spazio
  copy(a.elem, a.elem + a.sz, p);  // 2. copia gli elementi
  delete[] elem;                   // 3. de-alloco vecchio spazio
  elem = p;                        // 4. resetto puntatore
  sz = a.sz;                       // 5. resetto dimensione
  return *this;                    // 6. ritorno self-reference
}
```

L'assegnamento è leggermente diverso dal costruttore: deve gestire i dati vecchi

Il comportamento proposto è il seguente:
- Creare una copia dei dati di source
- Eliminare gli elementi vecchi
- Far puntare elem ai dati nuovi

<span style="color:#00b0f0">Perché non liberare i dati vecchi prima, per risparmiare un puntatore?</span>
- Non è una buona idea eliminare dei dati finché non siamo sicuri di poterli rimpiazzare

Nell'assegnamento di copia dobbiamo tenere conto un caso particolare: l'*auto assegnamento*

```c++
v = v;
```

Tipico caso che dimostra che non bisogna eliminare i dati finché non siamo sicuri di poterli rimpiazzare, deve essere correttamente gestito dalla classe.

## Copia – terminologia

**Shallow copy** (copia superficiale): copia di puntatori o reference, senza copiare i dati

**Deep copy** (copia profonda):
- Copia dei dati
- *Sono definiti costruttore di copia e assegnamento di copia*
- Comportamento predefinito di `std::vector, std::string, …`

Prima abbiamo trasformato la shallow copy in deep copy

- Tipi che implementano una shallow copy hanno una *pointer semantics* (o *reference semantics*)

- Tipi che implementano una *deep copy* hanno una *value semantics*

---

# Puntatore this

`this` utilizzato nell'assegnamento di copia:
```c++
vector& vector::operator=(const vector& a) {
  double* p = new double[a.sz];
  copy(a.elem, a.elem + a.sz, p);
  delete[] elem;
  elem = p;
  sz = a.sz;
  return *this;
}
```

Qua ritorno una *self-reference* dell'oggetto stesso.

- **this** è un puntatore all’oggetto stesso
- È un parametro implicito dell’oggetto, generato automaticamente
- Viene usato quando all’interno di una funzione membro:
	- È necessario accedere all’oggetto corrente
	- È necessario ritornare un riferimento all’oggetto corrente

- Ad esempio:
	- Assegnamento di copia
	- Assegnamento di spostamento
	- ...

this è immutabile, non può in nessun caso essere modificato
```c++
struct S {  // ...
  void mutate(s* p) {
    this = p;  // errore! // ...
  }
};
```

## Utilizzo di this
Ritornare una reference all’oggetto corrente:
```c++
vector& vector::operator=(const vector& a) {
  double* p = new double[a.sz];    // 1. alloca nuovo spazio
  copy(a.elem, a.elem + a.sz, p);  // 2. copia gli elementi
  delete[] elem;                   // 3. de-alloco vecchio spazio
  elem = p;                        // 4. resetto puntatore
  sz = a.sz;                       // 5. resetto dimensione
  return *this;                    // 6. ritorno self-reference
}
```

Accedere all’oggetto corrente (da funzione membro):
```c++
class vector {
  int sz;
  double *elem;

 public:
  int size(void);  // ... };
  int vector::size(void) { return this->sz; }
```

- Non è strettamente necessario
- Questioni di stile

---
# Spostamento di oggetti

In questo codice:
```c++
vector fill(std::istream& is) {
  vector res;
  for (double x; is >> x;) res.push_back(x);
  return res;
}
void use() {
  vector vec = fill(cin);  // uso di vec
}
```

Stiamo copiando una potenziale grande quantità di dati in res fuori da `fill` e poi dentro `vec`
In realtà, difficilmente avremo mai un ulteriore bisogno di res all’interno di questa funzione

## Muovere i dati
Esiste un modo per spostare i dati, invalidando la sorgente, ed è l'operazione di **move** (complementa le operazioni di copia).
```c++
class vector {
  int sz;
  double* elem;

 public:
  vector(vector&& a);             // move constructor
  vector& operator=(vector&& a);  // move assignment
  // ...
};
```
- La notazione && è chiamata rvalue reference
1. **Gli argomenti non sono const (non possono esserlo!)**
   - Gli argomenti dei costruttori e operatori di assegnazione di spostamento non sono dichiarati come `const` perché l'obiettivo di queste operazioni è trasferire la proprietà dei dati da un oggetto all'altro. Non è necessario mantenere l'oggetto originale in uno stato coerente dopo il trasferimento, poiché l'oggetto sorgente verrà spesso distrutto o comunque non sarà più utilizzato.

2. **Le definizioni di operazioni di move tendono a essere più semplici delle copie**
   - Le operazioni di spostamento sono più semplici delle corrispondenti operazioni di copia perché coinvolgono il trasferimento diretto della proprietà dei dati, senza la necessità di duplicare fisicamente i dati sottostanti. Questo è particolarmente vantaggioso quando si lavora con oggetti che gestiscono risorse dinamiche, come la memoria allocata dinamicamente, in quanto il trasferimento della proprietà può essere eseguito in modo più efficiente rispetto alla duplicazione dei dati.


## Costruttore e assegnamento move
```c++
vector::vector(vector&& a) : sz{a.sz}, elem{a.elem} {
    // Ruba il puntatore ad 'a'
    a.sz = 0;            // Annulla 'a'
    a.elem = nullptr;    // Invalida il puntatore di 'a'
}

vector& vector::operator=(vector&& a) {
    delete[] elem;       // Dealloca lo spazio
    elem = a.elem;       // Riassegna il puntatore
    sz = a.sz;           // Riassegna il size
    a.elem = nullptr;    // Invalida il vecchio puntatore di 'a'
    a.sz = 0;            // Azzera il size di 'a'
    return *this;        // Ritorna una self-reference
}
```

## fill() e use() con move constructor

```c++
vector fill(istream& is) {
  vector res;
  for (double x; is >> x;) res.push_back(x);
  return res;
}
void use() {
  vector vec = fill(cin);  // uso di vec
}
```

È il compilatore che implementa il return tramite move constructor

## Move constructor vs. copy elision
- In certe circostanze il move constructor non è chiamato
- Sostituito da una tecnica di ottimizzazione del compilatore – copy elision
	- L'oggetto è costruito direttamente nella funzione chiamante
- È possibile richiedere al compilatore di non applicare la copy elision
	- Opzione -fno-elide-constructors

## Alternativa a move constructor
Sappiamo che allocazione dinamica + puntatori permettono ai dati di uscire da una funzione

```c++
vector* fill2(istream& is) {
  vector* res = new vector;
  for (double x; is >> x;) res->push_back(x);
  return res;
}
void use2() {
  vector* vec = fill2(cin);  // ... usa vec ...
  delete vec;
}
```

Più verbosa e soggetta ad errori.

---
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
class complex { 
	public: complex(double); // conversione da double a complex 
	complex(double, double); 
	// ... 
}; 
complex z1 = 3.14; // ok: conversione complex 
z2 = complex{1.2, 3.4};
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

Lo posso eliminare con *explicit*, e impedisco che il compilatore accetti le righe viste nel codice precedente. Evita cast indesiderati


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