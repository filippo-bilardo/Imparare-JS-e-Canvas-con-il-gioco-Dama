# Guida Didattica: Dama in JavaScript - Step 6: Dame

## Obiettivo
In questo sesto step, implementeremo la promozione delle pedine a dame quando raggiungono il lato opposto della scacchiera. Le dame avranno la capacità di muoversi in tutte le direzioni diagonali, aumentando le possibilità strategiche del gioco.

## Concetti teorici da approfondire

### 1. Regole avanzate del gioco
- **Promozione a dama**: Come implementare la regola che promuove una pedina a dama quando raggiunge il lato opposto
- **Movimento delle dame**: Come implementare le regole di movimento specifiche per le dame

### 2. Rappresentazione visiva dello stato
- **Differenziazione visiva**: Come distinguere visivamente le dame dalle pedine normali
- **Feedback sulla promozione**: Come comunicare all'utente che una pedina è stata promossa a dama

### 3. Estensione di funzionalità esistenti
- **Polimorfismo**: Come gestire comportamenti diversi per oggetti simili (pedine normali vs dame)
- **Riutilizzo del codice**: Come estendere le funzioni esistenti per supportare nuove funzionalità

## Analisi del codice

### JavaScript: Aggiornamento della funzione di movimento

```javascript
// Muove una pedina da una posizione a un'altra, gestendo anche le catture e la promozione a dama
function movePiece(fromRow, fromCol, toRow, toCol, captures = []) {
    // Salva lo stato corrente per la funzionalità di annullamento
    const currentState = {
        board: JSON.parse(JSON.stringify(gameState.board)),
        currentPlayer: gameState.currentPlayer,
        captures: [...captures]
    };
    gameState.moveHistory.push(currentState);
    
    // Sposta la pedina
    const piece = gameState.board[fromRow][fromCol];
    gameState.board[toRow][toCol] = piece;
    gameState.board[fromRow][fromCol] = null;
    
    // Rimuovi le pedine catturate
    for (const capture of captures) {
        gameState.board[capture.row][capture.col] = null;
    }
    
    // Controlla se la pedina deve essere promossa a dama
    if (!piece.isKing) {
        if ((piece.color === WHITE && toRow === 0) || (piece.color === BLACK && toRow === BOARD_SIZE - 1)) {
            piece.isKing = true;
        }
    }
    
    // Cambia il turno
    gameState.currentPlayer = gameState.currentPlayer === WHITE ? BLACK : WHITE;
    
    // Aggiorna le informazioni di gioco
    updateGameInfo();
    
    // Deseleziona la pedina
    gameState.selectedPiece = null;
    gameState.validMoves = [];
    
    // Ridisegna la scacchiera
    drawBoard();
}
```

**Spiegazione:**
- Aggiorniamo la funzione `movePiece` per gestire la promozione a dama
- Verifichiamo se una pedina ha raggiunto il lato opposto della scacchiera (riga 0 per le bianche, riga 7 per le nere)
- Se sì, impostiamo la proprietà `isKing` a `true`

### JavaScript: Aggiornamento del calcolo delle mosse valide

```javascript
// Calcola le mosse valide per una pedina, incluse le catture
function getValidMoves(row, col) {
    const piece = gameState.board[row][col];
    if (!piece) return [];
    
    const validMoves = [];
    const captureMoves = []; // Mosse di cattura separate
    
    // Direzione di movimento
    const directions = [];
    
    // Le dame possono muoversi in tutte le direzioni diagonali
    if (piece.isKing) {
        directions.push({ rowDiff: -1, colDiff: -1 }); // In alto a sinistra
        directions.push({ rowDiff: -1, colDiff: 1 });  // In alto a destra
        directions.push({ rowDiff: 1, colDiff: -1 });  // In basso a sinistra
        directions.push({ rowDiff: 1, colDiff: 1 });   // In basso a destra
    }
    // Le pedine normali possono muoversi solo in avanti
    else if (piece.color === WHITE) {
        directions.push({ rowDiff: -1, colDiff: -1 }); // In alto a sinistra
        directions.push({ rowDiff: -1, colDiff: 1 });  // In alto a destra
    } else {
        directions.push({ rowDiff: 1, colDiff: -1 });  // In basso a sinistra
        directions.push({ rowDiff: 1, colDiff: 1 });   // In basso a destra
    }
    
    // Controlla le mosse in ogni direzione
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
- Aggiorniamo la funzione `getValidMoves` per gestire le mosse delle dame
- Per le dame, aggiungiamo tutte e quattro le direzioni diagonali (in alto a sinistra, in alto a destra, in basso a sinistra, in basso a destra)
- Per le pedine normali, manteniamo il comportamento precedente (solo in avanti)
- Il resto della logica rimane invariato: verifichiamo se è possibile muoversi in una casella vuota o catturare una pedina avversaria

### JavaScript: Aggiornamento della funzione di disegno delle pedine

```javascript
// Disegna una pedina
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
- La funzione `drawPiece` è stata aggiornata per distinguere visivamente le dame dalle pedine normali
- Per le dame, disegniamo un cerchio interno di colore leggermente diverso
- Questo fornisce un feedback visivo chiaro all'utente su quali pedine sono state promosse a dame

## Esercizi pratici

1. **Effetto di promozione**: Implementa un effetto visivo o sonoro quando una pedina viene promossa a dama, come un'animazione o un suono.

2. **Statistiche delle dame**: Aggiungi un contatore che mostra il numero di dame per ciascun giocatore.

3. **Mosse speciali per le dame**: Implementa regole speciali per le dame, come la possibilità di muoversi di più caselle in una singola mossa (come negli scacchi).

4. **Personalizzazione delle dame**: Aggiungi la possibilità di personalizzare l'aspetto delle dame, ad esempio utilizzando simboli o immagini diverse.

## Approfondimenti

- **Pattern di progettazione**: Studia il pattern Strategy per implementare in modo più elegante i diversi comportamenti delle pedine normali e delle dame
- **Animazioni avanzate**: Approfondisci come implementare animazioni fluide utilizzando requestAnimationFrame
- **Intelligenza artificiale**: Studia come implementare un'IA che tenga conto del valore strategico delle dame

## Conclusione

In questo sesto step, abbiamo implementato la promozione delle pedine a dame quando raggiungono il lato opposto della scacchiera. Abbiamo imparato a gestire comportamenti diversi per le pedine normali e le dame, a distinguerle visivamente e a implementare le regole di movimento specifiche per le dame. Nel prossimo step, implementeremo il controllo della fine del gioco e la gestione della vittoria.