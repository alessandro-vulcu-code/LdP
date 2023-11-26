## 1. Class book
Private:
- ISBN
	- String di 13 caratteri esatti 
	- Const
	- Check su isbn
- Titolo
- Nome
- Cognome
	- Solo un autore per libro
- Data di copyright - vedi Date
	- Data nullable
- Stato prestito/disponibile

Funzioni membro:
- Getter e setter
- Costruttori
	- Nome, cognome, titolo, isbn, data?
- Funzioni per gestire prestito e restituzione
- Private - check della data
- Overload che confrontano due Book basandosi sull'isbn
	- Operator==
	- Operator!=
	- Operator<<

## 2. Modificare myVector in bookshelf
Come costruttori:
- ` BookShelf shelf(10);` con size 
- `shelf.push_back(mybook);`, Aggiunge l’elemento mybook al vettore shelf; mybook è un oggetto della classe Book precedentemente creato
- `shelf.pop_back();` che rimuove l'ultimo elemento (se esiste) dal vettore shelf


