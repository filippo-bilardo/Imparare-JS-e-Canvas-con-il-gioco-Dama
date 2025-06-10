# Guida Didattica: Dama in JavaScript - Step 3: Selezione delle Pedine

## Obiettivo
In questo terzo step, implementeremo la selezione delle pedine e la gestione dei turni nel gioco della dama. I giocatori potranno selezionare le proprie pedine cliccandoci sopra, e il gioco alternerà i turni tra il giocatore bianco e quello nero.

## Concetti teorici da approfondire

### 1. Gestione degli eventi in JavaScript
- **Eventi del mouse**: Come intercettare e gestire i click del mouse
- **Event listeners**: Come associare funzioni agli eventi
- **Coordinate del mouse**: Come convertire le coordinate del mouse in posizioni sulla scacchiera

### 2. Gestione dello stato del gioco
- **Stato corrente**: Come memorizzare e aggiornare lo stato del gioco
- **Turni di gioco**: Come implementare l'alternanza dei turni tra i giocatori
- **Selezione delle pedine**: Come tenere traccia della pedina selezionata

### 3. Feedback visivo
- **Evidenziazione della selezione**: Come fornire un feedback visivo sulla pedina selezionata
- **Indicazione del turno**: Come comunicare visivamente di quale giocatore è il turno

## Analisi del codice

### Aggiornamento della struttura HTML

```html
<div class="game-info">
    <div>Turno: <span id="currentPlayer">Bianco</span></div>
    <div>Pedine Bianche: <span id="whitePieces">12</span></div>
    <div>Pedine Nere: <span id="blackPieces">12</span></div>
</div>

<div class="controls">
    <button id="newGameBtn">Nuova Partita</button>
</div>
```

**Spiegazione:**
- Aggiungiamo un'indicazione del turno corrente
- Aggiungiamo un pulsante per iniziare una nuova partita

### JavaScript: Aggiornamento dello stato del gioco

```javascript
// Riferimenti agli elementi DOM
const canvas = document.getElementById('checkerboard');
const ctx = canvas.getContext('2d');
const currentPlayerElement = document.getElementById('currentPlayer');
const whitePiecesElement = document.getElementById('whitePieces');
const blackPiecesElement = document.getElementById('blackPieces');
const newGameBtn = document.getElementById('newGameBtn');

// Costanti del gioco
const BOARD_SIZE = 8;
const SQUARE_SIZE = canvas.width / BOARD_SIZE;
const WHITE = 'white';
const BLACK = 'black';

// Stato del gioco
let gameState = {
    board: [], // Matrice 8x8 che rappresenta la scacchiera
    currentPlayer: WHITE, // Giocatore corrente (WHITE o BLACK)
    selectedPiece: null, // Pedina selezionata (oggetto con row e col)
    validMoves: [] // Array di mosse valide per la pedina selezionata
};
```

**Spiegazione:**
- Aggiungiamo riferimenti agli elementi DOM per il turno corrente e il pulsante di nuova partita
- Aggiungiamo nuove proprietà allo stato del gioco:
  - `currentPlayer`: indica il giocatore corrente (bianco o nero)
  - `selectedPiece`: memorizza la posizione della pedina selezionata
  - `validMoves`: memorizza le mosse valide per la pedina selezionata

### JavaScript: Gestione dei click sulla scacchiera

```javascript
// Gestisce il click sulla scacchiera
function handleBoardClick(event) {
    const rect = canvas.getBoundingClientRect();
    const x = event.clientX - rect.left;
    const y = event.clientY - rect.top;
    
    const col = Math.floor(x / SQUARE_SIZE);
    const row = Math.floor(y / SQUARE_SIZE);
    
    // Verifica se il click è su una pedina del giocatore corrente
    const piece = gameState.board[row][col];
    if (piece && piece.color === gameState.currentPlayer) {
        // Seleziona la pedina
        gameState.selectedPiece = { row, col };
        gameState.validMoves = getValidMoves(row, col);
        drawBoard();
    }
}

// Aggiungi event listener per il click sulla scacchiera
canvas.addEventListener('click', handleBoardClick);
```

**Spiegazione:**
- Implementiamo la funzione `handleBoardClick` che gestisce i click sulla scacchiera
- Convertiamo le coordinate del mouse in posizioni sulla scacchiera
- Verifichiamo se il click è su una pedina del giocatore corrente
- Se sì, selezioniamo la pedina e calcoliamo le mosse valide
- Aggiungiamo un event listener per il click sulla scacchiera

### JavaScript: Calcolo delle mosse valide

```javascript
// Calcola le mosse valide per una pedina
function getValidMoves(row, col) {
    const piece = gameState.board[row][col];
    if (!piece) return [];
    
    const validMoves = [];
    
    // Direzione di movimento (le pedine bianche si muovono verso l'alto, le nere verso il basso)
    const rowDiff = piece.color === WHITE ? -1 : 1;
    
    // Controlla le mosse diagonali in avanti
    const leftCol = col - 1;
    const rightCol = col + 1;
    const targetRow = row + rowDiff;
    
    // Verifica che la nuova posizione sia all'interno della scacchiera
    if (targetRow >= 0 && targetRow < BOARD_SIZE) {
        // Controlla la mossa diagonale a sinistra
        if (leftCol >= 0 && !gameState.board[targetRow][leftCol]) {
            validMoves.push({ row: targetRow, col: leftCol });
        }
        
        // Controlla la mossa diagonale a destra
        if (rightCol < BOARD_SIZE && !gameState.board[targetRow][rightCol]) {
            validMoves.push({ row: targetRow, col: rightCol });
        }
    }
    
    return validMoves;
}
```

**Spiegazione:**
- Implementiamo la funzione `getValidMoves` che calcola le mosse valide per una pedina
- Le pedine bianche si muovono verso l'alto (righe decrescenti), le nere verso il basso (righe crescenti)
- Controlliamo le mosse diagonali in avanti (a sinistra e a destra)
- Verifichiamo che la nuova posizione sia all'interno della scacchiera e che la casella di destinazione sia vuota

### JavaScript: Aggiornamento della funzione di disegno

```javascript
// Disegna la scacchiera e le pedine
function drawBoard() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    
    for (let row = 0; row < BOARD_SIZE; row++) {
        for (let col = 0; col < BOARD_SIZE; col++) {
            // Determina il colore della casella (alternando bianco e nero)
            const isLightSquare = (row + col) % 2 === 0;
            ctx.fillStyle = isLightSquare ? '#EBECD0' : '#779556';
            
            // Disegna la casella
            ctx.fillRect(col * SQUARE_SIZE, row * SQUARE_SIZE, SQUARE_SIZE, SQUARE_SIZE);
            
            // Se questa è una mossa valida per la pedina selezionata, evidenziala
            if (gameState.selectedPiece && gameState.validMoves.some(move => move.row === row && move.col === col)) {
                ctx.fillStyle = 'rgba(0, 255, 0, 0.3)';
                ctx.fillRect(col * SQUARE_SIZE, row * SQUARE_SIZE, SQUARE_SIZE, SQUARE_SIZE);
            }
            
            // Se c'è una pedina in questa posizione, disegnala
            const piece = gameState.board[row][col];
            if (piece) {
                drawPiece(row, col, piece);
                
                // Se questa è la pedina selezionata, evidenziala
                if (gameState.selectedPiece && gameState.selectedPiece.row === row && gameState.selectedPiece.col === col) {
                    ctx.strokeStyle = 'yellow';
                    ctx.lineWidth = 3;
                    ctx.strokeRect(col * SQUARE_SIZE + 3, row * SQUARE_SIZE + 3, SQUARE_SIZE - 6, SQUARE_SIZE - 6);
                }
            }
        }
    }
}
```

**Spiegazione:**
- Aggiorniamo la funzione `drawBoard` per evidenziare la pedina selezionata e le mosse valide
- Per la pedina selezionata, disegniamo un bordo giallo attorno alla casella
- Per le mosse valide, coloriamo la casella di verde trasparente

### JavaScript: Aggiornamento delle informazioni di gioco

```javascript
// Aggiorna le informazioni di gioco (conteggio pedine e turno corrente)
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
    currentPlayerElement.textContent = gameState.currentPlayer === WHITE ? 'Bianco' : 'Nero';
}
```

**Spiegazione:**
- Aggiorniamo la funzione `updateGameInfo` per mostrare anche il turno corrente
- Aggiorniamo l'elemento HTML che mostra il giocatore corrente

### JavaScript: Funzione di nuova partita

```javascript
// Inizia una nuova partita
function newGame() {
    initBoard();
    placePieces();
    gameState.currentPlayer = WHITE;
    gameState.selectedPiece = null;
    gameState.validMoves = [];
    updateGameInfo();
    drawBoard();
}

// Aggiungi event listener per il pulsante di nuova partita
newGameBtn.addEventListener('click', newGame);
```

**Spiegazione:**
- Implementiamo la funzione `newGame` che reimposta lo stato del gioco
- Reimpostiamo il giocatore corrente a bianco e deseleziona qualsiasi pedina
- Aggiungiamo un event listener per il pulsante di nuova partita

## Esercizi pratici

1. **Miglioramento dell'interfaccia utente**: Aggiungi un'animazione quando una pedina viene selezionata, ad esempio facendola lampeggiare o cambiando gradualmente il colore del bordo.

2. **Indicazione del turno**: Migliora l'indicazione del turno corrente, ad esempio cambiando il colore di sfondo dell'elemento che mostra il turno o aggiungendo un'icona accanto al nome del giocatore.

3. **Deseleziona pedina**: Implementa la possibilità di deselezionare una pedina cliccandoci sopra una seconda volta.

4. **Accessibilità da tastiera**: Aggiungi la possibilità di selezionare le pedine e muoverle utilizzando la tastiera, per migliorare l'accessibilità del gioco.

## Approfondimenti

- **Gestione avanzata degli eventi**: Studia come gestire eventi più complessi come il drag and drop
- **Pattern di progettazione**: Approfondisci il pattern Observer per implementare in modo più elegante l'aggiornamento dell'interfaccia utente
- **Accessibilità**: Studia le linee guida WCAG per rendere il gioco accessibile a tutti gli utenti

## Conclusione

In questo terzo step, abbiamo implementato la selezione delle pedine e la gestione dei turni nel gioco della dama. Abbiamo imparato a gestire gli eventi del mouse, a convertire le coordinate del mouse in posizioni sulla scacchiera, a calcolare le mosse valide per una pedina e a fornire un feedback visivo all'utente. Nel prossimo step, implementeremo lo spostamento delle pedine e le regole di base del gioco.