Reference:
[[L05.1 - Puntatori e gestione della memoria.pdf]]
[[L05.2 - Puntatori, references e cast.pdf]]
[[L05.3 - Array e artimetica dei puntatori.pdf]]
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
Oppure come un nome alternativo per un oggetto.

### Confronto
*Puntatori*:
1) Assegnamento: cambia il valore del puntatore, non dell'oggetto puntato
2) Per cambiare il valore dell’oggetto puntato: *deference*

```c++
int x1 = 10; 

int* p1 = &x1; 

p1 = 7; // sbagliato e // pericoloso!! 

*p1 = 7; // corretto
```

3) Per accedere all'oggetto puntato: _*_ oppure *[]*
```c++
*p1 = 4;
```

4) Assegnamento: copia del puntatore, non dell'oggetto puntato (**shallow copy**)
```c++
int i, j; 

int *p1 = &i; 

int *p2 = &j; 

p2 = p1; // copia del // puntatore
```

5) Esiste il null pointer

*Reference*:
1) Assegnamento: cambio valore dell'oggetto
```c++
int y1 = 10; 

int& r1 = y1; 

r1 = 7;
```

2)  Nessun operatore per accedere al dato puntato
```c++
r1 = 4;
```

3) Assegnamento: copia dell'oggetto a cui si riferisce (**deep copy**)
```c++
int i, j; 

int &r1 = i; 

int &r2 = j; 

r2 = r1; // copia del contenuto
```

4) Non esiste una reference non valida

## Parametri puntatori e reference
Sappiamo che facendo una chiamata a funzione passando un parametro reference, non creo una nuova copia di quell'oggetto ma passo il riferimento di quella variabile. 

Con i puntatori posso ottenere lo stesso effetto, volendo si può cambiare il valore di una variabile in tre modi:

```c++
int incr_v(int x) { return x + 1; } 

void incr_p(int* p) { ++*p; } 

void incr_r(int& r) { ++r; }
```

Quale scegliere? Come al solito dipende:
- Il ritorno del valore è:
	- Più chiaro e meno soggetto ad errori
	- Va bene per oggetti piccoli
	- Oppure per oggetti grandi se hanno il move constructor

Reference vs. Puntatore
- I puntatori sono espliciti
- I puntatori possono aver valore nullptr, le reference no
- Un criterio:
	- Se no-object è un valore plausibile: puntatore
	- Altrimenti: reference/const reference

---

# Array
Definzione: sequenza omogenea di oggetti allocati in spazi di memoria contigui.
- Stesso tipo, e nessuno spazio vuoto tra elementi
- Indicizzati con []
- Accesso casuale
- Nessun controllo su lettura/scrittura fuori range, quindi facile fare errori difficili da trovare

Gli std:: vector sono vettori dinamici, intelligenti, di alto livello. Gli array "stile C" sono strutture dati più semplici, più antiche. Sono necessari in alcune circostanze (più avanti nel corso).

### Dimensione di un array
- La dimensione di un array è fissa, una volta definito.
	- Per modificarne la lunghezza va creato un array più lungo in cui copio i dati dentro
- La dimensione di un array:
	- Deve essere una costante (literal o const) conosciuta a tempo di compilazione
- Esistono anche VLA (Variable Length Array)
	- Fanno parte dello standard C 99
	- Non sono standard C++
	- GCC li accetta

Un array può essere insaziato come variabile:
- Variabili globali
- Variabili locali
- Ma attenzione alla memoria nello stack!

Gli argomenti di funzioni vengono *trasformati in puntatori*

Esempio di codice
```c++
const int max = 100; int gai[max]; 
// array globale, sempre disponibile 

void f(int n) 
{ 
	char lac[20]; // array locale: vive fino all'uscita 
					// dallo scope 
	int lai[60]; 
	
	double lad[n]; // errore: dimensione non costante 
}
```

## Puntatori a elementi di un array
Possiamo definire puntatori a elementi di un array
```c++
double ad[10]; 
double* p = &ad[5];
```
![[puntatore1.png]]

Possiamo usare subscript e dereference su p
```c++
double ad[10]; 
double* p = &ad[5]; 

*p = 7; 
p[2] = 6; 
p[-3] = 9;
```
![[puntatore2.png]]

## Aritmetica dei puntatori
- I puntatori sono tipi che supportano la somma e la sottrazione con interi
- Sommare o sottrarre un intero N da un puntatore significa spostare il puntatore di N slot a destra o a sinistra
- Operatori: +, -, +=, -=
- Lo slot dipende dal dato puntato
- L'aritmetica è sensibile al contesto
#### Esempio
```c++
double* q = pd;
q = pd + 2;
```
![[puntatore ad array.png]]

```c++
*p = 7; 
p += 2;
```
![[puntatore ad array2.png]]

## Array e puntatori

### Decadimento di un array
- Il nome di un array è un puntatore const al suo primo elemento
- È un rvalue! Cioè un valore, non una variabile
- Non possiamo usarlo per la copia
```c++
int x[100]; 
int y[100]; 
// ... 
x = y;          // errore 
int z[100] = y; // errore
```

- Passare un array a una funzione significa passare il puntatore, perciò si ha una perdita di informazione sulla dimensione dell'array

Lo vediamo con un esempio sulle stringe (C-style)
```c++
int strlen(const char a[]) 
{ 
	int count = 0; 
	while (a[count]) 
	{ 
		++count; 
	} 
	return count; 
} 

char lots[100000]; 

void f() 
{ 
	int nchar = strlen(lots); 
	// ... 
}
```

- Strlen (lots) è considerato equivalente a strlen (&lots[0])
- Il decadimento serve per evitare copie non desiderate
- Un comportamento un po' antiquato che deriva da C, C++ lo mantiene per retrocompatibilità

```c++
char ac[] = "Beorn"; //stringa stile c 
char* pc = "Howdy"; //puntatore alla prima poszione dell'array 
					//cioé 'H'
```

### Inizializzazione di un array
Ecco alcuni modi per inizializzare un array
```c++
int ai[] = { 1, 2, 3, 4, 5, 6 }; // dimensione dedotta: 6 
int ai2[100] = { 1, 2, 3, 4, 5, 6, 7, 8, 9 }; // gli altri 
										// inizializzati a 0 
double ad[100] = {}; // tutti inizializzati a 0.0 
char chars[] = { 'a', 'b', 'c' }; // nessuno 0 terminatore
```

### Problemi legati agli array
Alcuni problemi classici:
- Accesso usando un puntatore non inizializzato (In particolare: puntatori membro)
```c++
int* p; 
*p = 9; // errore! Accesso usando ptr non 
		// inizializzato
```

- Accesso a un oggetto uscito dallo scope
```c++
int* f() 
{ 
	int x = 7; 
	// ... 
	return &x; 
} 

int* p = f(); 
*p = 15; // a cosa punta?
```

- Un caso logicamente analogo
```c++
std::vector& ff() 
{ 
	std::vector x(7); 
	return x; 
} // x è distrutto qui! 

std::vector& p = ff(); 
p[4] = 15; // ouch! (BS)
```

A cosa si riferisce p?
La funzione `ff` restituisce un riferimento a un oggetto locale, il che comporta un comportamento non definito. 
Quindi, il riferimento `p` diventa un riferimento non valido dopo la fine della funzione `ff`.
