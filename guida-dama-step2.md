# Guida Didattica: Dama in JavaScript - Step 2: Le Pedine

## Obiettivo
In questo secondo step, aggiungeremo le pedine sulla scacchiera creata nel primo step. Impareremo a posizionare le pedine bianche e nere nelle loro posizioni iniziali e a disegnarle sul canvas.

## Concetti teorici da approfondire

### 1. Manipolazione del DOM
- **Selezione di elementi**: Come selezionare elementi HTML tramite JavaScript
- **Aggiornamento dinamico del contenuto**: Come modificare il contenuto degli elementi HTML

### 2. Disegno avanzato con Canvas
- **Disegno di cerchi**: Utilizzo del metodo `arc()` per disegnare forme circolari
- **Stili di riempimento e bordi**: Personalizzazione dell'aspetto degli elementi disegnati

### 3. Strutture dati per rappresentare lo stato del gioco
- **Rappresentazione delle pedine**: Come memorizzare le informazioni sulle pedine nel nostro array bidimensionale
- **Conteggio degli elementi**: Come tenere traccia del numero di pedine per ogni giocatore

## Analisi del codice

### Aggiornamento della struttura HTML

```html
<div class="game-info">
    <div>Pedine Bianche: <span id="whitePieces">12</span></div>
    <div>Pedine Nere: <span id="blackPieces">12</span></div>
</div>
```

**Spiegazione:**
- Aggiungiamo una sezione informativa che mostra il numero di pedine per ogni giocatore
- Utilizziamo elementi `<span>` con ID specifici per poter aggiornare dinamicamente questi valori

### JavaScript: Costanti aggiuntive

```javascript
// Riferimenti agli elementi DOM
const canvas = document.getElementById('checkerboard');
const ctx = canvas.getContext('2d');
const whitePiecesElement = document.getElementById('whitePieces');
const blackPiecesElement = document.getElementById('blackPieces');

// Costanti del gioco
const BOARD_SIZE = 8;
const SQUARE_SIZE = canvas.width / BOARD_SIZE;
const WHITE = 'white';
const BLACK = 'black';
```

**Spiegazione:**
- Aggiungiamo riferimenti agli elementi DOM che mostrano il conteggio delle pedine
- Definiamo costanti per i colori delle pedine per rendere il codice più leggibile

### JavaScript: Posizionamento iniziale delle pedine

```javascript
// Step 2: Posizionamento delle pedine iniziali
function placePieces() {
    // Posiziona le pedine nere (prime 3 righe)
    for (let row = 0; row < 3; row++) {
        for (let col = 0; col < BOARD_SIZE; col++) {
            // Posiziona le pedine solo sulle caselle scure
            if ((row + col) % 2 !== 0) {
                gameState.board[row][col] = {
                    color: BLACK,
                    isKing: false
                };
            }
        }
    }
    
    // Posiziona le pedine bianche (ultime 3 righe)
    for (let row = BOARD_SIZE - 3; row < BOARD_SIZE; row++) {
        for (let col = 0; col < BOARD_SIZE; col++) {
            // Posiziona le pedine solo sulle caselle scure
            if ((row + col) % 2 !== 0) {
                gameState.board[row][col] = {
                    color: WHITE,
                    isKing: false
                };
            }
        }
    }
}
```

**Spiegazione:**
- La funzione `placePieces()` posiziona le pedine nelle loro posizioni iniziali
- Le pedine nere occupano le prime 3 righe della scacchiera
- Le pedine bianche occupano le ultime 3 righe della scacchiera
- Le pedine sono posizionate solo sulle caselle scure (dove `(row + col) % 2 !== 0`)
- Ogni pedina è rappresentata da un oggetto con proprietà `color` e `isKing`

### JavaScript: Aggiornamento della funzione di disegno

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
            
            // Se c'è una pedina in questa posizione, disegnala
            const piece = gameState.board[row][col];
            if (piece) {
                drawPiece(row, col, piece);
            }
        }
    }
}

// Step 2: Disegno delle pedine
function drawPiece(row, col, piece) {
    const centerX = col * SQUARE_SIZE + SQUARE_SIZE / 2;
    const centerY = row * SQUARE_SIZE + SQUARE_SIZE / 2;
    const radius = SQUARE_SIZE * 0.4; // Dimensione della pedina
    
    // Disegna il cerchio esterno (bordo)
    ctx.beginPath();
    ctx.arc(centerX, centerY, radius, 0, Math.PI * 2);
    ctx.fillStyle = piece.color;
    ctx.fill();
    ctx.strokeStyle = piece.color === WHITE ? '#999' : '#000';
    ctx.lineWidth = 2;
    ctx.stroke();
    
    // Se è una dama, disegna una corona o un cerchio interno
    if (piece.isKing) {
        ctx.beginPath();
        ctx.arc(centerX, centerY, radius * 0.6, 0, Math.PI * 2);
        ctx.fillStyle = piece.color === WHITE ? '#eee' : '#333';
        ctx.fill();
        ctx.strokeStyle = piece.color === WHITE ? '#999' : '#000';
        ctx.stroke();
    }
}
```

**Spiegazione:**
- Aggiorniamo la funzione `drawBoard()` per disegnare anche le pedine
- Aggiungiamo una nuova funzione `drawPiece()` che disegna una singola pedina
- Utilizziamo il metodo `arc()` del contesto Canvas per disegnare cerchi
- Aggiungiamo un indicatore visivo per le dame (anche se in questo step non abbiamo ancora implementato la promozione)

### JavaScript: Aggiornamento delle informazioni di gioco

```javascript
// Aggiorna le informazioni di gioco (conteggio pedine)
function updateGameInfo() {
    let whitePieces = 0;
    let blackPieces = 0;
    
    // Conta le pedine di ogni colore
    for (let row = 0; row < BOARD_SIZE; row++) {
        for (let col = 0; col < BOARD_SIZE; col++) {
            const piece = gameState.board[row][col];
            if (piece) {
                if (piece.color === WHITE) {
                    whitePieces++;
                } else {
                    blackPieces++;
                }
            }
        }
    }
    
    // Aggiorna gli elementi HTML
    whitePiecesElement.textContent = whitePieces;
    blackPiecesElement.textContent = blackPieces;
}
```

**Spiegazione:**
- La funzione `updateGameInfo()` conta il numero di pedine di ogni colore sulla scacchiera
- Aggiorna gli elementi HTML per mostrare questi conteggi
- Questa funzione viene chiamata dopo ogni modifica allo stato del gioco

### JavaScript: Inizializzazione aggiornata

```javascript
// Inizializzazione del gioco
function initGame() {
    initBoard();
    placePieces();
    drawBoard();
    updateGameInfo();
}
```

**Spiegazione:**
- Aggiorniamo la funzione `initGame()` per includere il posizionamento delle pedine e l'aggiornamento delle informazioni di gioco

## Esercizi pratici

1. **Personalizza l'aspetto delle pedine**: Modifica la funzione `drawPiece()` per cambiare l'aspetto delle pedine (ad esempio, usa forme diverse o aggiungi effetti di ombreggiatura).

2. **Aggiungi animazioni**: Implementa un'animazione semplice quando il gioco viene inizializzato, come far apparire le pedine una alla volta.

3. **Implementa una funzione di reset**: Aggiungi un pulsante "Nuova Partita" che reimposta la scacchiera allo stato iniziale.

4. **Visualizzazione alternativa**: Crea una visualizzazione alternativa per le pedine, ad esempio utilizzando immagini invece di forme disegnate con Canvas.

## Approfondimenti

- **Metodi di disegno avanzati in Canvas**: Esplora altri metodi di disegno come `quadraticCurveTo()`, `bezierCurveTo()` e `createRadialGradient()`
- **Manipolazione del DOM**: Approfondisci come JavaScript può interagire con gli elementi HTML
- **Gestione degli eventi**: Studia come implementare interazioni utente tramite eventi come `click`, `mousemove`, ecc.

## Conclusione

In questo secondo step, abbiamo aggiunto le pedine sulla scacchiera e implementato funzioni per disegnarle e tenere traccia del loro numero. Abbiamo imparato a utilizzare metodi avanzati di Canvas per disegnare forme circolari e a manipolare il DOM per aggiornare le informazioni di gioco. Nel prossimo step, implementeremo la selezione delle pedine e la gestione dei turni.