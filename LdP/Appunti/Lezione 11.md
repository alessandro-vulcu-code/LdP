# Standard Template Library (STL)

Parte di libreria standard che riguarda i template, ad esempio poter dichiarare vari tipi di vettori. 
Per gestire sequenze di dati possono esserci numerose operazioni.

La **Standard Template Library** è una parte della libreria standard che fornisce:
- Contenitori
	- std::vector
	- std::list
	- Std::map

- Algoritmi generici
	- std:: sort 
	- std:: find 
	- std::accumulate
Differenzia struttura dati dall'algoritmo

STL ha vari tipi di contenitori, ma hanno accesso unificato cioè la modifica del tipo di un contenitore non ha impatto sull'algoritmo.
Ci permettono di scrivere codice aggiuntivo solo quando vogliamo implementare funzioni aggiuntive.

## Sequenze e iteratori 

**Sequenza**: insieme di oggetti che hanno inizio e una fine
**Iteratore**: oggetto che all'interno di una sequenza mi identifica un elemento della sequenza.

L'iteratore ricorda un puntatore, concettualmente
- Begin: primo elemento della sequenza
- End: primo elemento dopo l'ultimo

Un iteratore punta a un elemento della sequenza.
È possibile confrontare due iteratori con operatori: == e !=.
È possibile riferirsi agli oggetti puntati usando l'operatore unario * (dereference)
È possibile ottenere l'elemento successivo usando ++.

Grazie agli iteratori abbiamo una *sequenza standard*, grazie all'interfaccia fornita da STL.

## Separare algoritmi e collezioni di dati

**Disaccoppiamento dell'algoritmo**: non è necessario conoscere i dettagli della struttura dati; è sufficiente usare i relativi iteratori.
- Es: un algoritmo di ordinamento può operare pur senza conoscere i dettagli della struttura dati

**Disaccoppiamento della struttura dati**: non è necessario esporre tutti i dettagli e gestire tutti i modi d'uso; è sufficiente implementare ed esporre gli iteratori.
- Es: uno std:: vector non ha necessità di conoscere l'algoritmo che opera su di esso per permettergli di funzionare; è sufficiente che esponga gli iteratori

### Esempio
```c++
// Restituisce l'elemento in [first, last) che ha il valore più alto 
template<typename Iterator> 
Iterator high(Iterator first, Iterator last) { 
	Iterator high = first; 
	for (Iterator p = first; p != last; ++p) 
		if (*high < *p) 
			high = p; 
	return high;
}
```

---

# linked list e il suo iteratore

## std:: vector come sequenza
Nel caso di std:: vector, gli elementi sono consecutivi in memoria 
Dall’element 0 all’elemento size-1
La nozione STL di sequenza non prevede che gli elementi siano consecutivi 
La struttura dati più vicina a una sequenza STL è:
![[Pasted image 20240116133019.png]]

```c++
template<typename Elem> struct Link { 
	Link* prev; // Link precedente 
	Link* succ; // Link successivo 
	Elem val; // Valore 
}; 
template<typename Elem> 
struct list { 
	Link<Elem>* first; 
	Link<Elem>* last; 
};
```

### Operazioni
Dobbiamo progettare le operazioni che sarà possibile effettuare su una list
- Quelle di vector (senza []) – costruttore, size, ...
- `Insert` ed `erase`
- Iteratori per attraversare la lista
Escludo `[]` perché non ha senso, lista non sequenziale

```c++
template <typename Elem>
class list

{ // rappresentazione e dettagli implementativi
public:
    class iterator;
    iterator begin();
    iterator end();
    
    iterator insert(iterator p, const Elem &v);
    iterator erase(iterator p);
    void push_back(const Elem &v);
    void push_front(const Elem &v);
    void pop_front();
    void pop_back();
    Elem &front();
    Elem &back(); // ...
};
```

Dopo aver progettato `list`, dobbiamo progettare il suo iteratore
- Deve contenere un puntatore a un elemento della lista
- Deve implementare i seguenti operatori: `++, --, *, ==, !=`

## Iteratore di lista
```c++
template <typename Elem>
class list<Elem>::iterator
{
    Link<Elem> *curr;
  
public:
    iterator(Link<Elem> *p) : curr{p} {}
    iterator &operator++()
    {
        curr = curr->succ;
        return *this;
    }
    iterator &operator--()
    {
        curr = curr->prev;
        return *this;
    }
    Elem &operator*() { return curr->val; }
    bool operator==(const iterator &b) const { return curr == b.curr; }
    bool operator!=(const iterator &b) const { return curr != b.curr; }
};
```

Se la lista è vuota, `begin == end`, condizione da verificare
**End che punta a un elemento dopo la fine permette di trattare la lista vuota come un caso non speciale**

## Iteratori e relativo codice
- Gli iteratori rendono il codice più complesso. Scorro l'iteratore
```c++
template <typename T>
void user(vector<T> &v, list<T> &lst)
{
    for (vector<T>::iterator p = v.begin(); p != v.end(); ++p)
    {
        cout << *p << '\n';
    }
    list<T>::iterator q = find(lst.begin(), lst.end(), T{42});
}
```
- `Find` funziona con varie strutture dati perché accetta iteratori, che sono standard.

- Molto spesso il tipo dell'iteratore può essere dedotto dal compilatore

Posso trasformare tutto quel casino con **auto**. Va usato con cautela. Infatti così posso usare qualsiasi tipo.
```c++
template <typename T>
void user(vector<T> &v, list<T> &lst)
{
    for (auto p = v.begin(); p != v.end(); ++p)
    {
        cout << *p << '\n';
    }
    auto q = find(lst.begin(), lst.end(); T{42});
}
```

Usare solo per fare gli iteratori.

---

# std:: vector, std:: list, std::string

Vector:
- accetta tutte le operazioni, inclusi `insert()` e `erase()`
- Fornisce `[]`
- `Insert()` ed `erase()` sono inefficienti
	- Devono muovere molti elementi in memoria
	- Un problema per collezioni di dati molto grandi
- Fornisce range check (su richiesta!) 
- Espandibile 
- Elementi contigui in memoria 
- Funzioni di confronto confrontano gli elementi

String:
- Come i vettori, ma aggiungono le operazioni di manipolazione del testo
- Concatenazione: `+, +=`
- Gli operatori di confronto confrontano gli elementi

`Std:: string vs. Char[]`: si noti la differenza tra `std:: string` e `char[]`
- `Char[]` è un array stile C, quindi ha molte cose in meno

List:
- Fornisce le operazioni abituali, ma non subscripting – `[]` 
- Possiamo usare `insert()` e `erase()` senza spostare gli altri elementi, sono efficenti
- Espandibile 
- Sono definiti gli operatori di confronto, che confrontano gli elementi

## Insert() e erase() con std::vector

- Effettuare una insert () o una erase () in un std:: vector può essere un'operazione inefficiente
- Muovere gli elementi ha una forte implicazione: rende non validi gli iteratori che stanno puntando al std::vector

```c++
std::vector<int>::iterator p = v.begin(); // puntatore al primo // elemento del vettore 
++p; ++p; ++p; // puntatore al quarto // elemento 
auto q = p; ++q; // puntatore al quinto // elemento
```
![[Pasted image 20240116133114.png]]

```c++
p = v.insert(p, 99);
```

![[Pasted image 20240116133128.png]]

<span style="color:#00b0f0">Si ripropone lo stesso problema con le liste? Perché?</span>
No, non si ripropone lo stesso problema, la lista incrementa il numero di elementi, il puntatore rimane sempre sullo stesso luogo in memoria, non sposto nulla.
