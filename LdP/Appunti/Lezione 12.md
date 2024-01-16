# Algoritmi STL – Introduzione e algoritmi di ricerca
- STL offre un grande numero di algoritmi 
- Algoritmi compatibili con tutti i container STL
- `#include <algoritm>`

![[Pasted image 20231219124344.png]]

---

### Find ()
- Parametro template
- Cerca un elemento con un dato valore in una sequenza
- Implementazione efficente e veloce

```c++
template<typename In, typename T> 
// In: iteratore di input 
// T deve essere confrontabile 
In find(In first, In last, const T& val) { 
	while(first != last && *first != val) ++first; 
	return first; 
}
```

- La sequenza su cui opera find () è definita mediante due iteratori 
	- `[first, last)` -> last è l'ultimo elemento non compreso, quindi sappiamo che non abbiamo trovato nulla se ci arriviamo
- `find()` è generico
- La chiamata a `find()` non cambia in funzione di questi due elementi
- Flessibilità comune agli algoritmi STL
	- Effetto dell'interfaccia standard gestita tramite iteratori

Esempio con differenze:
![[Pasted image 20231219125025.png]]

--- 

## Algoritmi con predicati

## find_if ()
- Find () confronta valori
- Find_if () verifica che un criterio sia soddisfatto

```c++
template<typename In, typename Pred> 
// In: iteratore di input 
// Pred è un predicato 
In find_if(In first, In last, Pred pred) { 
	while(first != last && !pred(*first)) ++first; 
	return first; 
}
```

- Criterio è espresso da un predicato
- Un predicato è una funzione che ritorna true o false
```c++
bool odd(int x) { return x%2; } 
void f(vector<int>& v) { 
	auto p = find_if(v.begin(), v.end(), odd); 
	if (p != v.end()) { /* numero dispari */ } 
}
```

- Posso passare una funzione con un puntatore
- Pred diventa un puntatore a funzione che accetta un intero e restituisce un bool, quindi odd è di un tipo definibile, quindi posso creare un puntatore che punta alle funzioni che accettano agli stessi argomenti.

### Passaggio di valori a un criterio
- Prendiamo in considerazione il criterio "maggiore di un dato valore"
	- Il valore è un parametro che deve essere passato alla funzione
	- Odd () non ne aveva bisogno
- <span style="color:#00b0f0">Come possiamo passare un valore a un predicato?</span>
	- Non possiamo usare argomenti – non esistono () nel puntatore a funzione

---
## Function object
- Un function object è un oggetto che può essere usato come una funzione
- Ci permette di creare un predicato -> overloading di ()

```c++
class Larger_than { 
	int v;
public: 
	// salvataggio argomento 
	Larger_than(int vv) : v{vv} {} 
	// confronto 
	bool operator() (int x) const { return x>v; } //function call operator o application operator
};
```

- Reminder: find_if effettua una chiamata di questo tipo:
```c++
while(first != last && !pred(*first)) ++first;
```

- È tradotta in una chiamata a `operator()`

- La funzione brutta di merda di prima diventa:

```c++
void f(list<double>& v, int x) { 
	auto p = find_if(v.begin(), v.end(), Larger_than(31)); 
	if (p != v.end()) { /* valore trovato */ } 
	
	auto q = find_if(v.begin(), v.end(), Larger_than(x)); 
	if (q != v.end()) { /* valore trovato */ } 

	// ... 
}
```

#### Funzionamento
- Function object creato nell'argomento
- Argomento passato a `find_if()`
- `Find_if()`, al suo interno, chiama `operator()`

- Usare i function object è una tecnica efficiente
- Per una migliore efficienza:
	- Piccoli function object passati per copia
	- Se più grandi di qualche byte, meglio per riferimento
- `Operator()` inline

Questa combinazione è più efficiente di una chiamata a funzione!

---

## Ordinamento e copia
Debolezza dei function object
- Definiti in una regione di codice
- Usati altrove

È possibile rimuovere questo fenomeno usando un altro strumento:
- Lambda expression, un modo per:
	- Definire un function object
	- Crearne uno immediatamente e usarlo come argomento di una funzione
	- Tale function object è anonimo e non utilizzabile altrove

### Sort () e lambda expression

#### Sort()
- Effettua un ordinamento
- Due versioni: 
	- Ordinamento con operator< 
```c++
template<typename Ran> 
	// Ran deve possedere un random_access_iterator 
void sort(Ran first, Ran last);
```

- Ordinamento con funzione dedicata

- Cmp è un comparison function object. Deve soddisfare i requisiti di Compare:
	- Accetta due argomenti a, b
	- Risultato convertibile implicitamente a bool
	- Risultato significa "a compare prima di b" in questo ordinamento
- Compare è un named requirement

`Sort()` ordina tra due iteratori. Possiamo quindi ordinare anche solo una parte di un container.
Spesso ci interessa ordinare un intero contenitore, è disponibile la funzione sort () per i contenitori:
- Non è parte della STL
- Definita in `std_lib_facilities.h`
```c++
template<typename C> 
// C è un contenitore 
void sort(C& c) { sort(c.begin(), c.end()); }
```

- Un esempio con l'algoritmo sort () e una lambda expression
```c++
struct Record{ 
	string name; 
	char addr[24]; 
	// ... 
};
```

### Criteri di ordinamento
- Dato un vector di record
- Abbiamo due modi di ordinarlo – in base a due diversi predicati:
```c++
struct Cmp_by_name{ bool operator()(const Record& a, const Record&b) const { return a.name < b.name; } }; struct Cmp_by_addr{ bool operator()(const Record& a, const Record&b) const { return strncmp(a.addr, b.addr, 24) < 0; } }; 
```

- Esistono due criteri di ordinamento perché sono presenti due dati membro
	- Situazione tipica quando si ordina in base ai dati membro
```c++
//... 

sort(vr.begin(), vr.end(), Cmp_by_name()); 

// ... 

sort(vr.begin(), vr.end(), Cmp_by_addr()); 

// ...
```

- Questa situazione è spesso gestita con le lambda expression
```c++
// ... 
sort(vr.begin(), vr.end(), 
	 [] (const Record& a, const Record& b) 
		 { return a.name < b.name; } 
); 
// ... 

sort(vr.begin(), vr.end(), 
	 [] (const Record& a, const Record& b) 
		 { return strncmp(a.addr, b.addr, 24) < 0; } 
); 
// ...
```

Lambda è poco immediato.

### Lambda expression
- Le lambda expression accettano argomenti come le funzioni
- Possono anche catturare le variabili locali
- La cattura delle variabili locali rende le lambda expression molto comode
![[Pasted image 20231219133734.png]]

### copy()
- Effettua una copia 
```c++
template<typename In, typename Out>
Out copy(In first, In last, Out res) {
    while (first != last) {
        *res = *first;
        ++res;
        ++first;
    }
    return res;
}

```

- Copia di una sequenza in un'altra sequenza
	- Possono essere container diversi
- Spetta all'utente verificare che sia disponibile sufficiente spazio a destinazione
	- Politica simile al range check per i vector: non sono effettuate operazioni potenzialmente costose e non sempre necessarie

Utilizzo:
- **Sorgente**: segnalati a inizio e fine
- **Destinazione**: segnalato solo l'inizio
```c++
void f(vector<double>& vd, list<int>& li) {
  if (vd.size() < li.size()) error("Target container too small\n");
  copy(li.begin(), li.end(), vd.begin());
}
```

Esiste anche la copia con verifica di un predicato, stessa sintassi, con un quarto argomento che specifica il predicato
```c++
void f(const vector<int>& v) {
  vector<int> v2(v.size());
  copy_if(v.begin(), v.end(), v2.begin(), Larger_than(6));
}

```

Algoritmi numerici 
- `Accumulate` 
- `Inner_product` 
- `Partial_sum` 
- `Adjacent_difference`

---

## Associative container
### Map
- Gestisce coppie chiave-valore
- Contenitore ordinato
- Ottimizzato per ricerche dalla chiave
- Rappresentato con un albero bilanciato 
- Header: `<map>`

```c++
int main() { 
	map<string, int> words; 
	for(string s; cin >> s; ) //elemento cruciani
		++words[s]; 
	for(const auto& p : words) //ciclo sugli elementi di words
		cout << p.first << ": " << p.second << '\n'; return 0; 
}
```

### Indicizzare map
- Indicizzazione con una string
- A partire da string, la map fornisce il relativo int
- Se la chiave non esiste nella map, è creato un elemento con quella chiave
	- Il corrispondente int è inizializzato a 0 – costruttore di default di int

### Pair
- Gli elementi della map sono coppie chiave- valore
	- Gestite come pair<string, int>
	- Elementi di una pair: first e second
	- Per creare una pair: make_pair ()

#### Map vs vector
- Map ordina i dati rispetto alla chiave
- Map offre una ricerca molto più veloce a
- Associazione esplicita di una chiave a un valore

### Set
- Un set è una map senza i valori
- Viene meno la ricerca del valore a partire dalla chiave
	- Non è disponibile `operator[]`
	- Non è disponibile `push_back()` – è il set che decide dove inserire l'elemento
	- Gestito tramite le operazioni tipiche delle liste -> `insert() ed erase()`
	- Set è un contenitore ordinato
	- Deve essere definita una funzione d'ordine per ciascun tipo contenuto

#### Esempio
```c++
struct Fruit{ 
	string name; 
	int count; 
	double unit_price; 
	Date last_sale_date; 
	// ... 
};
```

È necessario definire una funzione d'ordine
```c++
struct Fruit_order { 
	bool operator()(const Fruit& a, const Fruit& b) const { 
		return a.name < b.name; 
		} 
};
```

È ora possibile creare:
```c++
set<Fruit, Fruit_order> inventory; 
// Nota: Fruit_order serve per confrontare Fruit
```

È possibile popolare il set così:
```c++
inventory.insert(Fruit{"Quince", 5}); 
inventory.insert(Fruit{"Apple", 200, 0.37});
```

E scorrerlo / stamparlo così:

```c++
for(auto p = inventory.begin(); p != inventory.end(); ++p) 
	cout << *p << '\n';
```

Map non contiene pair, si accede al dato direttamente con *