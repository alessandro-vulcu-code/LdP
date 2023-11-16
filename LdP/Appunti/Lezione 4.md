### Stato di una classe
Esempio di una macchina del caffè. Lo stato di una macchina del caffè può essere:
- Acceso-spento
- Idle/not-idle
- Temperatura
- Filled or not
- Led on/led off
- Capsula del caffè
- Sportello

Tutti gli stati della macchina sono bool tranne la temperatura che lo consideriamo double.

- Caricare caffè
- Caricare acqua
- Accenderla o spegnerla
- Erogare sto caffè
- Check sulla temperatura

---
Per definire tipi in c++ si creano con le classi e gli enumeratori.
Una classe specifica per ogni tipo:
- Come sono rappresentati gli oggetti
- Come può essere creata la classe
- Come può essere usata
- Come può essere distrutta

## Classi
Una classe può essere composta da:
- Tipi built in
- Altre classi
- Funzioni

Data member e function member

- Una classe è accessibile attraverso la notazione _oggetto.membro
- Una classe si divide in due interfacce:
	- **Interfaccia public**:
		- Quello che l'utente vede
		- Accessibile dagli utenti
	- **Interfaccia private**:
		- Quello che solo il programmatore vede
		- Non è direttamente accessibile
		- Lettura e scrittura avvengono solo tramite interfaccia
- Di default, _i membri di una classe sono privati_

#### Public - Private
```c
class X { 
	public: 
		// membri pubblici: 
		// interfaccia utente 
		// funzioni // tipi
		// eventuali dati (meglio tenerli privati) 
	private: 
		// membri privati 
		// dettagli implementativi 
		// funzioni 
		// tipi 
		// dati 
};
```

### Struct
- Varianti delle classi
- Unica differenza: i membri di default sono pubblici
- Usate quando ci sono dati ma non funzioni
- Eredità di scrittura dal c

```c++
struct X{
	int m;
}

class X{
	public:
		int m;
}
```

In C++ posso programmare ad oggetti con una serie di potenti strumenti, come:
- Funzioni membro 
- Costruttori 
- Protezione dei dati 
- Overloading degli operatori 
- Gestione della copia dei dati
- (Helper function)

---
# La classe Date
Vediamo ora un esempio di classe, in particolare creeremo la **classe Date** per rappresentare e manipolare le date.

Iniziamo da una struttura semplice: una classe date rappresenta "un giorno nel passato, presente o futuro". 

Una struttura facile può essere la seguente, uno struct con tre interi:
```c++
struct Date { 
	int y; // anno 
	int m; // mese 
	int d; // giorno 
}; 

Date today
```

Una cosa del genere se proviamo ad assegnare un valore ai tre interi funziona e compila. L'unico problema è che a tali variabili posso assegnare un qualsiasi valore anche negativo, facendo perdere senso alla data. Inoltre serve un modo per incrementare la data quando c'è bisogno.

### Helper function
- Funzioni che eseguono operazioni comuni
	- Al posto nostro
	- Programmate una sola volta
	- Debuggate una volta

Implementiamo due funzioni comuni per Date: _init_date ()_ e _add_day ()_

```c++
void init_day(Date& dd, int y, int m, int d) { 
// verifica che y, m, d siano una data corretta
// se sì, inizializza 
} 

void add_day(Date& dd, int n) { 
// incrementa dd di n giorni 
}

int main(void) { 
Date today;
init_day(today, 12, 24, 2005); // errore! giorno 2005...
add_day(today, 1); 
return 0; 
}
```

Le helper function da sole non sono tutto. Sono a disposizione dell'utente, ma potrebbero non essere usate. Pertanto è necessario uno step in più: il _costruttore_ di una classe.

### Costruttore e funzioni membro
- Il costruttore è una funzione membro che ha lo stesso nome della classe $\rightarrow$ inizializza gli oggetti della classe
- Se esiste un solo costruttore con argomenti $\rightarrow$ non fornirli è _errore_

```c++
Date my_birthday; // errore: non inizializzato 

Date today{12, 24, 2007}; // run-time error (da verifica // del costruttore)

Date last {2000, 12, 31}; // ok, colloquiale 

Date next = {2014, 2, 14}; // ok, un po' verboso 

Date christmas = Date{1976, 12, 24}; // ok, un po' verboso

Date last(2000, 12, 31); // ok: colloquiale ma vecchio stile 

last.add_day(1); // ok 

add_day(2); // errore: a quale oggetto // fa riferimento
```

Per inizializzare un oggetto usando il costruttore è meglio utilizzare le graffe
```c++
int x {7}; 
Date next {2014, 2, 14};
```

## Controllo l'accesso ai membri
Finché lasciamo i dati accessibili, chiunque può modificarli intenzionalmente o per sbaglio, quindi è sempre meglio proteggere i dati membro rendendoli privati.

```c++
class Date { 
	int y, m, d; 
	public: 
		Date (int y, int m, int d); 
		void add_day(int n); 
		int month(void) { return m; } 
		int day(void) { return d; } 
		int year(void) { return y; } 
}; 

Date birthday {1970, 12, 30}; // ok 
birthday.m = 14; // errore: Date.m è privato 
std::cout << birthday.month() << '\n'; // ok
```

- Le funzioni membro garantiscono che i _dati membro siano sempre coerenti_
	- Suppongono che lo siano in partenza e garantiscono che lo siano in uscita
	- Garantiscono che l'oggetto sia sempre in uno **stato valido**

#### Riordinamento
Separo l'interfaccia dai dettagli implementativi:

_Date.h_
```c++
class Date { 
	private:
		int y, m, d;
		 
	public: 
		Date (int y, int m, int d); 
		void add_day(int n); 
		int month(void)
		int day(void)
		int year(void)
}; 
```

_Date.cpp_
```c++
#include "Date.h" 
Date::Date(int yy, int mm, int dd) : y{yy}, m{mm}, d{dd} { } 
void Date::add_day(int n) { 
	// ... 
} 
int month(void) // ops! Manca Date:: 
{ 
	return m; 
}
```

 `y{yy}, m{mm}, d{dd} ` è detto initializer list, un modo rapido per creare un oggetto assegnando i valori passati. è un'alternativa al classico assegnanento.

## Rilevamento di errori
Nella classe Date per vedere se ci sono degli errori possiamo creare una funzione _is_valid ()_ che verifica se la data ha senso oppure no. Una funzione del genere normalmente restituisce un bool che può essere restituito. 

Se la data non è valida, cioè se la funzione restituisce _false_, allora viene lanciata un'_eccezione_, che garantisce che l'errore sia gestito.

```c++
Date::Date(int yy, int mm, int dd) : y (yy), m(mm), d(dd) 
{ 
	if (!is_valid()) throw Invalid(); 
} 

bool Date::is_valid() 
{ 
if (m < 1 || m > 12) return false; // ... 
}
```

Una funzione esprime interesse per le eccezioni usando un blocco `try…catch`
```c++
void f(int x, int y) { 
	try { 
		Date dxy{2004, x, y}; 
		std::cout << dxy << '\n'; 
		dxy.add_day(2); 
	} 
	catch(Date::Invalid e) { 
		std::cout << "Invalid date: " << e.c_str() << std::endl; 
		} 
	}
```

# Enumerazioni
Un enum (enumeration) è un tipo definito dall'utente molto semplice 
- Specifica un set di valori 
	- Rappresentati con costanti simboliche 
	- Una variabile è una scelta tra questo set 
- Il corpo è la lista degli enumeratori

```c++
enum class Month { 
	jan = 1, feb, mar, apr, may, jun, jul, aug, sep, oct, nov, dec 
};
```

- Enum class crea uno scope 
	- Jan è specificato come Month:: jan 
- È possibile scegliere valori specifici 
	- In caso contrario: valore del precedente + 1 
	- Parte da 0 • 
- Un altro esempio:
```c++
enum class Day { 
	monday, tuesday, wednesday, thursday, friday, saturday, sunday ù
};
```

#### Assegnamento a enum
```c++
Month m = Month::feb; 

Month m2 = feb; // errore: feb non è in scope 
m = 7; // errore: assegnamento di int a Month 
int n = m; // errore: assegnamento di Month a int 
Month mm = Month(7); // conversione di int a Month (non verificata!)
```

Per gestire "errori" come `Month(9999)` posso scrivere  una semplice funzione di check.
#### Enumerazioni senza classi
È possibile usare un’enumerazione senza scope:
```c++
enum Month { 
	jan = 1, feb, mar, apr, may, jun, jul, aug, sep, oct, nov, dec 
}; 

Month m = feb; // non genera errore
```
---
## Overloading degli operatori
In C++ è possibile implementare quasi tutti gli operatori per operandi di tipo definito dall'utente. In sostanza, posso prendere un operatore classico come +,-, /,<<, ecc..., ed adattarlo al mio oggetto. 

L'overloading è implementato creando una funzione con un nome specifico 
- operator+, operator++, operator*, operator[], ... 
Il C++ riconosce questo pattern e traduce
```c++
Date d {2010, Mon::feb, 21}; 
	++d; // equivalente a operator++(d), 
		 // oppure a d.operator++()
```
Serve per forza almeno un argomento UDT, non è possibile fare l'overloading di tipi built-in già esistenti. Si tratta comunque di uno strumento potente, ma bisogna stare attenti a definire operatori con significati controintuitivi

### Overloading operatore++
Operator++ ha due implementazioni:
- Pre-incremento 
```c++
Month& operator++(Month& m) { 
	m = (m == Month::dec) ? Month::jan : Month(int(m) + 1); 
	return m;
}
```
- Post-incremento
```c++
Month operator++(Month& m, int) { 
	Month m_temp = m; m = (m == Month::dec) ? Month::jan : Month(int(m) + 1); 
	return m_temp;
}
```

L'operatore _?_ serve a controllare se `m == Month::dec`, sennò `Month(int(m) + 1)`.

Vedi esempio di overloading nel file: [[L04.4 - Esempio di overloading.pdf]]