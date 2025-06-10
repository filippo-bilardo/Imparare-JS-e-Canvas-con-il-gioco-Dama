# Guida Didattica: Dama in JavaScript - Step 1: La Scacchiera

## Obiettivo
In questo primo step, impareremo a creare la base del gioco della dama: la scacchiera. Realizzeremo una griglia 8x8 con caselle alternate di colore chiaro e scuro utilizzando HTML Canvas e JavaScript.

## Concetti teorici da approfondire

### 1. HTML Canvas
- **Cos'è Canvas**: Un elemento HTML che fornisce un'area di disegno bitmap controllabile tramite JavaScript
- **Contesto di rendering**: L'oggetto che fornisce i metodi per disegnare sul canvas
- **Sistema di coordinate**: Come funziona il sistema di coordinate in Canvas (0,0 è l'angolo in alto a sinistra)

### 2. Strutture dati in JavaScript
- **Array bidimensionali**: Come rappresentare una griglia utilizzando array annidati
- **Oggetti JavaScript**: Come utilizzare gli oggetti per memorizzare lo stato del gioco

### 3. Programmazione modulare
- **Funzioni**: Organizzazione del codice in funzioni con responsabilità specifiche
- **Scope delle variabili**: Differenza tra variabili globali e locali

## Analisi del codice

### Struttura HTML e CSS
```html
<!DOCTYPE html>
<html lang="it">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Dama - Step 1: Scacchiera</title>
    <style>
        body {
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            height: 100vh;
            margin: 0;
            background-color: #f0f0f0;
            font-family: Arial, sans-serif;
        }
        
        h1 {
            margin-bottom: 10px;
        }
        
        canvas {
            border: 2px solid #333;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.2);
        }
        
        .description {
            margin-top: 20px;
            max-width: 600px;
            text-align: center;
        }
    </style>
</head>
```

**Spiegazione:**
- Utilizziamo un elemento `<canvas>` con dimensioni 500x500 pixel
- Il CSS centra tutti gli elementi nella pagina e aggiunge stili per migliorare l'aspetto visivo
- Utilizziamo Flexbox (`display: flex`) per organizzare gli elementi in colonna

### JavaScript: Inizializzazione

```javascript
// Riferimenti agli elementi DOM
const canvas = document.getElementById('checkerboard');
const ctx = canvas.getContext('2d');

// Costanti del gioco
const BOARD_SIZE = 8;
const SQUARE_SIZE = canvas.width / BOARD_SIZE;

// Stato del gioco
let gameState = {
    board: [] // Matrice 8x8 che rappresenta la scacchiera
};
```

**Spiegazione:**
- Otteniamo un riferimento all'elemento canvas e al suo contesto 2D
- Definiamo costanti per la dimensione della scacchiera (8x8) e la dimensione di ogni casella
- Creiamo un oggetto `gameState` per memorizzare lo stato del gioco, con una proprietà `board` che sarà un array bidimensionale

### JavaScript: Creazione della scacchiera

```javascript
// Inizializzazione del gioco
function initGame() {
    initBoard();
    drawBoard();
}

// Step 1: Inizializzazione della scacchiera
function initBoard() {
    gameState.board = [];
    
    for (let row = 0; row < BOARD_SIZE; row++) {
        gameState.board[row] = [];
        for (let col = 0; col < BOARD_SIZE; col++) {
            gameState.board[row][col] = null; // Inizialmente tutte le caselle sono vuote
        }
    }
}
```

**Spiegazione:**
- La funzione `initGame()` è il punto di ingresso che chiama le altre funzioni
- `initBoard()` crea un array bidimensionale 8x8 inizializzando tutte le caselle a `null`
- Utilizziamo cicli `for` annidati per iterare su righe e colonne

### JavaScript: Disegno della scacchiera

```javascript
// Step 1: Disegno della scacchiera
function drawBoard() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    
    for (let row = 0; row < BOARD_SIZE; row++) {
        for (let col = 0; col < BOARD_SIZE; col++) {
            // Determina il colore della casella (alternando bianco e nero)
            const isLightSquare = (row + col) % 2 === 0;
            ctx.fillStyle = isLightSquare ? '#EBECD0' : '#779556';
            
            // Disegna la casella
            ctx.fillRect(col * SQUARE_SIZE, row * SQUARE_SIZE, SQUARE_SIZE, SQUARE_SIZE);
        }
    }
}

// Inizializza il gioco
initGame();
```

**Spiegazione:**
- `drawBoard()` disegna la scacchiera sul canvas
- `ctx.clearRect()` pulisce l'intero canvas prima di disegnare
- Utilizziamo cicli `for` annidati per iterare su ogni casella
- La formula `(row + col) % 2 === 0` determina se una casella deve essere chiara o scura
- `ctx.fillRect()` disegna un rettangolo pieno nella posizione corretta

## Esercizi pratici

1. **Modifica i colori della scacchiera**: Cambia i colori delle caselle per personalizzare l'aspetto del gioco.

2. **Aggiungi coordinate**: Modifica la funzione `drawBoard()` per aggiungere numeri e lettere ai bordi della scacchiera (come in una scacchiera di scacchi).

3. **Scacchiera ridimensionabile**: Modifica il codice per permettere di cambiare la dimensione della scacchiera (ad esempio 6x6 o 10x10) modificando solo la costante `BOARD_SIZE`.

4. **Animazione**: Aggiungi un'animazione semplice alla scacchiera, come un effetto di fade-in all'avvio del gioco.

## Approfondimenti

- **Canvas API**: Esplora la documentazione completa dell'API Canvas su [MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API)
- **Strutture dati in JavaScript**: Approfondisci come funzionano gli array e gli oggetti in JavaScript
- **Programmazione modulare**: Studia i principi della programmazione modulare e come applicarli in JavaScript

## Conclusione

In questo primo step, abbiamo creato la base del nostro gioco della dama: una scacchiera 8x8 con caselle alternate. Abbiamo imparato a utilizzare l'elemento Canvas di HTML per disegnare la scacchiera e abbiamo creato una struttura dati per rappresentarla. Nel prossimo step, aggiungeremo le pedine sulla scacchiera.