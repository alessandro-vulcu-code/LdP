# Eccezioni
- Accertarsi che il dato immesso sia corretto
- Posso creare ad esempio la mia classe `Invalid()`
- Potenziali problemi?

- La classe Invalid è stata creata per segnalare una eccezione
	- Non è standard
	- Eventuali catch devono conoscere l'esistenza di `Invalid()`
- La libreria standard contiene una classe base che descrive il concetto di eccezione
- Tutte le altre derivano da std::exception

Due famiglie di eccezioni
- **std::logic_error**: riporta un errore che consegue una logica difettata come la violazione di pre-condizioni o gli invarianti della classe
- **std::runtime_error**: riporta errori dovuti a eventi esterni al programma stesso che sono difficilmente prevedibili
![[Pasted image 20240116133357.png]]

### Classe Exception
```c++
class exception { 
	public: 
		exception(); 
		exception(const exception&); 
		exception& operator=(const exception&); 
		virtual ~exception(); 
		virtual const char* what() const; 
};
```

- Costruttore e assegnamento di copia disponibili
- Funzione what () per ottenere una stringa di descrizione

- Tipico pattern:
```c++
// ... 
try { 
	// ... 
	// ... 
} catch (std::exception& e) { 
		std::cerr << e.what() << std::endl; 
} 

// ...
```

- È anche possibile creare eccezioni derivate. Le eccezioni derivate permettono di personalizzare `what()`.
```c++
class My_error : runtime_error { public: My_error(int x) : error_value{x} {} int error_value; const char* what() const override { return "My_error"; } };
```

---

## Risorse ed eccezioni
- In passato abbiamo analizzato la gestione delle risorse
- Memoria
	- Acquisizione 
	- Utilizzo 
	- Rilascio
- La gestione delle risorse può essere corrotta dalle eccezioni

**Esempio**
- `std:: vector` può accedere alla memoria in due modalità
	- Con boundary check – funzione `at()`
	- Senza boundary check – `operator[]`
- <span style="color:#00b0f0">Nel primo caso: se i vincoli non sono soddisfatti, che fare?</span>
	- Eccezioni

- È normale che `std::vector` lanci eccezioni
- In questo contesto, dobbiamo assicurarci che eventuali risorse occupate siano liberate

### Gestione delle risorse
```c++
void suspicious(int s, int x) { 
	int* p = new int[s]; 
	// ... 
	delete[] p; 
}
```
- Come possiamo essere sicuri che la memoria sia rilasciata?
- Dipende, se ci metto un return prima di delete non ci arrivo a liberare la memoria

- Sul pdf ci sono vari esempi con codice

- Posso gestire o la eccezione o la gestione delle memorie:
	- Nel primo caso, uso ad esempio un `try-catch` quando c'è un eccezione e pulisco così la memoria se viene interrotto
	- Nel secondo caso, uso solo standard `vector`, eliminando l'allocazione dinamica, perché il compilatore alla fine elimina automaticamente lo spazio allocato
## RAII
- Occupare le risorse nel costruttore e liberarle nel distruttore risolve questi problemi-> prende il nome di **Resource Acquisition Is Initialization (RAII)**
- È un'idea generale.
- Il meccanismo visto prima si basa sull'uscita dallo scope
	- Uscita dallo scope libera le risorse mediante chiamata ai distruttori
	-  permette di ottenere un buon design nella maggior parte dei casi
- A volte desideriamo "far uscire" gli oggetti dallo scope
	- Es: funzione che costruisce un oggetto grande e lo ritorna usando un puntatore
```c++
vector<int>* make_vec() { 
	vector<int>* p = new vector<int>; 
	// ... riempimento del vettore con i dati – può 
	// ritornare un'eccezione! 
	return p; 
}
```

- A volte desideriamo "far uscire" gli oggetti dallo scope
	- Caso tipico: funzione che costruisce un oggetto grande e lo ritorna usando un puntatore

### RAII e scope
- Allocare dinamicamente uno `std::vector` è solitamente cattiva pratica
	- Usato come esempio perché il riempimento può causare un'eccezione
```c++
vector<int>* make_vec() { 
	vector<int>* p = new vector<int>; 
	// ... riempimento del vettore con i dati – può 
	// ritornare un'eccezione! 
	return p; 
}
```

- <span style="color:#00b0f0"> Cosa succede se un'eccezione è lanciata durante il riempimento?</span>
	- Caveat: p deve essere liberato dal chiamante

- La funzione make_vec potrebbe dover gestire un'eccezione
- Comportamento desiderato
	- Se non sono lanciate eccezioni, `make_vec` restituisce il puntatore
	- Se sono lanciate eccezioni, `return` non è eseguito, ma `make_vec` non deve comunque causare memory leak

- Implementazione del comportamento
```c++
vector<int>* make_vec() { 
	vector<int>* p = new vector<int>; 
	try { 
	// ... riempimento del vettore con i dati – può 
	// ritornare un'eccezione! 
	} catch(...) { 
		delete p; 
		throw; // rilancia l'eccezione 
	} return p; 
}
```

### Garanzie
- La tecnica vista è un pattern ricorrente che prende il nome di **basic guarantee**
- Basic guarantee: il blocco try/catch fa sì che make_vec () funziona, oppure lancia un'eccezione e non crea leak
	- **Strong guarantee**: la funzione rispetta la basic guarantee, e in più tutti i valori osservabili (valori non locali alle funzioni) sono gli stessi che erano presenti prima della chiamata in caso di fallimento.
		- Commit or rollback
	- **No-throw guarantee**: la funzione non lancia eccezioni
- <span style="color:#00b0f0">Che garanzia offre</span> `make_vec()` <span style="color:#00b0f0">?</span>
	- Sicuramente la basic
	- Anche la strong, se non ci sono istruzioni strane nel riempimento del vettore

Usiamo i smart pointer

---
## Smart pointer
- Puntatori intelligenti
- Classi template definite in `<memory>`
- Gestiscono automaticamente la deallocazione della memoria quando escono dallo scope
	- Eliminano memory leak
	- Eliminano dangling pointer
- Un overhead è presente, ma molto limitato rispetto a un garbage collector

- Esistono vari tipi di smart pointer
	- Unique_ptr
	- Shared_ptr
	- ~~Auto_ptr (deprecated)~~
- Diversi per alcune caratteristiche fondamentali che ne regolano l'uso
- Si è soliti dire che uno smart pointer detiene un puntatore

### Unique_ptr 
Unique_ptr è uno smart pointer che:
- Dealloca la memoria uscendo dallo scope
- Permette i move
- **Non permette le copie** -> Costruttore e assegnamento di copia disabilitati
- Non ha sostanziali overhead rispetto a un puntatore

- Inizializzazione 
	- Nel costruttore, oppure: 
	- Funzione reset
- Re-inizializzazione: funzione reset
	- Fa sì che `unique_ptr` detenga un nuovo puntatore 
	- Se `unique_ptr` deteneva un puntatore prima della chiamata, l'oggetto puntato è distrutto 
	- Senza argomenti: semplice deallocazione
#### Uso e rilascio 
- Accesso
	- È possibile usare gli operatori * e -> come se fosse un puntatore

- Liberazione automatica della memoria
	- Quando unique_ptr è distrutto, cancella l'oggetto puntato!

- Rilascio del puntatore
	- `Release()` estrae il puntatore dallo unique_ptr, il quale è resettato a nullptr

```c++
vector<int>* make_vec() { 
	unique_ptr<vector<int>> p { new vector<int> }; 
	// ... riempimento del vettore con i dati – può 
	// lanciare un'eccezione! 
	return p.release(); 
}
```

- Inizializzazione nel costruttore
- Release del puntatore:
	- A fine utilizzo
	- Quando è lanciata un'eccezione
		- Quindi in make_vec **non è necessario fare il catch**!
	- Release () estrae il puntatore dallo unique_ptr, il quale è resettato a nullptr
		- Quindi la successiva distruzione di p non cancella nulla
		- Si perde la deallocazione automatica

### Copia e spostamento
- Non può essere copiato ne spostato
- È possibile ritornare uno unique_ptr -> sarà usato il move constructor
```c++
unique_ptr<int> AllocateAndReturn() { 
	unique_ptr<int> ptr_to_return(new int(42)); 
	return ptr_to_return; 
}
```

- La copia è invece disabilitata
- Inizializzare due unique_ptr con lo stesso puntatore è un errore
	- Doppia delete
	- Non rilevabile dal compilatore

- L'esistenza del puntatore da incapsulare è un punto debole
	- Risolvibile con make_unique

- Lo shared pointer rimuove i vincoli di unique_ptr
- Uno shared_ptr
	- Può essere copiato
	- Può essere condiviso – molti shared_ptr possono detenere lo stesso puntatore