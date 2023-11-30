- Nomi poco significativi, codice difficile da leggere

Nella classe Bookshelf:
```c++
BookShelf::BookShelf(int a) { // Costruttore che riceve la lunghezza dell'utente
	vector_length = a;
	max_length = 2*vector_length;
	elem = new Book[max_length];
	reserve(minimumL);
}
```
Nel costruttore viene impostato il numero di elementi che è `vector_lenght` ad a, che è la dimensione specificata dall'utente. Di conseguenza, il numero di elementi occupati non è zero, ma è già `a`, nonostante la libreria sia vuota.
Poi, cosa fa `reserve(minimumL)`?
Manca il check per `size==0` 

```c++
BookShelf::BookShelf(std::initializer_list<Book> lst) { // initializer list
	vector_length = lst.size();
	elem = new Book[vector_length];
	std::copy(lst.begin(), lst.end(), elem);
}
```
- `vector_length = lst.size();` ma manca `max_length = lst.size();


- Avrei dato un nome diverso a `vector_length` e `max_lenght`, perché ambigui
- `"../include/Bookshelf.h"` inutile, legge lo stesso l'header file
- Il distruttore non ha `elem = nullptr`
- `Book at(int a) const`: non è necessario

- Nel `overload=:
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
- Mettendo come prima riga `delete[] elem` al momento della copia andrà a copiare una libreria vuota. Di conseguenza le due librerie non saranno uguali
- Il metodo `print()` andrebbe fatto con l'overload dell'operatore <<

