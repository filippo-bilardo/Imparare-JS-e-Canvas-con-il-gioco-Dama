# Guida Didattica: Dama in JavaScript - Step 7: Gioco Completo

## Obiettivo
In questo settimo e ultimo step, completeremo il gioco della dama implementando il controllo della fine del gioco e la gestione della vittoria. Il gioco terminerà quando un giocatore perde tutte le sue pedine o non può più muoversi.

## Concetti teorici da approfondire

### 1. Condizioni di fine gioco
- **Rilevamento della vittoria**: Come determinare quando un giocatore ha vinto
- **Stallo**: Come rilevare situazioni di stallo in cui un giocatore non può muoversi

### 2. Interfaccia utente per la fine del gioco
- **Messaggi di fine gioco**: Come comunicare all'utente l'esito della partita
- **Opzioni di fine gioco**: Come offrire all'utente la possibilità di iniziare una nuova partita

### 3. Refactoring e ottimizzazione
- **Organizzazione del codice**: Come strutturare il codice per un progetto completo
- **Ottimizzazione delle prestazioni**: Come migliorare l'efficienza del codice

## Analisi del codice

### JavaScript: Controllo della fine del gioco

```javascript
// Controlla se il gioco è finito
function checkGameOver() {
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
    
    // Se un giocatore non ha più pedine, l'altro ha vinto
    if (whitePieces === 0) {
        return BLACK; // Il nero ha vinto
    } else if (blackPieces === 0) {
        return WHITE; // Il bianco ha vinto
    }
    
    // Controlla se il giocatore corrente può muoversi
    const canMove = checkPlayerCanMove(gameState.currentPlayer);
    if (!canMove) {
        // Se il giocatore corrente non può muoversi, l'altro ha vinto
        return gameState.currentPlayer === WHITE ? BLACK : WHITE;
    }
    
    // Il gioco non è ancora finito
    return null;
}

// Controlla se un giocatore può muoversi
function checkPlayerCanMove(playerColor) {
    for (let row = 0; row < BOARD_SIZE; row++) {
        for (let col = 0; col < BOARD_SIZE; col++) {
            const piece = gameState.board[row][col];
            if (piece && piece.color === playerColor) {
                const moves = getValidMoves(row, col);
                if (moves.length > 0) {
                    return true; // Il giocatore può muoversi
                }
            }
        }
    }
    return false; // Il giocatore non può muoversi
}
```

**Spiegazione:**
- Implementiamo la funzione `checkGameOver` che controlla se il gioco è finito
- Contiamo le pedine di ogni colore e verifichiamo se un giocatore ha perso tutte le sue pedine
- Implementiamo la funzione `checkPlayerCanMove` che verifica se un giocatore può muoversi
- Se un giocatore non può muoversi, l'altro ha vinto

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
    
    // Controlla se il gioco è finito
    const winner = checkGameOver();
    if (winner) {
        // Mostra un messaggio di vittoria
        setTimeout(() => {
            alert(`Il giocatore ${winner === WHITE ? 'Bianco' : 'Nero'} ha vinto!`);
            newGame();
        }, 100);
    }
    
    // Ridisegna la scacchiera
    drawBoard();
}
```

**Spiegazione:**
- Aggiorniamo la funzione `movePiece` per controllare se il gioco è finito dopo ogni mossa
- Se il gioco è finito, mostriamo un messaggio di vittoria e iniziamo una nuova partita
- Utilizziamo `setTimeout` per assicurarci che la scacchiera venga ridisegnata prima di mostrare il messaggio

### JavaScript: Miglioramento della funzione di nuova partita

```javascript
// Inizia una nuova partita
function newGame() {
    // Reimposta lo stato del gioco
    initBoard();
    placePieces();
    gameState.currentPlayer = WHITE;
    gameState.selectedPiece = null;
    gameState.validMoves = [];
    gameState.moveHistory = [];
    gameState.capturedPieces = [];
    
    // Aggiorna le informazioni di gioco
    updateGameInfo();
    
    // Ridisegna la scacchiera
    drawBoard();
}

// Aggiungi event listener per il pulsante di nuova partita
newGameBtn.addEventListener('click', newGame);
```

**Spiegazione:**
- Miglioriamo la funzione `newGame` per reimpostare completamente lo stato del gioco
- Reimpostiamo tutte le proprietà dello stato del gioco, inclusi lo storico delle mosse e le pedine catturate
- Aggiungiamo un event listener per il pulsante di nuova partita

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
- Contiamo le pedine di ogni colore e aggiorniamo gli elementi HTML corrispondenti

## Esercizi pratici

1. **Statistiche di gioco**: Implementa un sistema di statistiche che tenga traccia del numero di partite vinte da ciascun giocatore.

2. **Timer di gioco**: Aggiungi un timer che limita il tempo disponibile per ogni mossa.

3. **Livelli di difficoltà**: Implementa diversi livelli di difficoltà per il gioco, ad esempio modificando il numero di pedine iniziali o le regole di movimento.

4. **Modalità di gioco alternative**: Implementa varianti del gioco della dama, come la dama italiana o la dama inglese, con regole leggermente diverse.

5. **Salvataggio e caricamento**: Aggiungi la possibilità di salvare e caricare partite utilizzando il localStorage del browser.

## Approfondimenti

- **Intelligenza artificiale**: Studia come implementare un'IA per giocare contro il computer, utilizzando algoritmi come Minimax o Monte Carlo Tree Search
- **Ottimizzazione delle prestazioni**: Approfondisci tecniche di ottimizzazione come la memorizzazione (memoization) per migliorare l'efficienza del calcolo delle mosse valide
- **Accessibilità**: Studia come rendere il gioco accessibile a persone con disabilità, ad esempio implementando comandi da tastiera o supporto per screen reader

## Conclusione

Congratulazioni! Hai completato l'implementazione del gioco della dama in JavaScript utilizzando l'elemento Canvas di HTML. In questo percorso, hai imparato a:

1. Creare una scacchiera interattiva con Canvas
2. Implementare le regole del gioco della dama
3. Gestire l'interazione dell'utente tramite eventi del mouse
4. Implementare algoritmi per il calcolo delle mosse valide e delle catture
5. Gestire lo stato del gioco e i turni dei giocatori
6. Implementare funzionalità avanzate come la promozione a dama e l'annullamento delle mosse
7. Controllare la fine del gioco e gestire la vittoria

Questo progetto ha coperto molti aspetti importanti della programmazione JavaScript, tra cui:

- Manipolazione del DOM
- Gestione degli eventi
- Disegno con Canvas
- Strutture dati complesse
- Algoritmi di gioco
- Gestione dello stato dell'applicazione

Puoi continuare a migliorare questo gioco implementando le funzionalità suggerite negli esercizi pratici o esplorando gli approfondimenti proposti. Buon divertimento con la programmazione JavaScript!