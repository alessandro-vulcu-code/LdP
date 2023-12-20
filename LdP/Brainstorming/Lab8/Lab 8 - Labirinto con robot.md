- Matrice 9 x 9 che rappresenta un labirinto
- Un robot deve trovare l'uscita di un labirinto

- Codifica:
	- Asterischi: posizione dove robot NON può andare
	- Carattere E: uscita "exit" - possono esserci più uscite
	- Carattere S: punto in cui si trova il robot a inizio navigazione (solo una) 
- Vanno fatti tutti i controlli del caso, almeno un'uscita, solo un robot, fill dei punti in più se mat < 9 x 9.

Il robot si muove in due modi: 
- **RandomRobot**: un robot che effettua movimenti casuali tra le 8 caselle vicine alla posizione corrente -> si sposta a caso per trovare l'uscita
- **RightHandRuleRobot**: un robot che si muove in modo che la sua mano destra sia sempre in contatto con una parete. Se la posizione iniziale non è a contatto con nessuna parete, si sceglie una direzione iniziale casuale che determina tutti i successivi spostamenti finché il robot non entra in contatto con una parete.
	- SerarchWall ()

- Il progetto è composto da:
	- Maze: è il labirinto, gestisce lettura da file e fornisce funzioni opportune per la navigazione
	- Robot: è il robot, ha classe virtuale `move` che accetta un argomento di tipo `Maze&` vedere se const o no, che gestisce il movimento dello specifico tipo di robot, gestito tramite le seguenti classi derivate:
		- RandomRobot: derivata di Robot che gestisce la relativa politica di movimento
		- RightHandRuleRobot: derivata di Robot, gestisce la relativa politica di movimento.

La funzione main gestisce l’interazione tra robot e labirinto.