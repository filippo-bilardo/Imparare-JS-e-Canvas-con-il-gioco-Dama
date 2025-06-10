# Guida Didattica: Dama in JavaScript - Step 5: Catture

## Obiettivo
In questo quinto step, implementeremo le regole di cattura nel gioco della dama. I giocatori potranno catturare le pedine avversarie saltandole quando sono adiacenti a una propria pedina e c'è uno spazio libero dietro di esse.

## Concetti teorici da approfondire

### 1. Algoritmi di cattura
- **Rilevamento delle catture possibili**: Come identificare le pedine avversarie che possono essere catturate
- **Implementazione del salto**: Come gestire il movimento di salto e la rimozione della pedina catturata

### 2. Gestione dello storico delle mosse
- **Stack di mosse**: Come implementare una struttura dati per memorizzare le mosse precedenti
- **Funzionalità di annullamento**: Come implementare la possibilità di annullare l'ultima mossa

### 3. Regole avanzate del gioco
- **Catture multiple**: Come gestire la possibilità di catture consecutive
- **Priorità di cattura**: Come implementare la regola che obbliga a catturare quando possibile

## Analisi del codice

### Aggiornamento della struttura HTML

```html
<div class="controls">
    <button id="newGameBtn">Nuova Partita</button>
    <button id="undoBtn">Annulla Mossa</button>
</div>
```

**Spiegazione:**
- Aggiungiamo un pulsante "Annulla Mossa" che permetterà ai giocatori di annullare l'ultima mossa effettuata

### JavaScript: Aggiornamento dello stato del gioco

```javascript
// Stato del gioco
let gameState = {
    board: [], // Matrice 8x8 che rappresenta la scacchiera
    currentPlayer: WHITE, // Giocatore corrente (WHITE o BLACK)
    selectedPiece: null, // Pedina selezionata (oggetto con row e col)
    validMoves: [], // Array di mosse valide per la pedina selezionata
    moveHistory: [], // Storico delle mosse per la funzionalità di annullamento
    capturedPieces: [] // Pedine catturate nell'ultima mossa (per catture multiple)
};
```

**Spiegazione:**
- Aggiungiamo due nuove proprietà allo stato del gioco:
  - `moveHistory`: un array che memorizza lo storico delle mosse per implementare la funzionalità di annullamento
  - `capturedPieces`: un array che memorizza le pedine catturate nell'ultima mossa (utile per catture multiple)

### JavaScript: Calcolo delle mosse di cattura

```javascript
// Calcola le mosse valide per una pedina, incluse le catture
function getValidMoves(row, col) {
    const piece = gameState.board[row][col];
    if (!piece) return [];
    
    const validMoves = [];
    const captureMoves = []; // Mosse di cattura separate
    
    // Direzione di movimento (le pedine bianche si muovono verso l'alto, le nere verso il basso)
    const directions = [];
    
    // Le pedine normali possono muoversi solo in avanti
    if (piece.color === WHITE) {
        directions.push({ rowDiff: -1, colDiff: -1 }); // In alto a sinistra
        directions.push({ rowDiff: -1, colDiff: 1 });  // In alto a destra
    } else {
        directions.push({ rowDiff: 1, colDiff: -1 });  // In basso a sinistra
        directions.push({ rowDiff: 1, colDiff: 1 });   // In basso a destra
    }
    
    // Controlla le mosse di movimento normale
    for (const dir of directions) {
        const newRow = row + dir.rowDiff;
        const newCol = col + dir.colDiff;
        
        // Verifica che la nuova posizione sia all'interno della scacchiera
        if (newRow >= 0 && newRow < BOARD_SIZE && newCol >= 0 && newCol < BOARD_SIZE) {
            // Verifica che la casella di destinazione sia vuota
            if (!gameState.board[newRow][newCol]) {
                validMoves.push({ row: newRow, col: newCol, captures: [] });
            }
            // Verifica se è possibile catturare una pedina avversaria
            else if (gameState.board[newRow][newCol].color !== piece.color) {
                const jumpRow = newRow + dir.rowDiff;
                const jumpCol = newCol + dir.colDiff;
                
                // Verifica che la casella dopo la pedina avversaria sia all'interno della scacchiera e vuota
                if (jumpRow >= 0 && jumpRow < BOARD_SIZE && jumpCol >= 0 && jumpCol < BOARD_SIZE && 
                    !gameState.board[jumpRow][jumpCol]) {
                    captureMoves.push({ 
                        row: jumpRow, 
                        col: jumpCol, 
                        captures: [{ row: newRow, col: newCol }] 
                    });
                }
            }
        }
    }
    
    // Se ci sono mosse di cattura, restituisci solo quelle (regola obbligatoria)
    if (captureMoves.length > 0) {
        return captureMoves;
    }
    
    return validMoves;
}
```

**Spiegazione:**
- Aggiorniamo la funzione `getValidMoves` per includere le mosse di cattura
- Aggiungiamo una proprietà `captures` a ogni mossa, che contiene un array di pedine catturate
- Verifichiamo se è possibile catturare una pedina avversaria saltandola
- Implementiamo la regola che obbliga a catturare quando possibile (se ci sono mosse di cattura, restituiamo solo quelle)

### JavaScript: Esecuzione delle mosse con cattura

```javascript
// Muove una pedina da una posizione a un'altra, gestendo anche le catture
function movePiece(fromRow, fromCol, toRow, toCol, captures = []) {
    // Salva lo stato corrente per la funzionalità di annullamento
    const currentState = {
        board: JSON.parse(JSON.stringify(gameState.board)),
        currentPlayer: gameState.currentPlayer,
        captures: [...captures]
    };
    gameState.moveHistory.push(currentState);
    
    // Sposta la pedina