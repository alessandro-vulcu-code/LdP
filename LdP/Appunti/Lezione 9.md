# Template
Vediamo cosa sono i template. Scriverne uno non è sempre facile.
Si tratta di un meccanismo che consente al programmatore di usare un tipo come parametro per una classe o una funzione.

Il compilatore genera il codice necessario per ogni tipo per cui il template è specializzato

```c++
std::vector<double> 
std::vector<int> 
std::vector<Month> 
std::vector<char> 
std::vector<std::vector<Record>> 
std::vector<Window*>
```

Lo standard vector è un *class template*. Un class template viene anche chiamato *type generator*.
Il processo di generazione di una classe su un tipo specifico è chiamato specializzazione (specialization) o template instantiation. 
Non è completamente definita, è definita a meno di un parametro. Infatti tipo `std::vector<char>` esiste solo se definisco il tipo della classe. Si parla di *specializzazione* o template installation.

Questo processo avviene a tempo di compilazione:
- In esecuzione è più veloce, più efficenza
- Vengono rilevati gli errori già al tempo di compilazione
- Però la compilazione non è più istantanea

### Notazione
```c++
template <typename T> oppure template <class T
```

Sono equivalenti. 
Typename:
- Più chiaro
- Evidente che possiamo  usare tipi built-in
Class:
- Più corto
- Class significa "tipo"

### Template per vector
Riprendiamo ora la classe vector sviluppata precedentemente. Sostituire un tipo generico T a double nella nostra classe vector. Notazione C++: prefisso che introduce il template.
```c++
template<typename T> 
class vector { 
	int sz; 
	T* elem; 
public: 
	vector() : sz{0}, elem{nullptr} {} 
};
```

Ora possiamo usate T come tipo definito. Quando la classe verrà specializzata a int avrò un puntatore a int, ad esempio.

```c++
template<typename T> 
class vector { 
	int sz; T* elem; 
public: 
	vector() : sz{0}, elem{nullptr} {} 
	
	explicit vector(int s) : sz{s}, elem{ new T[s] } { 
		for (int i = 0; i < sz; ++i) elem[i] = 0; 
	} 
};
```

#### Funzione push back
- È usata la funzione membro push_back () su v
- Funzione definita come:
```c++
template<typename T> 
void vector<T>::push_back(const T& d) {/* ... */}
```

- A partire da questa, il compilatore genera:
```c++
void vector<std::string>::push_back(const std::string& d) {/* ... */}
```

### Funzioni template
Quando parametrizziamo una funzione otteniamo un function template, detto anche *parametrized function* o *algorithm*

Esempio:
```c++
template <typename T> 
T myMax(T x, T y) 
{ 
	return (x > y) ? x : y; 
}
```

Perché è necessario usare un template in questo caso? 
- Se ho più classi che utilizzano il mio template allora è ottimale perché poi lascio al compilatore la specializzazione.

Che assunzione stiamo facendo su T?
- Bisogna garantire che siano confrontabili, lo garantisco implementando l'operator<< o >>. 

E se non è comparabile?
- Errore di compilazione

Per chiamare la funzione scritta sopra:
```c++
int x = i, j = 5; 
int max = myMax<int>(i, j)
```

## Interi come parametri template
Cosa succede se usiamo un intero come parametro template? Per esempio: un vector con template su tipo e numero

Ipotesi: il numero di elementi è noto a tempo di compilazione, quindi passiamo la dimensione come template

```c++
template<typename T, int N> 
struct array { 
	T elem[N]; 
	
	T& operator[] (int n); 
	const T& operator[] (int n) const; 
	
	T* data() { return elem; } 
	const T* data() const { return elem; } 
	
	int size() const { return N; } 
};
```

Questi array sono simili ai precedenti, ma sono molto meno flessibili. Allora perché usarli?
- Ragione principale: efficienza
	- Il compilatore può ottimizzare meglio
- Non necessita di free store
	- Alcuni programmi non possono usarlo
- Offrono, però, gli stessi vantaggi di interfaccia e di sicurezza dei vector

- Un array di dimensione non modificabile può esprimere un concetto
	- Un array di 3 elementi fissi esprime il concetto di "tripletta"
	- Es: le immagini a colori sono composte da triplette di valori (R, G, B)
	- Esprimibili come array<unsigned char, 3>
### Deduzione degli argomenti template
Il compilatore è in grado di dedurre gli argomenti template dagli argomenti di una funzione.
Ad esempio si consideri:
```c++
array<char, 1024> buf; 
array<double, 10> b2; 

template<class T, int N> 
void fill(array<T, N>& b, const T& val) { 
	for (int i = 0; i < N; ++i) b[i] = val; 
}
```

Le chiamate a `fill` sono interpretate così:
```c++
void f() 
{ 
	fill(buf, 'x’); // T è char e N è 1024, dedotto da buf 
	fill(b2, 0.0); // T è double e N è 10, dedotto da b2 }
```

Equivalenti a:
```c++
void f() 
{ 
	fill<char, 1024>(buf, 'x'); 
	fill<double, 10>(b2, 0.0); 
}
```

Analogamente per l'esempio precedente:
```c++
int x = i, j = 5; 
int max = myMax<int>(i, j)
```

Equivalenti a:
```c++
int x = i, j = 5; 
int max = myMax(i, j)
```

---

# Polimorfismo
### Definizione
- Software che assume più forme

La notazione del polimorfismo varia da autore ad autore.
Noi usiamo la notazione di Vandevoorde/Josuttis

Polimorfismo: abilità di associare comportamenti specifici diversi a un'unica notazione
#### Polimorfismo dinamico
- Implementato tramite ereditarietà e funzioni virtuali
- A tempo di esecuzione (run-time)

#### Polimorfismo statico
- Implementato tramite template
- A tempo di compilazione (compile-time)

### Programmazione generica
Il polimorfismo statico è collegato al concetto di programmazione generica:
- Visione semplicistica: programmazione generica in C++ equivale a usare i template
- Ma così confondiamo un concetto con uno strumento del linguaggio

Più correttamente: programmazione generica significa scrivere codice che funzioni con tipi diversi forniti come argomenti, posto che tali tipi soddisfino determinati requisiti sintattici e semantici. Lo strumento per farlo è il template.

---

# Template e header
Gli header hanno una sostanziale modifica con i template
I template combinano flessibilità e efficienza, ma effetti collaterali:
- Poca separazione tra dichiarazione e definizione
- Errori di compilazione poco comprensibili 
- Il compilatore controlla sia il template che il suo argomento

### Header con i template
Spesso i compilatori richiedono che i template siano completamente definiti prima di essere usati
- Le definizioni son spostate negli header
Non è imposto dallo standard, ma dallo specifico compilatore

<span style="color:#00b0f0">Come mantenere la separazione tra interfaccia e implementazione con i template?</span>

L'header viene suddiviso in due blocchi:
- **File .h**: tipico contenuto da header
	- Definizioni di classi
	- Dichiarazioni di funzioni
- **File .hpp**: tipico contenuto da .cpp
	- Definizioni e funzioni
I blocchi sono combinati con `#include` atipico, perché lo mettiamo in fondo.

vector.h
```c++
#ifndef vector_h 
#define vector_h 

template<typename T> 
	class vector{ //dichiarazione
// ... 
		void resize(int newsize); 
// ... 

}; 

// ... 

#include "vector.hpp" // inclusione dopo la dichiarazione 
#endif // vector_h
```

vector.hpp
```c++
#ifndef vector_hpp 
#define vector_hpp 

// ... 

template <typename T> 
void vector<T>::resize(int newsize) 
{ 
	// ... 
} 

// ... 

#endif // vector_hpp
```

---

# Controllo di versione
### Evoluzione del software
Lo sviluppatore singolo:
- Aggiunge nuove funzionalità
- Modifica funzionalità esistenti
- Eliminazione delle funzionalità

Sviluppatori multipli:
- Creazioni di diversi moduli in parallelo
- Modifiche nello stesso modulo in punti diversi
- Modifiche nello stesso modulo nello stesso punto (conflitto) 
- Merge delle modifiche 
- Utilizzo di software scritto da altri sviluppatori

### Tracciamento del software?
Perché tenere traccia delle modifiche?
- Salvare vari "punti di ripristino" (cit.) stabili 
- Rimozione di modifiche stabili ma non più desiderate 
- Evoluzione non sempre lineare (rami) che devono essere riuniti 
- Condivisione del codice e delle modifiche

Che attività richiede?
Base:
- Salvare le varie versioni 
- Evidenziare le modifiche tra una versione e l’altra 
- Distribuire le modifiche a tutti gli sviluppatori

Avanzati
- Suddivisione dello sviluppo software in rami (branches) 
- Condivisione delle modifiche tra i vari rami (branches) 
- Merge intelligente delle modifiche di sviluppatori diversi 
- Tool di lettura della storia passata

#### Soluzione 1: salvataggio di copie
- Ogni salvataggio contiene una copia completa
- Calcolo delle modifiche con diff/meld 
- Applicazione manuale delle modifiche di uno sviluppatore sul codice degli altri sviluppatori
- Facile tornare indietro, non andare avanti ☺ 
- Poco versatile, molto overhead, non fornisce gli strumenti avanzati

#### Soluzione 2: software di controllo versione
- Implementa in maniera automatica gli strumenti che abbiamo visto
- SCM software configuration managemento: "Is the task of controlling changes in the software. Configuration management practices include revision control"
- Revision control – version control – source control

### Controllo di versione
Diverse soluzioni:
- Diff/meld (manuale) 
- CVS/SVN (controllo di versione centralizzato) 
- GIT/Mercurial/Bazaar (controllo di versione distribuito)

### Modello centralizzato (es SVN)
Il server:
- Contiene tutta la storia e i rami 
- Riceve le modifiche dai client

Il client:
- Il client scarica una versione 
- Modifiche in locale 
- Upload delle modifiche

La stratificazione del codice avviene mediante l’upload sul server di una nuova versione

### Modello distribuito (Git)
- Scompare la distinzione tra client e server 
- Esistono solo alberi che sono pari (repository) 
- Tutta la storia passata è presente in locale 
- Stratificazione locale 
- Sincronizzazione tra gli alberi