# Programmazione Modulare in JavaScript: Guida Approfondita

## Introduzione

La programmazione modulare è un paradigma di programmazione che consiste nel suddividere un programma in moduli o componenti più piccoli e indipendenti. Questo approccio migliora la leggibilità, la manutenibilità e la riusabilità del codice. In JavaScript, la programmazione modulare è particolarmente importante per gestire la complessità delle applicazioni web moderne, inclusi i giochi come la dama.

## Principi della Programmazione Modulare

La programmazione modulare si basa su alcuni principi fondamentali:

1. **Suddivisione**: Dividere il programma in moduli più piccoli e gestibili
2. **Astrazione**: Nascondere i dettagli implementativi e fornire interfacce chiare
3. **Incapsulamento**: Raggruppare dati e funzionalità correlate
4. **Riusabilità**: Creare moduli che possono essere utilizzati in diverse parti del programma o in progetti diversi
5. **Manutenibilità**: Facilitare la modifica e l'aggiornamento del codice

## Funzioni in JavaScript

Le funzioni sono i blocchi fondamentali della programmazione modulare in JavaScript. Una funzione è un blocco di codice progettato per eseguire un compito specifico e può essere richiamata da altre parti del programma.

### Dichiarazione di Funzioni

In JavaScript, ci sono diversi modi per dichiarare una funzione:

#### 1. Dichiarazione di funzione (Function Declaration)

```javascript
function nomeFunzione(parametro1, parametro2) {
    // Corpo della funzione
    return risultato;
}
```

#### 2. Espressione di funzione (Lambda Expression) - ES5

```javascript
const nomeFunzione = function(parametro1, parametro2) {
    // Corpo della funzione
    return risultato;
};
```

#### 3. Funzione freccia (Arrow Function) - ES6

```javascript
const nomeFunzione = (parametro1, parametro2) => {
    // Corpo della funzione
    return risultato;
};

// Versione abbreviata per funzioni semplici
const somma = (a, b) => a + b;
```

### Parametri e Argomenti

I parametri sono le variabili elencate nella definizione della funzione, mentre gli argomenti sono i valori effettivamente passati alla funzione quando viene chiamata.

```javascript
// Definizione con due parametri
function moltiplica(a, b) {
    return a * b;
}

// Chiamata con due argomenti
let risultato = moltiplica(5, 3);  // risultato = 15
```

#### Parametri di default (ES6)

```javascript
function saluta(nome = 'Utente') {
    return `Ciao, ${nome}!`;
}

saluta();        // "Ciao, Utente!"
saluta('Mario');  // "Ciao, Mario!"
```

#### Parametri rest (ES6)

```javascript
function somma(...numeri) {
    return numeri.reduce((totale, numero) => totale + numero, 0);
}

somma(1, 2, 3, 4);  // 10
```

### Valore di Ritorno

Le funzioni possono restituire un valore utilizzando l'istruzione `return`. Se non viene specificato un valore di ritorno, la funzione restituisce `undefined`.

```javascript
function quadrato(x) {
    return x * x;
}

function stampaMessaggio(messaggio) {
    console.log(messaggio);
    // Nessun return esplicito, quindi restituisce undefined
}
```

### Funzioni Pure e Impure

Una funzione pura è una funzione che:
1. Dato lo stesso input, restituisce sempre lo stesso output
2. Non ha effetti collaterali (non modifica variabili esterne, non effettua operazioni I/O, ecc.)

```javascript
// Funzione pura
function somma(a, b) {
    return a + b;
}

// Funzione impura (modifica una variabile esterna)
let totale = 0;
function aggiungiAlTotale(valore) {
    totale += valore;
    return totale;
}
```

Le funzioni pure sono più facili da testare, debuggare e riutilizzare.

## Organizzazione del Codice in Funzioni

Una buona pratica nella programmazione modulare è organizzare il codice in funzioni con responsabilità specifiche. Questo principio è noto come "Single Responsibility Principle" (Principio di Responsabilità Singola).

### Esempio: Gioco della Dama

Nel gioco della dama, possiamo organizzare il codice in funzioni con responsabilità specifiche:

```javascript
// Inizializzazione del gioco
function initGame() {
    initBoard();
    drawBoard();
    setupEventListeners();
}

// Inizializzazione della scacchiera
function initBoard() {
    gameState.board = [];
    
    for (let row = 0; row < BOARD_SIZE; row++) {
        gameState.board[row] = [];
        for (let col = 0; col < BOARD_SIZE; col++) {
            gameState.board[row][col] = null;
        }
    }
    
    // Posizionamento iniziale delle pedine
    setupPieces();
}

// Disegno della scacchiera
function drawBoard() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    
    for (let row = 0; row < BOARD_SIZE; row++) {
        for (let col = 0; col < BOARD_SIZE; col++) {
            // Determina il colore della casella
            const isLightSquare = (row + col) % 2 === 0;
            ctx.fillStyle = isLightSquare ? '#EBECD0' : '#779556';
            
            // Disegna la casella
            ctx.fillRect(col * SQUARE_SIZE, row * SQUARE_SIZE, SQUARE_SIZE, SQUARE_SIZE);
            
            // Disegna la pedina, se presente
            const piece = gameState.board[row][col];
            if (piece) {
                drawPiece(piece, row, col);
            }
        }
    }
}

// Disegno di una pedina
function drawPiece(piece, row, col) {
    const x = col * SQUARE_SIZE + SQUARE_SIZE / 2;
    const y = row * SQUARE_SIZE + SQUARE_SIZE / 2;
    const radius = SQUARE_SIZE * 0.4;
    
    ctx.beginPath();
    ctx.arc(x, y, radius, 0, Math.PI * 2);
    ctx.fillStyle = piece.color === 'white' ? '#FFFFFF' : '#000000';
    ctx.fill();
    ctx.strokeStyle = '#333333';
    ctx.lineWidth = 2;
    ctx.stroke();
    
    // Se è una dama (pezzo promosso), disegna una corona
    if (piece.isKing) {
        drawCrown(x, y, radius * 0.6);
    }
}
```

Questo approccio rende il codice più leggibile e manutenibile, poiché ogni funzione ha una responsabilità specifica e un nome descrittivo.

## Scope delle Variabili

Lo scope (ambito) di una variabile determina dove la variabile è accessibile nel codice. In JavaScript, ci sono diversi tipi di scope:

### Scope Globale

Le variabili dichiarate fuori da qualsiasi funzione o blocco hanno scope globale e sono accessibili da qualsiasi parte del programma.

```javascript
// Variabile globale
const BOARD_SIZE = 8;

function initBoard() {
    // BOARD_SIZE è accessibile qui
    console.log(BOARD_SIZE);  // 8
}
```

### Scope Locale (o di Funzione)

Le variabili dichiarate all'interno di una funzione hanno scope locale e sono accessibili solo all'interno della funzione stessa.

```javascript
function initGame() {
    // Variabile locale
    const gameStarted = true;
    
    console.log(gameStarted);  // true
}

// gameStarted non è accessibile qui
// console.log(gameStarted);  // ReferenceError
```

### Scope di Blocco (ES6)

Le variabili dichiarate con `let` e `const` hanno scope di blocco, il che significa che sono accessibili solo all'interno del blocco in cui sono dichiarate.

```javascript
function drawBoard() {
    for (let row = 0; row < BOARD_SIZE; row++) {
        // row è accessibile solo all'interno di questo ciclo for
        console.log(row);
        
        for (let col = 0; col < BOARD_SIZE; col++) {
            // col è accessibile solo all'interno di questo ciclo for interno
            console.log(col);
        }
        
        // col non è accessibile qui
        // console.log(col);  // ReferenceError
    }
    
    // row non è accessibile qui
    // console.log(row);  // ReferenceError
}
```

### Closure

Una closure è una funzione che ha accesso alle variabili del suo scope esterno, anche dopo che lo scope esterno è stato chiuso.

```javascript
function createCounter() {
    let count = 0;
    
    return function() {
        count++;
        return count;
    };
}

const counter = createCounter();
console.log(counter());  // 1
console.log(counter());  // 2
console.log(counter());  // 3
```

Le closure sono utili per creare funzioni con stato privato.

## Moduli in JavaScript

I moduli sono un modo per organizzare il codice in file separati e gestire le dipendenze. JavaScript ha diversi sistemi di moduli:

### CommonJS (utilizzato in Node.js)

```javascript
// math.js
function somma(a, b) {
    return a + b;
}

function sottrazione(a, b) {
    return a - b;
}

module.exports = {
    somma,
    sottrazione
};

// main.js
const math = require('./math');
console.log(math.somma(5, 3));  // 8
```

### ES Modules (standard moderno)

```javascript
// math.js
export function somma(a, b) {
    return a + b;
}

export function sottrazione(a, b) {
    return a - b;
}

// main.js
import { somma, sottrazione } from './math.js';
console.log(somma(5, 3));  // 8
```

### Moduli tramite IIFE (Immediately Invoked Function Expression)

Prima dell'introduzione dei moduli nativi, si utilizzavano spesso le IIFE per creare moduli:

```javascript
// Modulo per il gioco della dama
const CheckersGame = (function() {
    // Variabili private
    const BOARD_SIZE = 8;
    let gameState = {
        board: [],
        currentPlayer: 'white'
    };
    
    // Funzioni private
    function initBoard() {
        // Implementazione...
    }
    
    function drawBoard() {
        // Implementazione...
    }
    
    // Interfaccia pubblica
    return {
        init: function() {
            initBoard();
            drawBoard();
        },
        
        movePiece: function(fromRow, fromCol, toRow, toCol) {
            // Implementazione...
        },
        
        getCurrentPlayer: function() {
            return gameState.currentPlayer;
        }
    };
})();

// Utilizzo
CheckersGame.init();
```

Questo pattern è noto come "Module Pattern" e permette di nascondere i dettagli implementativi esponendo solo un'interfaccia pubblica.

## Vantaggi della Programmazione Modulare

### 1. Manutenibilità

Il codice modulare è più facile da mantenere perché:
- I problemi possono essere isolati in moduli specifici
- Le modifiche possono essere apportate a un modulo senza influenzare gli altri
- Il codice è più leggibile e comprensibile

### 2. Riusabilità

I moduli ben progettati possono essere riutilizzati in diverse parti dell'applicazione o in progetti diversi, risparmiando tempo e sforzo.

### 3. Collaborazione

La programmazione modulare facilita la collaborazione tra sviluppatori, poiché diversi membri del team possono lavorare su moduli diversi contemporaneamente.

### 4. Testabilità

I moduli con responsabilità ben definite sono più facili da testare, poiché è possibile isolare e testare singole funzionalità.

### 5. Scalabilità

Le applicazioni modulari sono più facili da scalare, poiché nuove funzionalità possono essere aggiunte come nuovi moduli senza dover modificare l'intera base di codice.

## Best Practices

### 1. Principio di Responsabilità Singola

Ogni modulo o funzione dovrebbe avere una sola responsabilità, cioè dovrebbe fare una cosa sola e farla bene.

### 2. Nomi Descrittivi

Utilizza nomi descrittivi per funzioni e variabili che spieghino chiaramente il loro scopo.

```javascript
// Buono
function calculateTotalScore(player) { ... }

// Da evitare
function calc(p) { ... }
```

### 3. Funzioni Piccole e Focalizzate

Le funzioni dovrebbero essere piccole e focalizzate su un compito specifico. Se una funzione diventa troppo grande o complessa, considera di suddividerla in funzioni più piccole.

### 4. Minimizzare le Dipendenze

Cerca di minimizzare le dipendenze tra moduli. Un modulo dovrebbe dipendere il meno possibile da altri moduli.

### 5. Evitare Variabili Globali

Limita l'uso di variabili globali, poiché possono causare conflitti di nomi e rendere difficile il debug.

### 6. Documentazione

Documenta il tuo codice, specialmente le interfacce dei moduli, per facilitare l'uso da parte di altri sviluppatori.

```javascript
/**
 * Calcola il punteggio totale di un giocatore.
 * @param {Object} player - L'oggetto giocatore
 * @returns {number} Il punteggio totale
 */
function calculateTotalScore(player) {
    // Implementazione...
}
```

## Esempio Completo: Struttura Modulare per il Gioco della Dama

Ecco come potrebbe essere strutturato un gioco della dama utilizzando un approccio modulare:

```javascript
// game.js - Modulo principale del gioco
import { initBoard, movePiece, isValidMove } from './board.js';
import { drawBoard, drawPiece, highlightSquare } from './renderer.js';
import { setupEventListeners } from './input.js';

// Stato globale del gioco
const gameState = {
    board: [],
    currentPlayer: 'white',
    selectedPiece: null,
    validMoves: []
};

// Inizializzazione del gioco
function initGame() {
    initBoard(gameState);
    drawBoard(gameState);
    setupEventListeners(gameState, handleSquareClick);
}

// Gestione del click su una casella
function handleSquareClick(row, col) {
    const piece = gameState.board[row][col];
    
    // Se è stata selezionata una casella con una pedina del giocatore corrente
    if (piece && piece.color === gameState.currentPlayer) {
        // Seleziona la pedina
        gameState.selectedPiece = { row, col };
        gameState.validMoves = getValidMoves(row, col);
        
        // Ridisegna la scacchiera con la pedina selezionata evidenziata
        drawBoard(gameState);
        highlightSquare(row, col, 'selected');
        
        // Evidenzia le mosse valide
        gameState.validMoves.forEach(move => {
            highlightSquare(move.row, move.col, 'validMove');
        });
    }
    // Se è stata selezionata una casella vuota ed è una mossa valida
    else if (gameState.selectedPiece && isValidMove(gameState, gameState.selectedPiece.row, gameState.selectedPiece.col, row, col)) {
        // Esegui la mossa
        movePiece(gameState, gameState.selectedPiece.row, gameState.selectedPiece.col, row, col);
        
        // Cambia il turno
        gameState.currentPlayer = gameState.currentPlayer === 'white' ? 'black' : 'white';
        gameState.selectedPiece = null;
        gameState.validMoves = [];
        
        // Ridisegna la scacchiera
        drawBoard(gameState);
    }
    // Se è stata selezionata una casella non valida
    else {
        gameState.selectedPiece = null;
        gameState.validMoves = [];
        
        // Ridisegna la scacchiera
        drawBoard(gameState);
    }
}

// Avvia il gioco
initGame();
```

```javascript
// board.js - Modulo per la gestione della scacchiera
const BOARD_SIZE = 8;

// Inizializzazione della scacchiera
export function initBoard(gameState) {
    gameState.board = [];
    
    for (let row = 0; row < BOARD_SIZE; row++) {
        gameState.board[row] = [];
        for (let col = 0; col < BOARD_SIZE; col++) {
            gameState.board[row][col] = null;
        }
    }
    
    // Posizionamento iniziale delle pedine
    setupPieces(gameState);
}

// Posizionamento iniziale delle pedine
function setupPieces(gameState) {
    for (let row = 0; row < BOARD_SIZE; row++) {
        for (let col = 0; col < BOARD_SIZE; col++) {
            // Posiziona le pedine solo sulle caselle nere (row + col dispari)
            if ((row + col) % 2 === 1) {
                if (row < 3) {
                    gameState.board[row][col] = { color: 'black', isKing: false };
                } else if (row > 4) {
                    gameState.board[row][col] = { color: 'white', isKing: false };
                }
            }
        }
    }
}

// Verifica se una mossa è valida
export function isValidMove(gameState, fromRow, fromCol, toRow, toCol) {
    // Implementazione...
}

// Esegue una mossa
export function movePiece(gameState, fromRow, fromCol, toRow, toCol) {
    // Implementazione...
}

// Ottiene le mosse valide per una pedina
export function getValidMoves(gameState, row, col) {
    // Implementazione...
}
```

```javascript
// renderer.js - Modulo per il rendering grafico
const canvas = document.getElementById('checkerboard');
const ctx = canvas.getContext('2d');
const BOARD_SIZE = 8;
const SQUARE_SIZE = canvas.width / BOARD_SIZE;

// Disegno della scacchiera
export function drawBoard(gameState) {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    
    for (let row = 0; row < BOARD_SIZE; row++) {
        for (let col = 0; col < BOARD_SIZE; col++) {
            // Determina il colore della casella
            const isLightSquare = (row + col) % 2 === 0;
            ctx.fillStyle = isLightSquare ? '#EBECD0' : '#779556';
            
            // Disegna la casella
            ctx.fillRect(col * SQUARE_SIZE, row * SQUARE_SIZE, SQUARE_SIZE, SQUARE_SIZE);
            
            // Disegna la pedina, se presente
            const piece = gameState.board[row][col];
            if (piece) {
                drawPiece(piece, row, col);
            }
        }
    }
}

// Disegno di una pedina
export function drawPiece(piece, row, col) {
    // Implementazione...
}

// Evidenziazione di una casella
export function highlightSquare(row, col, type) {
    // Implementazione...
}
```

```javascript
// input.js - Modulo per la gestione degli input
// Configurazione degli event listener
export function setupEventListeners(gameState, handleSquareClick) {
    const canvas = document.getElementById('checkerboard');
    
    canvas.addEventListener('click', function(event) {
        const rect = canvas.getBoundingClientRect();
        const x = event.clientX - rect.left;
        const y = event.clientY - rect.top;
        
        // Converti le coordinate del click in indici di riga e colonna
        const col = Math.floor(x / (canvas.width / 8));
        const row = Math.floor(y / (canvas.height / 8));
        
        // Chiama la funzione di gestione del click
        handleSquareClick(row, col);
    });
}
```

## Conclusione

La programmazione modulare è un approccio potente per organizzare il codice in modo chiaro, manutenibile e riusabile. In JavaScript, le funzioni e i moduli sono gli strumenti principali per implementare questo paradigma. Utilizzando la programmazione modulare, è possibile sviluppare applicazioni complesse come il gioco della dama in modo strutturato e scalabile.

La comprensione dello scope delle variabili e l'organizzazione del codice in funzioni con responsabilità specifiche sono competenze fondamentali per scrivere codice JavaScript di qualità. Adottando le best practices della programmazione modulare, è possibile creare applicazioni robuste, facili da mantenere e da estendere.