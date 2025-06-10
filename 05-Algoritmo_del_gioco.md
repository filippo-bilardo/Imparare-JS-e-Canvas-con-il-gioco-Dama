# Algoritmo del Gioco della Dama: Guida Approfondita

## Introduzione

L'algoritmo di un gioco è l'insieme di regole e procedure che ne definiscono il funzionamento e la logica. Nel caso della dama, l'algoritmo comprende le regole del gioco, la gestione dei turni, il movimento delle pedine, le catture e le condizioni di vittoria. In questa guida, esploreremo in dettaglio l'algoritmo del gioco della dama e come implementarlo in JavaScript.

## Regole del Gioco della Dama

Prima di implementare l'algoritmo, è importante comprendere le regole del gioco:

### Configurazione Iniziale

- La dama si gioca su una scacchiera 8×8 con caselle alternate di due colori
- Ogni giocatore inizia con 12 pedine posizionate sulle caselle scure delle prime tre righe dal proprio lato
- Le pedine si muovono solo sulle caselle scure

### Movimento delle Pedine

- Le pedine normali si muovono in diagonale di una casella in avanti (verso il lato avversario)
- Le dame (pedine promosse) possono muoversi in diagonale di una casella in qualsiasi direzione
- Una pedina viene promossa a dama quando raggiunge l'ultima riga del lato avversario

### Catture

- Una pedina può catturare una pedina avversaria saltandola diagonalmente e atterrando su una casella vuota
- Dopo una cattura, se è possibile effettuare un'altra cattura con la stessa pedina, il giocatore deve continuare a catturare (cattura multipla)
- Le dame possono catturare in qualsiasi direzione diagonale

### Fine del Gioco

- Un giocatore vince quando l'avversario non ha più pedine o non può più muovere
- Il gioco può terminare in pareggio se entrambi i giocatori concordano o in situazioni di stallo

## Struttura dell'Algoritmo

L'algoritmo del gioco della dama può essere suddiviso in diverse componenti:

1. **Inizializzazione del gioco**: creazione della scacchiera e posizionamento delle pedine
2. **Gestione dei turni**: alternanza tra i giocatori
3. **Selezione e movimento delle pedine**: gestione dell'input dell'utente e validazione delle mosse
4. **Logica di cattura**: implementazione delle regole di cattura, incluse le catture multiple
5. **Promozione a dama**: gestione della promozione quando una pedina raggiunge l'ultima riga
6. **Verifica delle condizioni di fine gioco**: controllo della vittoria, sconfitta o pareggio

## Implementazione dell'Algoritmo in JavaScript

### 1. Inizializzazione del Gioco

```javascript
// Costanti
const BOARD_SIZE = 8;
const EMPTY = null;
const WHITE = 'white';
const BLACK = 'black';

// Stato del gioco
const gameState = {
    board: [], // Scacchiera
    currentPlayer: WHITE, // Giocatore corrente
    selectedPiece: null, // Pedina selezionata
    possibleMoves: [], // Mosse possibili
    captureRequired: false, // Indica se è obbligatorio catturare
    gameOver: false, // Indica se il gioco è finito
    winner: null // Vincitore
};

// Inizializzazione del gioco
function initGame() {
    // Crea una scacchiera vuota
    gameState.board = [];
    for (let row = 0; row < BOARD_SIZE; row++) {
        gameState.board[row] = [];
        for (let col = 0; col < BOARD_SIZE; col++) {
            gameState.board[row][col] = EMPTY;
        }
    }
    
    // Posiziona le pedine iniziali
    setupPieces();
    
    // Imposta il giocatore iniziale
    gameState.currentPlayer = WHITE;
    
    // Resetta gli altri stati
    gameState.selectedPiece = null;
    gameState.possibleMoves = [];
    gameState.captureRequired = false;
    gameState.gameOver = false;
    gameState.winner = null;
    
    // Verifica se ci sono catture obbligatorie all'inizio
    checkForRequiredCaptures();
    
    // Disegna la scacchiera
    drawBoard();
}

// Posizionamento iniziale delle pedine
function setupPieces() {
    // Posiziona le pedine nere (prime tre righe)
    for (let row = 0; row < 3; row++) {
        for (let col = 0; col < BOARD_SIZE; col++) {
            // Le pedine vanno solo sulle caselle scure (somma di riga e colonna dispari)
            if ((row + col) % 2 === 1) {
                gameState.board[row][col] = { color: BLACK, isKing: false };
            }
        }
    }
    
    // Posiziona le pedine bianche (ultime tre righe)
    for (let row = BOARD_SIZE - 3; row < BOARD_SIZE; row++) {
        for (let col = 0; col < BOARD_SIZE; col++) {
            // Le pedine vanno solo sulle caselle scure (somma di riga e colonna dispari)
            if ((row + col) % 2 === 1) {
                gameState.board[row][col] = { color: WHITE, isKing: false };
            }
        }
    }
}
```

### 2. Gestione dei Turni

```javascript
// Cambio del turno
function switchPlayer() {
    gameState.currentPlayer = gameState.currentPlayer === WHITE ? BLACK : WHITE;
    
    // Verifica se ci sono catture obbligatorie per il nuovo giocatore
    checkForRequiredCaptures();
    
    // Verifica se il gioco è finito
    checkGameOver();
}

// Verifica se ci sono catture obbligatorie
function checkForRequiredCaptures() {
    gameState.captureRequired = false;
    
    // Controlla tutte le pedine del giocatore corrente
    for (let row = 0; row < BOARD_SIZE; row++) {
        for (let col = 0; col < BOARD_SIZE; col++) {
            const piece = gameState.board[row][col];
            if (piece && piece.color === gameState.currentPlayer) {
                // Calcola le mosse di cattura possibili per questa pedina
                const captureMoves = calculateCaptureMoves(row, col);
                if (captureMoves.length > 0) {
                    gameState.captureRequired = true;
                    return; // Basta trovare una pedina con catture possibili
                }
            }
        }
    }
}
```

### 3. Selezione e Movimento delle Pedine

```javascript
// Gestione del click su una casella
function handleSquareClick(row, col) {
    // Se il gioco è finito, ignora i click
    if (gameState.gameOver) return;
    
    const piece = gameState.board[row][col];
    
    // Caso 1: Nessuna pedina selezionata e la casella contiene una pedina del giocatore corrente
    if (!gameState.selectedPiece && piece && piece.color === gameState.currentPlayer) {
        selectPiece(row, col);
    }
    // Caso 2: Pedina selezionata e la casella è una mossa valida
    else if (gameState.selectedPiece && isPossibleMove(row, col)) {
        movePiece(gameState.selectedPiece.row, gameState.selectedPiece.col, row, col);
    }
    // Caso 3: Pedina selezionata ma la casella non è una mossa valida
    else if (gameState.selectedPiece) {
        // Se si clicca su un'altra pedina del giocatore corrente, cambia selezione
        if (piece && piece.color === gameState.currentPlayer) {
            selectPiece(row, col);
        } else {
            // Altrimenti, deseleziona
            deselectPiece();
        }
    }
    
    // Aggiorna la visualizzazione
    drawBoard();
}

// Selezione di una pedina
function selectPiece(row, col) {
    const piece = gameState.board[row][col];
    
    // Verifica se la pedina appartiene al giocatore corrente
    if (piece && piece.color === gameState.currentPlayer) {
        gameState.selectedPiece = { row, col };
        
        // Calcola le mosse possibili
        if (gameState.captureRequired) {
            // Se ci sono catture obbligatorie, calcola solo le mosse di cattura
            gameState.possibleMoves = calculateCaptureMoves(row, col);
            
            // Se questa pedina non ha mosse di cattura, deselezionala
            if (gameState.possibleMoves.length === 0) {
                gameState.selectedPiece = null;
            }
        } else {
            // Altrimenti, calcola tutte le mosse possibili
            gameState.possibleMoves = calculatePossibleMoves(row, col);
        }
    }
}

// Deselezione della pedina
function deselectPiece() {
    gameState.selectedPiece = null;
    gameState.possibleMoves = [];
}

// Verifica se una mossa è possibile
function isPossibleMove(row, col) {
    return gameState.possibleMoves.some(move => move.row === row && move.col === col);
}
```

### 4. Logica di Movimento e Cattura

```javascript
// Calcolo delle mosse possibili per una pedina
function calculatePossibleMoves(row, col) {
    const piece = gameState.board[row][col];
    if (!piece) return [];
    
    const moves = [];
    
    // Calcola le mosse di cattura
    const captureMoves = calculateCaptureMoves(row, col);
    
    // Se ci sono mosse di cattura, sono le uniche mosse possibili
    if (captureMoves.length > 0) {
        return captureMoves;
    }
    
    // Altrimenti, calcola le mosse normali
    
    // Direzione di movimento (le pedine normali si muovono solo in avanti)
    const directions = [];
    
    if (piece.isKing) {
        // Le dame possono muoversi in tutte le direzioni
        directions.push({ rowDir: -1, colDir: -1 }); // Alto-sinistra
        directions.push({ rowDir: -1, colDir: 1 });  // Alto-destra
        directions.push({ rowDir: 1, colDir: -1 });  // Basso-sinistra
        directions.push({ rowDir: 1, colDir: 1 });   // Basso-destra
    } else if (piece.color === WHITE) {
        // Le pedine bianche si muovono verso l'alto
        directions.push({ rowDir: -1, colDir: -1 }); // Alto-sinistra
        directions.push({ rowDir: -1, colDir: 1 });  // Alto-destra
    } else {
        // Le pedine nere si muovono verso il basso
        directions.push({ rowDir: 1, colDir: -1 });  // Basso-sinistra
        directions.push({ rowDir: 1, colDir: 1 });   // Basso-destra
    }
    
    // Controlla ogni direzione
    for (const dir of directions) {
        const newRow = row + dir.rowDir;
        const newCol = col + dir.colDir;
        
        // Verifica se la nuova posizione è valida e vuota
        if (isValidPosition(newRow, newCol) && gameState.board[newRow][newCol] === EMPTY) {
            moves.push({ row: newRow, col: newCol, isCapture: false });
        }
    }
    
    return moves;
}

// Calcolo delle mosse di cattura possibili
function calculateCaptureMoves(row, col) {
    const piece = gameState.board[row][col];
    if (!piece) return [];
    
    const captureMoves = [];
    
    // Direzioni di movimento per le catture
    const directions = [];
    
    if (piece.isKing) {
        // Le dame possono catturare in tutte le direzioni
        directions.push({ rowDir: -1, colDir: -1 }); // Alto-sinistra
        directions.push({ rowDir: -1, colDir: 1 });  // Alto-destra
        directions.push({ rowDir: 1, colDir: -1 });  // Basso-sinistra
        directions.push({ rowDir: 1, colDir: 1 });   // Basso-destra
    } else if (piece.color === WHITE) {
        // Le pedine bianche possono catturare in diagonale verso l'alto e verso il basso
        directions.push({ rowDir: -1, colDir: -1 }); // Alto-sinistra
        directions.push({ rowDir: -1, colDir: 1 });  // Alto-destra
        directions.push({ rowDir: 1, colDir: -1 });  // Basso-sinistra
        directions.push({ rowDir: 1, colDir: 1 });   // Basso-destra
    } else {
        // Le pedine nere possono catturare in diagonale verso l'alto e verso il basso
        directions.push({ rowDir: -1, colDir: -1 }); // Alto-sinistra
        directions.push({ rowDir: -1, colDir: 1 });  // Alto-destra
        directions.push({ rowDir: 1, colDir: -1 });  // Basso-sinistra
        directions.push({ rowDir: 1, colDir: 1 });   // Basso-destra
    }
    
    // Controlla ogni direzione
    for (const dir of directions) {
        const adjacentRow = row + dir.rowDir;
        const adjacentCol = col + dir.colDir;
        
        // Verifica se c'è una pedina avversaria adiacente
        if (isValidPosition(adjacentRow, adjacentCol) && 
            gameState.board[adjacentRow][adjacentCol] && 
            gameState.board[adjacentRow][adjacentCol].color !== piece.color) {
            
            // Verifica se la casella dopo la pedina avversaria è vuota
            const landingRow = adjacentRow + dir.rowDir;
            const landingCol = adjacentCol + dir.colDir;
            
            if (isValidPosition(landingRow, landingCol) && 
                gameState.board[landingRow][landingCol] === EMPTY) {
                
                captureMoves.push({
                    row: landingRow,
                    col: landingCol,
                    isCapture: true,
                    capturedRow: adjacentRow,
                    capturedCol: adjacentCol
                });
            }
        }
    }
    
    return captureMoves;
}

// Movimento di una pedina
function movePiece(fromRow, fromCol, toRow, toCol) {
    const piece = gameState.board[fromRow][fromCol];
    const move = gameState.possibleMoves.find(m => m.row === toRow && m.col === toCol);
    
    if (!piece || !move) return;
    
    // Esegui la mossa
    gameState.board[toRow][toCol] = piece;
    gameState.board[fromRow][fromCol] = EMPTY;
    
    // Se è una mossa di cattura
    let continueCaptureWithSamePiece = false;
    
    if (move.isCapture) {
        // Rimuovi la pedina catturata
        gameState.board[move.capturedRow][move.capturedCol] = EMPTY;
        
        // Verifica se ci sono altre catture possibili con la stessa pedina
        const furtherCaptures = calculateCaptureMoves(toRow, toCol);
        
        if (furtherCaptures.length > 0) {
            // Seleziona automaticamente la pedina per continuare la cattura
            gameState.selectedPiece = { row: toRow, col: toCol };
            gameState.possibleMoves = furtherCaptures;
            continueCaptureWithSamePiece = true;
        }
    }
    
    // Verifica se la pedina deve essere promossa a dama
    if (!piece.isKing && 
        ((piece.color === WHITE && toRow === 0) || 
         (piece.color === BLACK && toRow === BOARD_SIZE - 1))) {
        piece.isKing = true;
    }
    
    // Se non ci sono ulteriori catture con la stessa pedina, passa il turno
    if (!continueCaptureWithSamePiece) {
        deselectPiece();
        switchPlayer();
    }
}

// Verifica se una posizione è valida
function isValidPosition(row, col) {
    return row >= 0 && row < BOARD_SIZE && col >= 0 && col < BOARD_SIZE;
}
```

### 5. Verifica delle Condizioni di Fine Gioco

```javascript
// Verifica se il gioco è finito
function checkGameOver() {
    // Conta le pedine di ciascun giocatore
    let whitePieces = 0;
    let blackPieces = 0;
    
    // Verifica se il giocatore corrente ha mosse disponibili
    let currentPlayerHasMoves = false;
    
    for (let row = 0; row < BOARD_SIZE; row++) {
        for (let col = 0; col < BOARD_SIZE; col++) {
            const piece = gameState.board[row][col];
            
            if (piece) {
                // Conta le pedine
                if (piece.color === WHITE) {
                    whitePieces++;
                } else {
                    blackPieces++;
                }
                
                // Verifica se il giocatore corrente ha mosse disponibili
                if (piece.color === gameState.currentPlayer) {
                    const moves = calculatePossibleMoves(row, col);
                    if (moves.length > 0) {
                        currentPlayerHasMoves = true;
                    }
                }
            }
        }
    }
    
    // Condizioni di fine gioco
    if (whitePieces === 0) {
        // Il nero vince
        gameState.gameOver = true;
        gameState.winner = BLACK;
    } else if (blackPieces === 0) {
        // Il bianco vince
        gameState.gameOver = true;
        gameState.winner = WHITE;
    } else if (!currentPlayerHasMoves) {
        // Il giocatore corrente non può muovere, quindi perde
        gameState.gameOver = true;
        gameState.winner = gameState.currentPlayer === WHITE ? BLACK : WHITE;
    }
    
    // Se il gioco è finito, mostra un messaggio
    if (gameState.gameOver) {
        const winnerText = gameState.winner === WHITE ? 'Bianco' : 'Nero';
        console.log(`Il gioco è finito! Il vincitore è: ${winnerText}`);
        // Qui si potrebbe mostrare un messaggio all'utente
    }
}
```

## Ottimizzazione dell'Algoritmo

L'algoritmo presentato può essere ottimizzato in vari modi:

### 1. Memorizzazione delle Mosse Valide

Invece di ricalcolare le mosse valide ad ogni click, è possibile memorizzarle e aggiornarle solo quando necessario:

```javascript
// All'inizio del gioco, calcola tutte le mosse valide per ogni pedina
function calculateAllValidMoves() {
    const allMoves = {};
    
    for (let row = 0; row < BOARD_SIZE; row++) {
        for (let col = 0; col < BOARD_SIZE; col++) {
            const piece = gameState.board[row][col];
            if (piece && piece.color === gameState.currentPlayer) {
                const key = `${row},${col}`;
                allMoves[key] = calculatePossibleMoves(row, col);
            }
        }
    }
    
    return allMoves;
}
```

### 2. Algoritmo di Ricerca per l'IA

Per implementare un'intelligenza artificiale per il gioco, è possibile utilizzare algoritmi di ricerca come Minimax con potatura Alpha-Beta:

```javascript
// Funzione principale Minimax con potatura Alpha-Beta
function minimaxAlphaBeta(board, depth, alpha, beta, isMaximizingPlayer) {
    // Caso base: profondità zero o nodo terminale
    if (depth === 0 || isGameOver(board)) {
        return evaluateBoard(board);
    }
    
    if (isMaximizingPlayer) {
        let maxEval = -Infinity;
        const moves = getAllPossibleMoves(board, WHITE);
        
        for (const move of moves) {
            const newBoard = makeMove(board, move);
            const evaluation = minimaxAlphaBeta(newBoard, depth - 1, alpha, beta, false);
            maxEval = Math.max(maxEval, evaluation);
            alpha = Math.max(alpha, evaluation);
            if (beta <= alpha) {
                break; // Potatura Beta
            }
        }
        
        return maxEval;
    } else {
        let minEval = Infinity;
        const moves = getAllPossibleMoves(board, BLACK);
        
        for (const move of moves) {
            const newBoard = makeMove(board, move);
            const evaluation = minimaxAlphaBeta(newBoard, depth - 1, alpha, beta, true);
            minEval = Math.min(minEval, evaluation);
            beta = Math.min(beta, evaluation);
            if (beta <= alpha) {
                break; // Potatura Alpha
            }
        }
        
        return minEval;
    }
}

// Funzione di valutazione della scacchiera
function evaluateBoard(board) {
    let score = 0;
    
    for (let row = 0; row < BOARD_SIZE; row++) {
        for (let col = 0; col < BOARD_SIZE; col++) {
            const piece = board[row][col];
            if (piece) {
                // Valore base delle pedine
                const pieceValue = piece.isKing ? 3 : 1;
                
                // Aggiungi o sottrai il valore in base al colore
                if (piece.color === WHITE) {
                    score += pieceValue;
                    
                    // Bonus per le pedine avanzate
                    score += (BOARD_SIZE - 1 - row) * 0.1;
                } else {
                    score -= pieceValue;
                    
                    // Bonus per le pedine avanzate
                    score -= row * 0.1;
                }
            }
        }
    }
    
    return score;
}
```

## Gestione dell'Interfaccia Utente

L'algoritmo del gioco deve essere integrato con l'interfaccia utente per creare un'esperienza di gioco completa:

```javascript
// Disegno della scacchiera
function drawBoard() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    
    // Disegna le caselle
    for (let row = 0; row < BOARD_SIZE; row++) {
        for (let col = 0; col < BOARD_SIZE; col++) {
            // Determina il colore della casella
            const isLightSquare = (row + col) % 2 === 0;
            ctx.fillStyle = isLightSquare ? '#EBECD0' : '#779556';
            
            // Evidenzia la casella selezionata
            if (gameState.selectedPiece && gameState.selectedPiece.row === row && gameState.selectedPiece.col === col) {
                ctx.fillStyle = '#ffff99'; // Giallo chiaro
            }
            
            // Evidenzia le mosse possibili
            if (isPossibleMove(row, col)) {
                ctx.fillStyle = isLightSquare ? '#aaffaa' : '#88cc88'; // Verde chiaro
            }
            
            // Disegna la casella
            ctx.fillRect(col * SQUARE_SIZE, row * SQUARE_SIZE, SQUARE_SIZE, SQUARE_SIZE);
            
            // Disegna la pedina, se presente
            const piece = gameState.board[row][col];
            if (piece) {
                drawPiece(piece, row, col);
            }
        }
    }
    
    // Disegna informazioni sul turno corrente
    ctx.fillStyle = '#000000';
    ctx.font = '16px Arial';
    ctx.fillText(`Turno: ${gameState.currentPlayer === WHITE ? 'Bianco' : 'Nero'}`, 10, canvas.height + 20);
    
    // Se il gioco è finito, mostra il vincitore
    if (gameState.gameOver) {
        ctx.fillStyle = 'rgba(0, 0, 0, 0.7)';
        ctx.fillRect(0, 0, canvas.width, canvas.height);
        
        ctx.fillStyle = '#FFFFFF';
        ctx.font = '24px Arial';
        ctx.textAlign = 'center';
        ctx.fillText(`Il gioco è finito!`, canvas.width / 2, canvas.height / 2 - 20);
        ctx.fillText(`Vincitore: ${gameState.winner === WHITE ? 'Bianco' : 'Nero'}`, canvas.width / 2, canvas.height / 2 + 20);
    }
}

// Disegno di una pedina
function drawPiece(piece, row, col) {
    const x = col * SQUARE_SIZE + SQUARE_SIZE / 2;
    const y = row * SQUARE_SIZE + SQUARE_SIZE / 2;
    const radius = SQUARE_SIZE * 0.4;
    
    // Disegna il cerchio della pedina
    ctx.beginPath();
    ctx.arc(x, y, radius, 0, Math.PI * 2);
    ctx.fillStyle = piece.color === WHITE ? '#FFFFFF' : '#000000';
    ctx.fill();
    
    // Disegna il bordo
    ctx.strokeStyle = '#333333';
    ctx.lineWidth = 2;
    ctx.stroke();
    
    // Se è una dama, disegna una corona
    if (piece.isKing) {
        ctx.fillStyle = piece.color === WHITE ? '#000000' : '#FFFFFF';
        ctx.font = `${radius * 0.8}px Arial`;
        ctx.textAlign = 'center';
        ctx.textBaseline = 'middle';
        ctx.fillText('♔', x, y);
    }
}
```

## Conclusione

L'algoritmo del gioco della dama è un esempio eccellente di come la logica di gioco possa essere implementata in JavaScript. Comprende diversi aspetti fondamentali della programmazione:

1. **Gestione dello stato**: mantenimento dello stato del gioco e aggiornamento in base alle azioni dell'utente
2. **Validazione delle regole**: implementazione delle regole del gioco per garantire che solo le mosse valide siano consentite
3. **Logica condizionale complessa**: gestione di scenari come catture multiple e promozioni
4. **Intelligenza artificiale**: possibilità di implementare un avversario computerizzato

Implementare l'algoritmo del gioco della dama è un ottimo esercizio per migliorare le proprie competenze di programmazione e comprendere meglio i concetti di logica, strutture dati e interattività in JavaScript.

La combinazione di questo algoritmo con le tecniche di disegno su Canvas, la gestione degli eventi e le strutture dati appropriate permette di creare un gioco della dama completo e funzionale, offrendo un'esperienza di gioco coinvolgente e interattiva.