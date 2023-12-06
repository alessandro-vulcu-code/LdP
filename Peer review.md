A cura degli studenti Vulcu Alessandro, Gabriele Zolla e Giorgio Prizzon (gruppo **DreamTeam**).
La seguente revisione fa riferimento al progetto intermedio del gruppo *Cristian-Marco-Gomex*.
## Generale
- Nel programma sono stati dati dei nomi poco significativi alle variabili
- Il fatto che il programma sia interattivo non consente il testing di tutte le funzioni messe a disposizione da esso. Piuttosto, in questo caso, sarebbe stato meglio creare un "main di test".
- Il README è privo di informazioni dettagliate. Un file del genere secondo noi dovrebbe descrivere bene il progetto, che cosa fa, come è stato implementato e strutturato, oltre che come può essere eseguito.
	- <span style="color:#00b050">SUGGERIMENTO</span>: per un progetto di programmazione generalmente il README viene scritto in markdown, il che rende più chiaro e leggibile tale documento perché supporta la scrittura di titoli, vari stili di testo e l'inserimento di snippet di codice. Non è assolutamente una critica perché non tutti sanno scrivere in markdown quindi giustamente il professore ha detto di scrivere un file di testo, però secondo noi impararlo renderà i vostri futuri README più leggibili e funzionali
- Nel `CmakeLists.txt` anche in questo caso gli `include/<headerfile>` non sono necessari, basta mettere il nome della libreria, perché automaticamente `include_directories (include)` include gli header in quella cartella.

- Gli header file sono scritti correttamente e tutte le funzioni scritte sono poi definite nei rispettivi .cpp. 
- Con un main di testing, ad esempio quello implementato nel nostro codice, il programma compila ed esegue correttamente

---
## Bookshelf.cpp
1) Nel seguente costruttore:
	- Viene impostato il numero di elementi che è `vector_lenght` ad `a`, che è la dimensione specificata dall'utente. Di conseguenza, il numero di elementi occupati non è zero, ma è già `a`, nonostante la libreria sia vuota.
	- Manca il check per `size==0` 
	- Buona idea l'implementazione di `minimumL` che consente di evitare errori nel caso in cui size fosse impostato uguale a zero, richiamando in caso `reserve(minimumL)` che ridimensiona il vector a `size = 1`
```c++
BookShelf::BookShelf(int a) { // Costruttore che riceve la lunghezza dell'utente
	vector_length = a;
	max_length = 2*vector_length;
	elem = new Book[max_length];
	reserve(minimumL);
}
```

2) Nel seguente costruttore con l' `initializer_list`:
	- `vector_length = lst.size();` va bene, ma manca `max_length = lst.size();
```c++
BookShelf::BookShelf(std::initializer_list<Book> lst) { // initializer list
	vector_length = lst.size(); //X
	elem = new Book[vector_length];
	std::copy(lst.begin(), lst.end(), elem);
}
```


3) In `Book& BookShelf::at(int a)`, la classe `Invalid()` avrebbe senso come idea gestire l'errore, però va implementata. Una classe vuota per lanciare un errore secondo noi è inutile.
```c++
Book BookShelf::at(int a) const { // ritorna elem in posizione inserita dall'utente
    if (a < vector_length && a>=0)
        return elem[a];
    else
        throw Invalid(); // va bene, però...
}
```
4) `Book at(int a) const`: non dovrebbe essere necessario, nella classe del Vector non è presente. 

5) In `push_back()`: <span style="color:#00b050">consiglio</span>: richiamare il metodo `reserve()` invece di riscriverne il codice
```c++
void BookShelf::push_back(Book b) {
    if (max_length == 0) {
        max_length++;
        elem = new Book[max_length];
    }
    if (vector_length == max_length) {
        max_length = 2*max_length;            //Queste due righe sono implementate
        Book* p = new Book[max_length];       //in reserve() --> codice ripetuto
        std::copy(elem, elem + vector_length, p);
        delete[] elem;
        elem = p;
    }
    elem[vector_length] = b;
    vector_length++;
}
```

6) <span style="color:#00b050">Suggerimento</span>: nel file header `Bookshelf.h` avrei dato un nome diverso a `vector_length` e `max_lenght`, perché ambigui e possono essere confusi anche da chi legge il codice.

7) Il metodo `print()` andrebbe fatto con l'overload di `operator<<`

8) Nel `overload=:
	- Di per se è funzionante, però scrivere `delete[] elem` nella prima riga non è ideale, andrebbe fatto dopo la copia
	- Quel `for` che copia gli elementi di v in `elem` funziona ma sarebbe meglio usare `copy(...)`
```c++
BookShelf& BookShelf::operator=(const BookShelf& v) {

	delete[] elem;
	max_length = v.max_length;
	vector_length = v.vector_length;
	elem = new Book[max_length];
	for (int i = 0; i < vector_length; i++)
		elem[i] = v[i];
	return *this;

}
```

9) Nel costruttore di copia, ci è parso molto strano l'utilizzo di `*this = v;`. A seguito di alcune prove abbiamo capito che ciò che fa è sostituire l'operazione di copia. La copia avviene con successo, tuttavia secondo noi, per evitare errori spiacevoli soprattutto se l'overload di `operator=` è mal implementato, è più sicuro utilizzare `copy(v.elem, v.elem + vector_length, elem)`, come abbiamo visto a lezione
```c++
BookShelf::BookShelf(const BookShelf& v) { // copy constructor
	vector_length = v.vector_length;
	max_length = v.max_length;
	elem = new Book[max_length];
	*this = v;
	// meglio copy(v.elem, v.elem + vector_length, elem);
}
```
1) Il distruttore non ha `elem = nullptr`
```c++
BookShelf::~BookShelf() { // distruttore
    delete[] elem;
    // elem = nullptr <-- MANCANTE
}
```


---
## Book.cpp
1) Definizioni metodi e overload operatore “<<” corrette
2) [riga 30 - 35]: all'interno del costruttore dell'istanza book, nella verifica della validità del codice ISBN, viene controllata solo la lunghezza della stringa. Essendo il suddetto codice formato da numeri e trattini (es. xxx-xxx-xxx-x) disposti con ordine preciso, sarebbe più opportuno controllarne anche la correttezza.
```c++
{ ...
	if (isbn.length() == 13) {
		book_ISBN = isbn;
		book_state = true;
	} else {
		std::cout << "\n\n****Errore: Lunghezza del codice ISBN non valida. Il codice ISBN deve avere lunghezza 13. ****\n\n" << std::endl;
		book_state = false;
	}
}   //verifica della corretteza del codice ISBN approssimativa
```

3) [riga 98 - 99]: nella funzione returnBook () viene settato lo stato di book a true, ma se uso la funzione su un libro già disponibile, non viene controllata la disponibilità. Quindi controintuitivamente potrei restituire un libro già presente. Sarebbe meglio mandare in output un messaggio che informi della disponibilità (come fatto per il metodo borrowBook ())
```c++
void Book::returnBook() {
	book_state = true;
} // non genera errori ma sarebbe meglio inserire un check per verificare se un libro fosse già stato restituito
```

La classe Book è funzionate, da rivedere con un po' di accortezza soprattutto l'implementazione del costruttore.

---
## Date.cpp
- L'overloading di `operator=` non è necessario, in quanto non ritorna un riferimento
- Qual è il senso di fare l'overload di `operator<<` solo per il Month? Lo `static_cast` è preferibile non usarlo
A parte questi due punti, Date è corretto e funzionante

---

## Conclusioni
Dopo aver esaminato il progetto, è evidente che, nonostante alcune correzioni da apportare in vari punti, il programma compila ed esegue correttamente, svolgendo le operazioni previste. Alcuni consigli utili basati su questa revisione includono:

- Migliorare la chiarezza dei nomi delle variabili, evitando denominazioni generiche (es. `a`, `b`, `pippo`, `paperino`), e cercando di evitare nomi simili per variabili che hanno scopi diversi (ad esempio, `vector_length` e `max_length`).
- Nei progetti che richiedono dimostrazioni di corretto funzionamento delle funzioni implementate, è consigliabile creare un main dimostrativo e di test. Tale main, una volta compilato ed eseguito, deve mostrare in modo esplicito le azioni compiute dalle funzioni. Sebbene il main interattivo possa essere piacevole, risulta meno funzionale per chi deve effettuare i test.
- Prestare particolare attenzione agli errori associati alla teoria appresa durante le lezioni, specialmente quelli riscontrati nella classe Bookshelf. Anche se il programma può funzionare in modo apparentemente corretto, errori legati alla teoria possono causare problemi inaspettati.

Questi suggerimenti mirano a migliorare la leggibilità, la funzionalità e la correttezza del vostro progetto e di quelli futuri. 
