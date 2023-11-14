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