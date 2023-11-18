# Gestione della memoria e puntatori

La gestione della memoria è spesso considerata un compito di basso livello, spesso affidato alle librerie. 
Tuttavia, è un'area fondamentale e istruttiva da esplorare. Studiare come utilizzare la memoria ci consente di comprendere meglio il funzionamento delle librerie, evitando di affidarci ciecamente a una sorta di "magia". 
Essere in grado di implementare nuovi contenitori richiede una conoscenza approfondita di come la memoria viene gestita e utilizzata nei nostri programmi.

### Puntatori
- Particolare tipo di variabile
- Contiene un *indirizzo di memoria*
```c++
ptr: 0x1001054a0
```

- L'uso dell'indirizzo di memoria consente di accedere al *dato puntato*.

### Definizione di puntatore
- Un puntatore punta a un tipo
	- È noto il tipo di dato che si trova all'indirizzo di memoria contenuto nel puntatore
- La definizione contiene
	- Il tipo puntato
	- Il carattere * che significa che la variabile definita è un puntatore
```c++
double d; // variabile double 
double* p; // variabile puntatore a double
```

Modi per scrivere più punatatori:
```c++
double *p1, *p2; // due puntatori a double 
double *p3, d1; // un puntatore a double e un double 
double* p4, d2; // un puntatore a double e un double 
double * p5, d3; // un puntatore a double e un double
```

La spaziatura non influisce sulla semantica, ma può essere controintuitiva (es., terza riga)

#### Esempio di utilizzo
Supponendo di avere un intero var e un puntatore intero ptr, posso accedere all'indirizzo della variabile scrivendo:
```c++
int var = 17;
int* ptr = &var;
//il puntatore punta all'indirizzo di memoria di var
```

### Tipi di puntatori
- Ogni tipo ha il suo puntatore.
- Un puntatore ha un contenuto esprimibile con un numero intero (l’indirizzo di memoria), ma è un tipo a sé. Tale tipo supporta operazioni tra indirizzi.

- Due puntatori a tipi diversi sono essi stessi tipi diversi
```c++
int x = 17; 
int* pi = &x; 
char* pc = pi; // errore, non si può assegnare int* a char* 
pi = pc; // errore, non si può assegnare char* a int*
```

- Ciò è necessario perché è diverso ciò che posso fare col dato puntato.
#### Attenzione
Supponiamo si possa fare la seguente operazione
```c++
char ch1 = 'a'; 
char ch2 = 'b'; 
char ch3 = 'c'; 
char ch4 = 'd'; 
int* pi = &ch3; // anche se potessi, cosa accadrebbe 
				// nelle righe seguenti? 
*pi = 12345; 
*pi = 67890;
```

- Con * pi = 12345 si cambiano i valori anche di ch 3 e ch 4.
- Nel caso più sfortunato andremo a sovrascrivere anche pi
- Nel assegnamento successivo * pi=67890, il valore viene scritto in una locazione non definite della memoria
### Accedere al dato puntato
- Il dato al puntatore è passato per riferimento
- Per accedere al dato puntato utilizzo l'operatore di _dereference *_
	- Restituisce un *lvalue*
	- Posso assegnare un puntatore *int* ad un puntatore *double*
```c++
int x = 17; 
int* pi = &x; 
double e = 2.71828; 
double* pd = &e;

//in seguito

cout << "pi == " << pi << ", content of pi == " << *pi << '\n’; 
cout << "pd == " << pd << ", content of pd == " << *pd << '\n';
```

### Nullpointer e sizeof ()
Comodo avere un valore non valido per segnalare un puntatore non inizializzato
```c++
double* p0 = nullptr;
```

Fa parte di C++11, sennò si usa o *0* o *null*.

In questo contesto è molto utile *sizeof()*.
- Funziona con un nome di tipo o di variabile/espressione
```c++
void sizes(char ch, int i, int* p) { 
	cout << "The size of char is " << sizeof(char) << ' ' << sizeof(ch) << '\n';
	 
	cout << "The size of int is " << sizeof(int) << ' ' << sizeof(i) << '\n'; 
	
	cout << "The size of int* is " << sizeof(int*) << ' ' << sizeof(p) << '\n'; 
}
```

### In sintesi
Creato un puntatore tipo* p:
- Usi l'indirizzo del puntatore con p
- Usi il contenuto del puntatore con * p

---

# Puntatori, reference, cast

### Void* e cast
I puntatori hanno un minimo check sul tipo. **Void*** permette di saltare qualsiasi controllo. 
- Void* rappresenta il concetto puro di "indirizzo di memoria" senza indicazioni su come usarla
- È un puntatore a memoria raw
- Devo fornire indicazioni su come deve essere usata la memoria puntata
Occhio alla differenza: void e void*.
- È possibile assegnare qualsiasi puntatore a un void*

```c++
int v1 = 7; 
double v2 = 3.14; 
void* pv1 = &v1; 
void* pv2 = &v2;

void f(void* pv) 
{ 
	void* pv2 = pv; // ok 
	double* pd = pv; // no! Conversione tra tipi 
					// incompatibili 
	*pv = 7 // no! non posso dereferenziare 
			// (che oggetto è?) 
	pv[2] = 7; // no! Stessa ragione 
	
	int* pi = static_cast<int*> (pv); // ok, conversione 
									// esplicita }
```

### Cast
Si tratta di conversione esplicita tra i tipi (inclusi i puntatori): *static_cast*
- Check avviene al tempo di compilazione, nessun check run-time
Due strumenti di conversione "*potentially even nastier*", da usare consapevolmente:
- **reinterpret_cast**: può fare il cast fra tipi totalmente indipendenti, es.: int e double* 
- **const_cast**: elimina const

In sostanda dice al compilatore: fidati di me, so quello che faccio! DA STARE ATTENTI

#### I pochi casi in cui è giusto farlo
I cast servono principalmente:
- A interfacciarsi con HW
- O altro codice non modificabile

``` c++
Register* in = reinterpret_cast<Register*>(0xff); 
//Dice al compilatore che questa area di memoria deve essere interpretata come un Register

void f(const Buffer* p)
{ 
	Buffer* b = const_cast<Buffer*>(p); 
	//Necessario perché la libreria (di terze parti) che sto usando mi fornisce solo un const Buffer* che però io voglio modificare
	// ... 
}
```
Regole d'oro:
- Prima di utilizzare un cast, riguardare il codice e vedere se si può fare a meno
- Se si deve utilizzare, meglio lo static_cast

## Puntatori e references
Una reference è come un puntatore:
- Immutabile
- Dereferenziato automaticamente