# Guida Didattica: Dama in JavaScript - Step 4: Mosse Base

## Obiettivo
In questo quarto step, implementeremo le mosse base nel gioco della dama. I giocatori potranno selezionare una pedina e poi spostarla in una delle caselle valide. Implementeremo anche il cambio di turno dopo ogni mossa.

## Concetti teorici da approfondire

### 1. Gestione dello stato del gioco
- **Transizioni di stato**: Come gestire i cambiamenti di stato del gioco dopo una mossa
- **Validazione delle mosse**: Come verificare che una mossa sia valida secondo le regole del gioco

### 2. Interazione utente avanzata
- **Selezione e movimento**: Come implementare un sistema di selezione e movimento in due fasi
- **Feedback visivo interattivo**: Come fornire feedback visivo durante l'interazione

### 3. Algoritmi di movimento
- **Calcolo delle posizioni valide**: Come determinare le posizioni in cui una pedina può muoversi
- **Implementazione delle regole di movimento**: Come codificare le regole di movimento della dama

## Analisi del codice

### JavaScript: Aggiornamento della gestione dei click

```javascript
// Gestisce il click sulla scacchiera
function handleBoardClick(event) {
    const rect = canvas.getBoundingClientRect();
    const x = event.clientX - rect.left;
    const y = event.clientY - rect.top;
    
    const col = Math.floor(x / SQUARE_SIZE);
    const row = Math.floor(y / SQUARE_SIZE);
    
    // Se c'è una pedina selezionata, verifica se il click è su una mossa valida
    if (gameState.selectedPiece) {
        const validMove = gameState.validMoves.find(move => move.row === row && move.col === col);
        
        if (validMove) {
            // Esegui la mossa
            const { row: fromRow, col: fromCol } = gameState.selectedPiece;
            movePiece(fromRow, fromCol, row, col);
            return;
        }
    }
    
    // Verifica se il click è su una pedina del giocatore corrente
    const piece = gameState.board[row][col];
    if (piece && piece.color === gameState.currentPlayer) {
        // Seleziona la pedina
        gameState.selectedPiece = { row, col };
        gameState.validMoves = getValidMoves(row, col);
        drawBoard();
    }
}
```

**Spiegazione:**
- Aggiorniamo la funzione `handleBoardClick` per gestire sia la selezione che il movimento
- Se c'è già una pedina selezionata e il click è su una mossa valida, eseguiamo la mossa
- Altrimenti, se il click è su una pedina del giocatore corrente, la selezioniamo
- Questo implementa un sistema di interazione in due fasi: prima selezione, poi movimento

### JavaScript: Implementazione del movimento delle pedine

```javascript
// Muove una pedina da una posizione a un'altra
function movePiece(fromRow, fromCol, toRow, toCol) {
    // Sposta la pedina
    const piece = gameState.board[fromRow][fromCol];
    gameState.board[toRow][toCol] = piece;
    gameState.board[fromRow][fromCol] = null;
    
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
- Implementiamo la funzione `movePiece` che sposta una pedina da una posizione a un'altra
- Aggiorniamo la matrice `board` per riflettere il movimento
- Cambiamo il turno al giocatore opposto
- Aggiorniamo le informazioni di gioco e ridisegniamo la scacchiera
- Deseleziona la pedina dopo il movimento

### JavaScript: Miglioramento del calcolo delle mosse valide

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
- Manteniamo la funzione `getValidMoves` che calcola le mosse valide per una pedina
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
- Manteniamo la funzione `drawBoard` che disegna la scacchiera e le pedine
- Evidenziamo la pedina selezionata con un bordo giallo
- Evidenziamo le mosse valide con un colore verde trasparente
- Questo fornisce un feedback visivo chiaro all'utente su quali mosse sono disponibili

## Esercizi pratici

1. **Animazione del movimento**: Implementa un'animazione fluida quando una pedina viene spostata, invece di un cambiamento istantaneo.

2. **Evidenziazione del percorso**: Quando una pedina viene selezionata, evidenzia il percorso che seguirebbe per raggiungere ciascuna delle destinazioni valide.

3. **Suggerimenti**: Aggiungi un pulsante "Suggerimento" che evidenzia una mossa consigliata per il giocatore corrente.

4. **Cronologia delle mosse**: Implementa una visualizzazione della cronologia delle mosse effettuate durante la partita.

## Approfondimenti

- **Algoritmi di pathfinding**: Studia algoritmi come A* o Dijkstra per implementare percorsi ottimali in giochi più complessi
- **Animazioni avanzate**: Approfondisci l'uso di `requestAnimationFrame` per creare animazioni fluide in Canvas
- **Macchina a stati finiti**: Studia come implementare una macchina a stati finiti per gestire in modo più elegante le diverse fasi del gioco

## Conclusione

In questo quarto step, abbiamo implementato le mosse base nel gioco della dama. Abbiamo imparato a gestire un sistema di interazione in due fasi (selezione e movimento), a calcolare le posizioni valide per una pedina e a fornire un feedback visivo durante l'interazione. Nel prossimo step, implementeremo le regole di cattura, che permetteranno ai giocatori di catturare le pedine avversarie.