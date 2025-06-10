# Strutture Dati in JavaScript: Guida Approfondita

## Introduzione

Le strutture dati sono fondamentali nella programmazione in quanto permettono di organizzare e gestire i dati in modo efficiente. JavaScript offre diverse strutture dati native che possono essere utilizzate per risolvere problemi specifici. In questa guida, esploreremo in dettaglio le strutture dati in JavaScript, con particolare attenzione agli array bidimensionali e agli oggetti, che sono particolarmente utili per lo sviluppo di giochi come la dama.

## Array in JavaScript

Gli array sono una delle strutture dati più utilizzate in JavaScript. Sono collezioni ordinate di elementi, dove ogni elemento può essere di qualsiasi tipo (numeri, stringhe, oggetti, altri array, ecc.).

### Creazione di array

```javascript
// Creazione di un array vuoto
let array1 = [];

// Creazione di un array con elementi
let array2 = [1, 2, 3, 4, 5];

// Array con elementi di tipi diversi
let array3 = [1, "due", true, {nome: "Mario"}, [5, 6]];

// Creazione di un array con il costruttore Array
let array4 = new Array(5);  // Array vuoto con lunghezza 5
```

### Accesso agli elementi

Gli elementi di un array sono indicizzati a partire da 0:

```javascript
let numeri = [10, 20, 30, 40, 50];

console.log(numeri[0]);  // 10 (primo elemento)
console.log(numeri[2]);  // 30 (terzo elemento)
console.log(numeri[numeri.length - 1]);  // 50 (ultimo elemento)
```

### Metodi principali degli array

JavaScript offre numerosi metodi per manipolare gli array:

- `push()` e `pop()`: aggiungono/rimuovono elementi alla fine dell'array
- `unshift()` e `shift()`: aggiungono/rimuovono elementi all'inizio dell'array
- `splice()`: aggiunge/rimuove elementi in qualsiasi posizione
- `slice()`: estrae una porzione dell'array
- `concat()`: unisce due o più array
- `join()`: converte l'array in una stringa
- `forEach()`, `map()`, `filter()`, `reduce()`: metodi per iterare e trasformare gli array

```javascript
let numeri = [1, 2, 3];

numeri.push(4);  // [1, 2, 3, 4]
numeri.pop();    // [1, 2, 3]

numeri.unshift(0);  // [0, 1, 2, 3]
numeri.shift();     // [1, 2, 3]

numeri.splice(1, 1, 'due');  // [1, 'due', 3]

let numeriDoppi = numeri.map(n => typeof n === 'number' ? n * 2 : n);  // [2, 'due', 6]
```

## Array Bidimensionali

Gli array bidimensionali sono array di array, e sono particolarmente utili per rappresentare strutture a griglia come una scacchiera.

### Creazione di array bidimensionali

```javascript
// Metodo 1: Creazione diretta
let griglia1 = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
];

// Metodo 2: Creazione dinamica
let griglia2 = [];
for (let i = 0; i < 3; i++) {
    griglia2[i] = [];
    for (let j = 0; j < 3; j++) {
        griglia2[i][j] = i * 3 + j + 1;
    }
}
```

### Accesso agli elementi

```javascript
let griglia = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
];

console.log(griglia[0][0]);  // 1 (elemento in alto a sinistra)
console.log(griglia[1][1]);  // 5 (elemento centrale)
console.log(griglia[2][2]);  // 9 (elemento in basso a destra)
```

### Iterazione su array bidimensionali

```javascript
let griglia = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
];

// Iterazione con cicli for annidati
for (let i = 0; i < griglia.length; i++) {
    for (let j = 0; j < griglia[i].length; j++) {
        console.log(`Elemento in posizione [${i}][${j}]: ${griglia[i][j]}`);
    }
}

// Iterazione con forEach
griglia.forEach((riga, i) => {
    riga.forEach((elemento, j) => {
        console.log(`Elemento in posizione [${i}][${j}]: ${elemento}`);
    });
});
```

### Applicazione: rappresentare una scacchiera

Un array bidimensionale è perfetto per rappresentare una scacchiera, come nel gioco della dama:

```javascript
// Inizializzazione di una scacchiera 8x8 vuota
let scacchiera = [];
for (let riga = 0; riga < 8; riga++) {
    scacchiera[riga] = [];
    for (let colonna = 0; colonna < 8; colonna++) {
        scacchiera[riga][colonna] = null;  // Casella vuota
    }
}

// Posizionamento delle pedine (esempio)
scacchiera[0][1] = 'pedina_nera';
scacchiera[7][0] = 'pedina_bianca';

// Verifica se una casella è occupata
function isCasellaOccupata(riga, colonna) {
    return scacchiera[riga][colonna] !== null;
}
```

## Oggetti JavaScript

Gli oggetti in JavaScript sono collezioni di coppie chiave-valore, dove le chiavi sono stringhe (o simboli) e i valori possono essere di qualsiasi tipo. Gli oggetti sono estremamente versatili e sono alla base di molte funzionalità avanzate di JavaScript.

### Creazione di oggetti

```javascript
// Notazione letterale (object literal)
let persona1 = {
    nome: "Mario",
    cognome: "Rossi",
    età: 30,
    indirizzo: {
        via: "Via Roma",
        città: "Milano"
    }
};

// Costruttore Object
let persona2 = new Object();
persona2.nome = "Luigi";
persona2.cognome = "Verdi";
persona2.età = 25;

// Funzione costruttore personalizzata
function Persona(nome, cognome, età) {
    this.nome = nome;
    this.cognome = cognome;
    this.età = età;
    this.presentati = function() {
        return `Mi chiamo ${this.nome} ${this.cognome} e ho ${this.età} anni.`;
    };
}

let persona3 = new Persona("Anna", "Bianchi", 28);
```

### Accesso alle proprietà

```javascript
let persona = {
    nome: "Mario",
    cognome: "Rossi",
    età: 30
};

// Notazione con punto
console.log(persona.nome);  // "Mario"

// Notazione con parentesi quadre
console.log(persona["cognome"]);  // "Rossi"

// Utile quando il nome della proprietà è in una variabile
let proprietà = "età";
console.log(persona[proprietà]);  // 30
```

### Metodi degli oggetti

I metodi sono funzioni che appartengono a un oggetto:

```javascript
let persona = {
    nome: "Mario",
    cognome: "Rossi",
    età: 30,
    presentati: function() {
        return `Mi chiamo ${this.nome} ${this.cognome} e ho ${this.età} anni.`;
    },
    // Sintassi abbreviata (ES6)
    saluta() {
        return `Ciao, sono ${this.nome}!`;
    }
};

console.log(persona.presentati());  // "Mi chiamo Mario Rossi e ho 30 anni."
console.log(persona.saluta());      // "Ciao, sono Mario!"
```

### Iterazione sulle proprietà

```javascript
let persona = {
    nome: "Mario",
    cognome: "Rossi",
    età: 30
};

// for...in
for (let chiave in persona) {
    console.log(`${chiave}: ${persona[chiave]}`);
}

// Object.keys()
Object.keys(persona).forEach(chiave => {
    console.log(`${chiave}: ${persona[chiave]}`);
});

// Object.entries()
Object.entries(persona).forEach(([chiave, valore]) => {
    console.log(`${chiave}: ${valore}`);
});
```

### Applicazione: stato del gioco

Gli oggetti sono ideali per memorizzare lo stato di un gioco, come nel caso della dama:

```javascript
let statoGioco = {
    // Scacchiera (array bidimensionale)
    scacchiera: [],
    
    // Turno corrente
    turnoCorrente: 'bianco',  // 'bianco' o 'nero'
    
    // Conteggio pedine
    pedine: {
        bianche: 12,
        nere: 12
    },
    
    // Pedina selezionata
    pedinaSelezionata: null,
    
    // Mosse disponibili
    mosseDisponibili: [],
    
    // Metodo per cambiare turno
    cambiaTurno() {
        this.turnoCorrente = this.turnoCorrente === 'bianco' ? 'nero' : 'bianco';
    },
    
    // Metodo per verificare se il gioco è finito
    isGameOver() {
        return this.pedine.bianche === 0 || this.pedine.nere === 0;
    },
    
    // Metodo per ottenere il vincitore
    getVincitore() {
        if (!this.isGameOver()) return null;
        return this.pedine.bianche > 0 ? 'bianco' : 'nero';
    }
};

// Inizializzazione della scacchiera
statoGioco.scacchiera = [];
for (let riga = 0; riga < 8; riga++) {
    statoGioco.scacchiera[riga] = [];
    for (let colonna = 0; colonna < 8; colonna++) {
        statoGioco.scacchiera[riga][colonna] = null;
    }
}
```

## Combinare Array e Oggetti

La combinazione di array e oggetti è particolarmente potente e comune in JavaScript:

```javascript
// Array di oggetti
let studenti = [
    { id: 1, nome: "Mario", voti: [8, 7, 9] },
    { id: 2, nome: "Luigi", voti: [6, 8, 7] },
    { id: 3, nome: "Anna", voti: [9, 9, 8] }
];

// Calcolo della media dei voti per ogni studente
let medie = studenti.map(studente => {
    let somma = studente.voti.reduce((acc, voto) => acc + voto, 0);
    let media = somma / studente.voti.length;
    return { id: studente.id, nome: studente.nome, media: media };
});

// Oggetto con array come proprietà
let corso = {
    nome: "Programmazione JavaScript",
    docente: "Prof. Bianchi",
    studenti: studenti,
    getStudenteMigliore() {
        return this.studenti.reduce((migliore, corrente) => {
            let mediaMigliore = migliore.voti.reduce((acc, voto) => acc + voto, 0) / migliore.voti.length;
            let mediaCorrente = corrente.voti.reduce((acc, voto) => acc + voto, 0) / corrente.voti.length;
            return mediaCorrente > mediaMigliore ? corrente : migliore;
        });
    }
};
```

## Strutture Dati Avanzate

Oltre agli array e agli oggetti, JavaScript offre altre strutture dati avanzate:

### Map

La classe `Map` è simile agli oggetti, ma con alcune differenze chiave:
- Le chiavi possono essere di qualsiasi tipo (non solo stringhe)
- Mantiene l'ordine di inserimento
- Ha metodi dedicati per la manipolazione

```javascript
let mappa = new Map();

// Aggiunta di elementi
mappa.set('chiave1', 'valore1');
mappa.set(42, 'valore2');
mappa.set({id: 1}, 'valore3');

// Accesso agli elementi
console.log(mappa.get('chiave1'));  // 'valore1'
console.log(mappa.has(42));        // true

// Iterazione
mappa.forEach((valore, chiave) => {
    console.log(`${chiave} => ${valore}`);
});

// Altre operazioni
console.log(mappa.size);  // 3
mappa.delete(42);         // Rimuove l'elemento con chiave 42
mappa.clear();            // Rimuove tutti gli elementi
```

### Set

La classe `Set` rappresenta una collezione di valori unici:

```javascript
let set = new Set([1, 2, 3, 3, 4, 4, 5]);

console.log(set);  // Set(5) {1, 2, 3, 4, 5} (i duplicati sono stati rimossi)

// Aggiunta di elementi
set.add(6);
set.add(1);  // Non viene aggiunto perché già presente

// Verifica della presenza
console.log(set.has(3));  // true

// Rimozione di elementi
set.delete(4);

// Iterazione
set.forEach(valore => {
    console.log(valore);
});

// Conversione in array
let array = [...set];
```

### WeakMap e WeakSet

Sono versioni "deboli" di Map e Set, utili per evitare memory leak quando si usano oggetti come chiavi.

## Prestazioni e Considerazioni

La scelta della struttura dati giusta può avere un impatto significativo sulle prestazioni dell'applicazione:

- Gli array sono ottimizzati per accessi sequenziali e operazioni alla fine dell'array
- Gli oggetti sono ottimizzati per accessi diretti tramite chiave
- Map e Set hanno prestazioni migliori per operazioni frequenti di aggiunta/rimozione

Considerazioni sulla complessità temporale:

| Operazione | Array | Oggetto | Map | Set |
|------------|-------|---------|-----|-----|
| Accesso    | O(1)  | O(1)    | O(1)| N/A |
| Ricerca    | O(n)  | O(1)    | O(1)| O(1)|
| Inserimento (fine) | O(1) | O(1) | O(1)| O(1)|
| Inserimento (inizio) | O(n) | O(1) | O(1)| O(1)|
| Rimozione  | O(n)  | O(1)    | O(1)| O(1)|

## Serializzazione e Deserializzazione

Per salvare e ripristinare lo stato di un'applicazione, è comune convertire strutture dati in stringhe JSON:

```javascript
// Serializzazione
let statoGioco = {
    scacchiera: [[null, 'pedina_nera'], ['pedina_bianca', null]],
    turnoCorrente: 'bianco'
};

let statoSerializzato = JSON.stringify(statoGioco);
console.log(statoSerializzato);
// '{"scacchiera":[[null,"pedina_nera"],["pedina_bianca",null]],"turnoCorrente":"bianco"}'

// Salvataggio (esempio con localStorage)
localStorage.setItem('statoGioco', statoSerializzato);

// Deserializzazione
let statoRipristinato = JSON.parse(localStorage.getItem('statoGioco'));
console.log(statoRipristinato);
```

Limitazioni di JSON.stringify/parse:
- Non serializza funzioni, undefined, simboli
- Non mantiene riferimenti circolari
- Non mantiene tipi come Date, Map, Set (vengono convertiti)

## Conclusione

Le strutture dati sono fondamentali per organizzare e manipolare i dati in modo efficiente. In JavaScript, gli array bidimensionali e gli oggetti sono particolarmente utili per lo sviluppo di giochi come la dama, permettendo di rappresentare la scacchiera e lo stato del gioco in modo chiaro e funzionale.

La scelta della struttura dati giusta dipende dal problema specifico e dalle operazioni che si devono eseguire più frequentemente. Una buona comprensione delle strutture dati disponibili e delle loro caratteristiche è essenziale per scrivere codice efficiente e mantenibile.