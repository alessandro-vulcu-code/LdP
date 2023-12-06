# Ereditarietà

Vogliamo descrivere entità geometriche:
- Punti 
- Forme
- ...
Che classi creare per rappresentare queste varie forme?

Una forma possiede:
- Alcune caratteristiche comuni 
- Descritta da un insieme di punti 
- Funzionalità disegno

È possibile raggruppare le caratteristiche comuni
- Classe base
Le classi più specifiche **ereditano** tali caratteristiche comuni, e le estendono. 
Una scala progettuale usata spesso è produrre:
- Molte classi
- Con poche operazioni

### Entità di base: punto
Definito come struct: ogni valore è ammissibile
Raggruppa le due coordinate

```c++
struct Point { 
	int x, y; 
};
```

Shape implementa un'**interfaccia** comune a tutte le forme. L'effettiva implementazione dipende dalla specifica forma.

## Classe Shape
```c++
class Shape { 
	public: void draw() const; 
	virtual void move(int dx, int dy); 
	
	Point point(int i) const; // accesso ai punti 
	int number_of_points() const; 
	
	Shape(const Shape&) = delete; 
	Shape& operator=(const Shape&) = delete; 
	
	virtual ~Shape() { } 
	
	// ... 
	// continua..
```

```c++
protected: ù
	Shape() { } 
	Shape(initializer_list<Point> lst); 
	
	void draw_lines() const; // disegno 
	void add(Point p); // aggiunge un punto 
	void set_point(int i, Point p); // points[i] = p; 
	
private: 
	std::vector<Point> points; // non usato da tutte le 
	Shape Color lcolor; // dalla libreria grafica 
	Line_style ls; // dalla libreria grafica 
	Color fcolor; // dalla libreria grafica 
};
```

- Costruttore di default e con inizializzazione sono protected, come se fossero `private`.

### Classi astratte
- Se provo a implementare Shape ho un errore di compilazione perché chiamo un costruttore protected! Si dice che è una classe *astratta*. 
- Esiste un altro modo (più frequentemente usato) per definire le classi astratte: **Funzioni virtuali pure**
- Classe astratta: implementa un'interfaccia 

## Ereditarietà
È possibile definire classi che ereditano dalla classe Shape
```c++
class Circle : 
	public Shape { 
	// ... 
};
```

La classe Circle è derivata di Shape, cosa eredita?

- Ereditarietà public: lasse derivata può accedere ai membri public e protected della classe base, ciò che è private non viene visto
- Private poco usato

### "is a"

- Una relazione, esempio Circle è una Shape, cioè un oggetto di classe derivata è un (is a) oggetto di classe base.
- Relazione non biunivoca

### "has a"
- Altro tipo di relazione, definisce la composizione tra classi:
	- Es: un'automobile **ha** un volante
	- Es: una forma **ha** dei punti

## Slicing
La relazione is a rende possibili alcune operazioni problemariche
- Es: potrei inserire un oggetto `Circle` in un `srd::vector` di Shape
- Quali potenziali problemi? 
	- Se circle è derivata di Shape, avrà dei dati membro aggiuntivi (es. Raggio)
	- Usare il costruttore di copia (tramite push_back) causa uno **slicing**
```c++
void my_fct(const Circle& c) 
{ 
	vector<Shape> v; 
	v.push_back(c); 
}
```

Il fenomeno di slicing verifica quando tentiamo di inserire una classe derivata (es., Circle) in uno slot che riesce a contenere solo i membri della classe base.

Il compilatore risolve affettando l'oggetto, lasciando solo i membri che trovano posto (chiesto molto alla prova scritta)

Vengono inseriti nel vector solo i membri che trovano il posto.

Costruttore di copia e operator= devono essere disabilitati tramite la keyword delete
- Evita la creazione di costruttore di copia e operator= di default 
- Ora la seconda riga causa errore di compilazione

```c++
void my_fct(const Circle& c) { 
	vector<Shape> v; 
	v.push_back(c); // errore: costruttore di copia 
					// disabilitato 
}
```

Le classi create per funzionare come classi base di una gerarchia richiedono di **disabilitare**:
- Costruttore di copia (copy constructor) 
- Assegnamento di copia (copy assignment)

**Dubbi: qual è il vero problema dello slicing?** 

---

# Funzioni virtuali
## Gerarchia di classi
Con le gerarchie di classi utilizziamo tre meccanismi fondamentali

1. **Ereditarietà / derivazione:** una classe eredita funzioni e dati membro dalla classe base
2. **Funzioni virtuali:** possibilità di definire la stessa funzione nella classe base e in quella derivata (AKA polimorfismo run-time perché è a tempo di esecuzione che si determina quale funzione deve essere chiamata)
3. **Incapsulamento:** membri private e protected per nascondere i dettagli implementativi semplifica la manutenzione

### Classe Shape
```c++
class Shape

{

public:

    void draw() const;

    void move(int dx, int dy);

    Point point(int i) const;

    int number_of_points() const;

    Shape(const Shape &) = delete;

    Shape &operator=(const Shape &) = delete;

    ~Shape() {} // ...
```

```c++
protected:

Shape() {}

Shape(initializer_list<Point> lst);

void draw_lines() const;

void add(Point p);

void set_point(int i, Point p);

  

private:

std::vector<Point> points;

Color lcolor;

Line_style ls;

Color fcolor;

};
```

### Classe Shape (2)
```c++
class Shape

{

public:

    void draw() const;

    virtual void move(int dx, int dy);

    Point point(int i) const;

    int number_of_points() const;

    Shape(const Shape &) = delete;

    Shape &operator=(const Shape &) = delete;

    virtual ~Shape() {} // ...
```

```c++
protected:

Shape() {}

Shape(initializer_list<Point> lst);

virtual void draw_lines() const;

void add(Point p);

void set_point(int i, Point p);

  

private:

std::vector<Point> points;

Color lcolor;

Line_style ls;

Color fcolor;

};
```

## Overriding
- Una classe derivata che ridefinisce una funzione virtuale di una classe base effettua un override
- Questo fa sì che la funzione nella classe derivata sfrutti l'interfaccia della classe base
- La funzione oggetto di override ha stesso nome, stessi tipi e stessa constness della funzione nella classe base
- **L'overriding è alla base del polimorfismo dinamico (run-time)**

### Overriding e puntatori
- L'overriding ha un funzionamento interessante con i puntatori 
- Consideriamo le classi Base, Derived, Derived 2:
```c++
class Base { // ... }; 

class Derived : public Base { // ... }; 

class Derived2 : public Derived { // ... };
```

- Con i puntatori ho comportamento interessante:
	- Un puntatore a Base può puntare a un oggetto Derived o Derived2
- Un vector<Base*> può gestire una collezione di oggetti diversi (tutti derivati da Base)

```c++
std::vector<Base*> vd, vd2; 

Derived D1, D2, D3; 
Derived2 DD1, DD2, DD3
```

```c++
vd.push_back(&D1); 
vd.push_back(&D2); 
vd.push_back(&D3); 

vd.push_back(&DD1); 
vd.push_back(&DD2); 
vd.push_back(&DD3);
```

- Un vettore di puntatore a Base può gestire oggetti diversi, purché legati da gerarchia di classi
- Il puntatore risolve il problema di Slicing

Vediamo ora alcuni esempi di funzionamento

## Virtual vs. Non virtual

```c++
class Base

  

{

  

public:

        virtual void f() const { cout << "Base::f "; }
        void g() const { cout << "Base::g "; } // non virtuale

};

class Derived : public Base
{
public:

    void f() const { cout << "Derived::f "; }

    void g() const { cout << "Derived::g "; }
};

class Derived2 : public Derived

{

public:

    void f() { cout << "Derived2::f "; } // nessun override: // non è const 
    void g() const { cout << "Derived2::g "; } 
};
```

```c++
void call(const Base& base) { 
	base.f(); 
	base.g(); 
}
```

<span style="color:#00b0f0">Che oggetto può entrare in base? </span>
- <span style="color:#00b0f0">Base? </span>
- <span style="color:#00b0f0">Derived? </span>
- <span style="color:#00b0f0">Derived 2?</span>

Derived è un tipo di Base, quindi può essere fornito come argomento. 
Derived 2 è un tipo di Derived, quindi può essere fornito come argomento.

```c++
int main()

{

    Base base; 
    Derived derived;
    Derived2 derived2;

    call(base); // Base::f, Base::g
    call(derived); // Derived::f, Base::g
    call(derived2); // Derived::f, Base::g

    base.f(); // Base::f
    base.g(); // Base::g

    derived.f(); // Derived::f
    derived.g(); // Derived::g

    derived2.f(); // Derived2::f
    derived2.g(); // Derived2::g

}
```

**RIVEDI PERCHE' IMPORTANTE!**

# Override esplicito
Alcune funzioni sono progettate per fare l'overriding
È possibile dichiarare esplicitamente questa intenzione 
- Override 
- Genera un errore di compilazione se l'override non è implementato 
- Utile sia per chiarezza che per essere sicuri che l'override sia gestito correttamente
Usalo per debuggare che è molto utile

```c++
class Base

{

    virtual void f() const { cout << "Base::f "; }

    void g() const { cout << "Base::g "; } // non virtuale }; class Derived : public Base { void f() const override { cout << "Derived::f "; } void g() const override { cout << "Derived::g "; } // errore }; class Derived2 : public Derived { void f() override { cout << "Derived2::f "; } // errore void g() const override { cout << "Derived2::g "; } // errore };
```

### Funzioni virtuali pure
Funzioni che esplicitamente non possono essere implementate nella classe base
Rendono la classe virtuale **pura**
È impossibile istanziare oggetti di questa classe 
```c++
class Base { 
	public: virtual void f() = 0; // è richiesto l'overriding 
	virtual void g() = 0; // è richiesto l'overriding 
}; 

Base base; // errore: Base è virtuale pura
```

Perché creare una funzione virtuale pura?
- Non riesco a implementare queste funzioni, lo faranno le mie classi derivate
- Per impedire di istanziare oggetti di una determinata classe
- Può essere un effetto voluto
- Per obbligare tutte le classi derivate a implementare una determinata funzione (l'overriding è obbligatorio per le funzioni virtuali pure)

### Classi astratte
- Abbiamo visto (modulo precedente) che Shape è una classe astratta (o classe virtuale pura)
- Per Shape ciò era ottenuto rendendo i costruttori protected
- Realizzato più correttamente con le funzioni virtuali pure 

### Classe Shape (3)
```c++
class Shape
{
public:

    Shape() {}

    Shape(initializer_list<Point> lst);

    void draw() const;

    virtual void move(int dx, int dy) = 0;

    Point point(int i) const;

    int number_of_points() const;

    Shape(const Shape &) = delete;

    Shape &operator=(const Shape &) = delete;

    virtual ~Shape() {} 
    
    // ...
```

```c++
protected:
	virtual void draw_lines() const = 0;
	void add(Point p);
	void set_point(int i, Point p);

private:

	vector<Point> points;
	Color lcolor;
	Line_style ls;
	Color fcolor;
};
```

Se una funzione è virtuale, il suo comportamento è quello atteso dal programmatore
Se una funzione non è virtuale porta a dei problemi, perché il C++ le funzioni non sono tutte così?

Quando ha bisogno di questo servizio, lo chiamo, altrimenti no, cioè ottimizzazione

---

# Ereditarietà: Memory layout degli oggetti

**molto chieste all'esame**

Prendiamo due classi derivate e creiamo una classe Circle e una Open_polyline
- Open_polyline: Nessun dato membro aggiuntivo, si appoggia su points
- Circle: Dato membro aggiuntivo: il raggio (r)

I dati membro di una classe derivata sono aggiunti dopo quelli della classe base:
![[Pasted image 20231129155856.png]]

## Memory layout degli oggetti derivati (funzioni membro)

Una chiamata a funzione virtuale è gestita tramite le virtual table (vtbl)
- Vtbl gestite tramite vptr in ogni oggetto. Nel nostro caso:
	- `Open_polyline` non ridefinisce né `draw_line` né move
	- Ma chiama `Shape:: draw_line` o `Shape::move`
	- `Circle` ridefinisce solo `draw_lines`

### Vtbl e vptr
Per ogni classe (non ogni oggetto!) è definita una tabella che definisce quali funzioni devono essere chiamate
![[Pasted image 20231129160108.png]]

La virtual table è implementata per ogni classe, bla bla bla
Il virtual pointer mi fa arrivare alla virtual table, è definito per ogni oggetto.

---

# Recap polimorfismo
- Polimorfismo: capacità di associare comportamenti diversi alla stessa notazione generica (Vandevoorde/Josuttis)
- Ereditarietà e funzioni virtuali e template ci permettono di usare il polimorfismo

### Polimorfismo dinamico
Polimorfismo dinamico: gestito tramite 
- Ereditarietà 
- Funzioni virtuali

Identificare un insieme comune di funzionalità tra classi collegate
Creare il corrispondente insieme di funzioni virtuali nella classe base 

![[Pasted image 20231206144303.png]]
![[Pasted image 20231206144315.png]]
Il polimorfismo aggangiato all'esigenza di reimplementare `draw()`, altrimenti non compila.

```c++
void MyDraw(GeoObj const& obj) { 
	obj.draw(); // chiamata alla funzione draw() della // relativa classe 
	}
```

Chiamata a funzione virtuale, esiste una sola funzione `MyDraw()` per disegnare tutte le forme:
- La differenziazione è gestita dalle varie funzioni `draw()` nelle classi derivate
- Formalmente chiama solo una delle tre `draw()`, grazie all'override

## Polimorfismo statico
La stessa sintassi è condivisa dalle varie forme. Le classi o funzioni sono definite in maniera indipendente, ma devono poter gestire la stessa sintassi.
Il polimorfismo si concretizza specificando la classe o funzione con vari tipi diversi.

Nel caso del polimorfismo statico, MyDraw diventa una funzione template, in questo caso però la classe è specificata dal template.
```c++
template <typename T> void MyDraw(T const& obj) { 
	obj.draw(); // chiamata alla funzione draw() della 
				// classe specificata nel template 
	}
```
<span style="color:#00b0f0">Quante funzioni MyDraw() esistono?</span>
- Esistono 3 funzioni `MyDraw()`, in questo caso
	- `MyDraw<Circle>(...)`
	-  `MyDraw<Line>(...)`
	- `MyDraw<Rectangle>(...)`
<span style="color:#00b0f0">Perché compila?</span>
- Perché esiste funzione draw per il template, vincola Rectangle ad implementare `draw()`

**Polimorfismo tramite ereditarietà**:
- Vincolato: l'interfaccia delle classi derivate è vincolata dalla classe base
- Dinamico: binding effettuato a run-time (dinamicamente)

**Polimorfismo tramite template**:
- Svincolato (unbounded): l'interfaccia delle classi derivate non è predeterminata (non esiste il corrispondente di una classe base)
- Statico: binding effettuato a tempo di compilazione (staticamente)

## Altre forme di polimorfismo
- Overloading di funzioni
- Overloading di operatori

### dynamic_cast
- è utilizzato per "navigare" nella gerarchia di classi
![[Pasted image 20231206150041.png]]

### Upcasting
- Upcasting è sempre lecito. Un oggetto di classe derivata è un (is a) oggetto di classe base
- Non necessita di cast
- Passo oggetto di classe derivata a oggetto che vorrebbe classe base

```c++
Base base; 

Derived derived; 

Base* pBase = &derived;
```

### Downcasting
- Downcasting: effettuato con *dynamic_cast*
- Funziona con i puntatori e con le reference 
- Unico cast che effettua verifiche a run-time

- Se il cast non è lecito: 
	- Puntatori: ritorna nullptr 
	- Reference: lancia un'eccezione (*bad_cast*)

```c++
Base base; 
Derived derived; 

Derived* pDerived = dynamic_cast<Derived*>(&base);
```