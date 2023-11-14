# Introduzione
## Informazioni generali
Mail: luca.tonin@unipd.it

## Modalità d'esame
- Test di programmazione intermedio e peer review
	- Max __1 punto__
	- *facoltativo*
- Progetto finale a gruppi (uno solo a fine corso)
	- Max __10 punti__
- Per ogni appello
	- Prova scritta di **23 punti**
	- Prova di programmazione su carta da **7 punti**, questo in alternativa al progetto finale (in sostanza ideale fare il progetto finale)

L'esame è superato con una delle combinazioni:
- Consegna del progetto finale e superamento della prova scritta
	- Chiede di lasciare commenti con chi ha fatto cosa ecc
- Prova scritta >= 16 e totale con prova finale è 18

# Esami
24/01/2024 14:00
21/02/2024 14:00
27/06/2024 9:30
10/09/2024 9:30


# Hello World!

### Operatore <<
- Stampa a schermo una stringa, quella che è scritta a destra
- Std:: cout è uno stream (cout è funzione dentro il namespace std)

### # include
- Include tutti i componenti necessari
- Importa la libreria che serve
- Il file è chiamato header file
	- Ostream fa parte della libreria standard --> usa solo output

### Main
- Funzione speciale
- Punto di partenza per ogni eseguibile
- Il programma parte dal main 

### Compilatore
- C++ è linguaggio compilato che traduce il codice sorgente in codice macchina
- Il compilatore verifica grammatica significato e presenza di errori
- Può essere invocato da terminale su riga di comando
`g++ my_exec my_source.cpp -lmy_lib`
- My_exec è l'output eseguibile
- My_source è il nome sorgente
- -lmy_lib nome libreria 

## Processo di compilazione
1) sorgente: formato leggibile
2) file oggetto: codice macchina
3) Linker: codice macchina + librerie necessarie per l'esecuzione

Prova di scrittura