Reference:
[[L06.1 - Allocazione dinamica della memoria.pdf]]
[[L06.2 - Liberare la memoria.pdf]]
[[L06_3-Liste.pdf]]
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
![[Pasted image 20240116132553.png]]
```c++
pd[2] = 2.2; // ok! 
pd[4] = 4.4; // ok! 
pd[-3] = -3.3; // ok, ma...
```
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
![[Pasted image 20240116132711.png]]

### Garbage
- Un'area di memoria di cui si perdono i riferimenti diventa garbage
- Tale zona di memoria rimane allocata ma è irraggiungibile e inservibile
- Questo fenomeno è noto come **memory leak**
	- Può portare all'esaurimento di memoria

---

# Liberare la memoria

È importante liberare la memoria quando non serve più, perché in C++ non lo fa nessuno al posto nostro. 
La produzione di garbage è un **errore tecnico grave**
La gestione corretta della memoria dinamica include la liberazione della memoria

La memoria va liberata manualmente per due motivi:
- *Efficienza*: il garbage collector impiega molte risorse, deve capire quale memoria è ancora raggiungibile
- *Controllo*: decidiamo quando la memoria è rilasciata, perciò di nuovo disponibile per altro

Per liberare la memoria si usano istruzioni dedicate:
- `delete`: se la memoria è allocata con `new`
- `delete[]`: se la memoria è allocata con `new[]`
Entrambi si applicano ai puntatori.

## Passaggio di memoria tra funzioni
Con l'allocazione dinamica della memoria possiamo creare un oggetto in uno scope (es., funzione) e passarlo a un altro scope (es., il chiamante).
In questo caso:

```c++
double *calc(int res_size, int max)
{
	double *p = new double[max];
	double *res = new double[res_size];
	delete[] p;
	return res;
}	// ... 
double *r = calc(100, 1000);	
// ... 
delete[] r;
```
calc() alloca res e lo ritorna al chiamante. È copiato solo il puntatore, la memoria allocata rimane tale quale.

Esempio visivo:
![[Pasted image 20240116084000.png]]
![[Pasted image 20240116084014.png]]

## Deallocazione nel chiamante

```c++
vector* f(int s)
{
	vector *p = new vector(s);	// fill p 
	return p;
}

void pf()
{
	vector *q = f(4);	// use q 
	delete q; //deallocazione del chiamante
}
```

## Doppia cancellazione
È un errore grave deallocare due volte la memoria
```c++
int *p = new int{5};
delete p;
	// ... – ma nessun uso di p delete p;
delete p;
```
- p non è più a nostra disposizione
- Il proprietario è il free store manager
- Potrebbe esserci un altro oggetto

## Dangling pointer
Dopo il delete, il puntatore mantiene lo stesso valore. Tale valore non è più valido ed è un errore utilizzarlo. Questa situazione prende il nome di **dangling pointer**

Per evitare il dangling pointer è opportuno settare il puntatore a nullptr.

## Liberare la memoria con UDT

```c++
class vector
{
	int sz;
	double *elem;
	public: vector(int s): sz{s}, elem{new double[s]}
	{
		for (int i = 0; i < s; ++i) elem[i] = 0;
	}
		// ... 
};

```
- A fine vita, è necessario deallocare la memoria
- Come possiamo fare per essere sicuri che ciò avvenga in maniera automatica?
	- Potrei usare una funzione clean_up(), ma se l'utente non la chiamasse?

## Distruttore
Un distruttore è una funzione che il compilatore chiama quando un oggetto deve essere distrutto, per esempio se:
- Esce dallo scope
- C'è deallocazione di un oggetto
Viene chiamato **implicitamente**

### Distruttore di Vector
```c++
class vector {
  int sz;
  double* elem;

 public:
  vector(int s) : sz{s}, elem{new double[s]} {
    for (int i = 0; i < s; ++i) elem[i] = 0;
  }
  ~vector() { delete[] elem; }  //distruttore!
  // ...
};
```
Chiamata implicita del distruttore
```c++
void f3(int n) {
  double* p = new double[n];
  vector v(n);  // ... uso p e v ...
  delete[] p;
}
```
In questo caso v è liberato automaticamente

I distruttori sono molto utili quando dobbiamo rilasciare risorse acquisite, come *File, Thread, Lock, Stream*.

## Distruttori creati dal compilatore
Quando il distruttore non è scritto esplicitamente, è generato automaticamente dal compilatore (**compiler-generated destructor**)
- Non è "intelligente": non dealloca al posto nostro
- Chiama i distruttori di eventuali oggetti membro
Tipi standard hanno il loro costruttore e viene chiamato dal compilatore

## Pattern acquisizione risorse
- Un oggetto acquisisce risorse nel costruttore
- Durante la sua vita, l'oggetto può:
	- Acquisire altre risorse
	- Liberare alcune risorse
- A fine vita, l'oggetto rilascia tutte le risorse

--- 
# Liste
Struttura dati con caratteristiche specifiche
- Accesso seriale
- Le operazioni di aggiunta e rimozione di un nodo sono veloci
- Utilizzo dell'allocazione dinamica
![[Pasted image 20240116111137.png]]

- Una lista è fatta di «link» che possiedono informazioni sul dato e puntatori ad altri link
- Una lista double-linked ha puntatori ai link precedente e successivo

```c++
struct Link {
  std::string value;
  Link* prev;
  Link* succ;
  Link(const std::string& v, Link* p = nullptr, Link* s = nullptr)
      : value{v}, prev{p}, succ{s} {}
};
```
Per accedere a un membro di Link, useremo l'operatore:
- Equivale a `(*…).xxxx`
- `(*p).field` equivale a` p->field

Per costruire la lista:
```c++
Link* norse_gods = new Link{"Thor", nullptr, nullptr};
norse_gods = new Link{"Odin", nullptr, norse_gods};
norse_gods->succ->prev = norse_gods;
norse_gods = new Link{"Freia", null_ptr, norse_gods};
norse_gods->succ->prev = norse_gods;
```

## Funzione insert
- L'inserimento è un'operazione molto comune!
- Ha senso usare una funzione dedicata
```c++
Link* insert(Link* p, Link* n) {  // inserisce n prima di p
  n->succ = p;
  p->prev->succ = n;
  n->prev = p->prev;
  p->prev = n;
  return n;
}
```
Vincolo: questa funziona se p punta a un Link, e se questo ha un prev!

## Funzione insert con check per nullptr

```c++
Link* insert(Link* p, Link* n) {  // inserisce n prima di p
  if (!n) return p;
  if (!p) return n;
  n->succ = p;
  if (p->prev) p->prev->succ = n;
  n->prev = p->prev;
  p->prev = n;
  return n;
}
```
Inserire i link ora è più semplice:
```c++
Link* norse_gods = new Link{"Thor"}; 
norse_gods = insert(norse_gods, new Link{"Odin"}); 
norse_gods = insert(norse_gods, new Link{"Freia"});
```
La gestione dei puntatori è nascosta!

## Operazioni su liste
Funzioni utili per la gestione di una lista:
- **Costruttore**
- **insert**: inserimento prima di un elemento
- **add**: inserimento dopo un elemento
- **erase**: rimozione di un elemento
- **find**: trova un Link con un certo valore
- **advance**: trova l'n-simo successivo

## Operazioni su liste

```c++
Link* add(Link* p, Link* n){...}  // esercizio
Link* erase(Link* p) {
  if (!p) return nullptr;
  if (p->succ) p->succ->prev = p->prev;
  if (p->prev) p->prev->succ = p->succ;
  return p->succ;
  // attenzione: ritorna un puntatore ma
  // non dealloca!
}
```

```c++
Link* find(Link* p, const string& s) {
  while (p) {
    if (p->value == s) return p;
    p = p->succ;  // anziché ++p
  }
  return nullptr;
}
```

```c++
Link* advance(Link* p, int n) {
  if (!p) return nullptr;
  if (0 < n) {
    while (n--) {
      if (!p->succ) return nullptr;
      p = p->succ;
    }
  } else if (n < 0) {
    while (n++) {
      if (!p->prev) return nullptr;
      p = p->prev;
    }
  }
  return p;
}
```

Ora posso usare le liste cos:
```c++
Link* norse_gods = new Link{"Thor"};
norse_gods = insert(norse_gods, new Link{"Odin"});
norse_gods = insert(norse_gods, new Link{"Zeus"});
norse_gods = insert(norse_gods, new Link{"Freia"});
Link* greek_gods = new Link{"Hera"}; greek_gods = insert(greek_gods, new Link{"Mars"}; greek_gods = insert(greek_gods, new Link{"Poseidon"};
```

## Modifica di liste
```c++
Link* p = find(norse_gods, "Zeus");
if (p) {
  erase(p);
  insert(greek_gods, p);
}
```
Funziona, ma ha due problemi:
- Cosa succede se p è `norse_gods`?
- `greek_gods` non è aggiornato

```c++
Link* p = find(norse_gods, "Zeus");
if (p) {
  if (p == norse_gods) norse_gods = p->succ;
  erase(p);
  greek_gods = insert(greek_gods, p);
}
```

## Stampa di liste

```c++
void print_all(Link* p) {
  cout << "{";
  while (p) {
    cout << p->value;
    if (p = p->succ) cout << ", ";
  }
  cout << "]";
}
print_all(norse_gods);
cout << "\n";
print_all(greek_gods);
```

# Puntatore this
## Migrazione a funzioni membro?
Molte delle funzioni viste prima accettano un Link* come argomento
- Ha senso che diventino funzioni membro?
- Sì, e ciò nasconde all'esterno l'uso dei puntatori
Nuova versione della classe

```c++
class Link {
public:
    string value;  // public: sono solo dati

    Link(const string& v, Link* p = nullptr, Link* n = nullptr) : value{v}, prev{p}, succ{n} {}

    Link* insert(Link* n);  // Inserimento prima di questo
    Link* add(Link* n);     // Inserimento dopo questo
    Link* erase();          // Rimuove questo
    Link* find(const string& s);
    const Link* find(const string& s) const;
    Link* advance(int n) const;

    Link* next() const { return succ; }
    Link* previous() const { return prev; }

private:
    Link* prev;
    Link* succ;
};
```

## Inserimento prima di questo oggetto
Se devo inserire prima di questo oggetto, ho bisogno di un riferimento
- **this** è un puntatore a questo oggetto
```c++
Link* Link::insert(Link* n) {
  Link* p = this;  // posso anche lasciare this
  if (n == nullptr) return p;
  n->succ = p;
  if (p->prev) p->prev->succ = n;
  n->prev = p->prev;
  p->prev = n;
  return n;
}
```