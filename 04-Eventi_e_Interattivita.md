# Eventi e Interattività in JavaScript: Guida Approfondita

## Introduzione

L'interattività è un elemento fondamentale delle applicazioni web moderne, specialmente nei giochi come la dama. In JavaScript, l'interattività è gestita principalmente attraverso il sistema di eventi, che permette al codice di rispondere alle azioni dell'utente e ad altri cambiamenti nell'ambiente. Questa guida esplorerà in dettaglio il sistema di eventi in JavaScript e come implementare l'interattività in un gioco della dama.

## Il Modello di Eventi in JavaScript

Il modello di eventi in JavaScript si basa sul concetto di "event-driven programming" (programmazione guidata dagli eventi), dove l'esecuzione del codice è determinata dal verificarsi di eventi specifici piuttosto che da un flusso sequenziale predefinito.

### Tipi di Eventi

JavaScript supporta numerosi tipi di eventi, tra cui:

- **Eventi del mouse**: click, dblclick, mousedown, mouseup, mousemove, mouseover, mouseout
- **Eventi della tastiera**: keydown, keypress, keyup
- **Eventi del form**: submit, change, focus, blur
- **Eventi del documento**: DOMContentLoaded, load, unload, resize, scroll
- **Eventi touch**: touchstart, touchmove, touchend, touchcancel
- **Eventi personalizzati**: eventi creati dallo sviluppatore

### Registrazione degli Event Listener

Per rispondere agli eventi, è necessario registrare degli "event listener" (ascoltatori di eventi). Ci sono diversi modi per farlo:

#### 1. Metodo addEventListener()

Questo è il metodo moderno e più flessibile per gestire gli eventi:

```javascript
element.addEventListener(eventType, handlerFunction, options);
```

Esempio:

```javascript
const button = document.getElementById('myButton');

button.addEventListener('click', function(event) {
    console.log('Il pulsante è stato cliccato!');
    console.log('Evento:', event);
});
```

#### 2. Proprietà on-event

Un modo più vecchio ma ancora utilizzato:

```javascript
element.onevent = handlerFunction;
```

Esempio:

```javascript
const button = document.getElementById('myButton');

button.onclick = function(event) {
    console.log('Il pulsante è stato cliccato!');
};
```

#### 3. Attributi HTML (sconsigliato per applicazioni moderne)

```html
<button onclick="handleClick()">Clicca qui</button>
```

```javascript
function handleClick() {
    console.log('Il pulsante è stato cliccato!');
}
```

### L'Oggetto Event

Quando si verifica un evento, JavaScript crea un oggetto `Event` che contiene informazioni sull'evento stesso. Questo oggetto viene passato alla funzione di gestione dell'evento.

```javascript
document.addEventListener('click', function(event) {
    console.log('Tipo di evento:', event.type);
    console.log('Target dell\'evento:', event.target);
    console.log('Coordinate del mouse:', event.clientX, event.clientY);
});
```

Proprietà comuni dell'oggetto Event:

- `type`: il tipo di evento (es. "click", "keydown")
- `target`: l'elemento che ha generato l'evento
- `currentTarget`: l'elemento a cui è attaccato l'event listener
- `clientX/clientY`: le coordinate del mouse relative alla finestra
- `pageX/pageY`: le coordinate del mouse relative al documento
- `key/keyCode`: informazioni sul tasto premuto (per eventi della tastiera)
- `preventDefault()`: metodo per prevenire l'azione predefinita dell'evento
- `stopPropagation()`: metodo per fermare la propagazione dell'evento

### Propagazione degli Eventi

In JavaScript, gli eventi si propagano attraverso il DOM in tre fasi:

1. **Fase di cattura (Capturing Phase)**: l'evento parte dal nodo radice e scende verso l'elemento target
2. **Fase target (Target Phase)**: l'evento raggiunge l'elemento target
3. **Fase di bubbling (Bubbling Phase)**: l'evento risale dall'elemento target verso il nodo radice

```javascript
// Registrazione di un event listener nella fase di bubbling (default)
document.getElementById('child').addEventListener('click', function() {
    console.log('Evento sul figlio');
});

document.getElementById('parent').addEventListener('click', function() {
    console.log('Evento sul genitore');
});

// Registrazione di un event listener nella fase di cattura
document.getElementById('parent').addEventListener('click', function() {
    console.log('Evento sul genitore (fase di cattura)');
}, true); // Il terzo parametro 'true' attiva la fase di cattura
```

### Event Delegation

L'event delegation è una tecnica che sfrutta la propagazione degli eventi per gestire eventi su più elementi con un singolo event listener. È particolarmente utile quando si hanno molti elementi simili o quando gli elementi vengono aggiunti dinamicamente.

```javascript
// Invece di aggiungere un event listener a ogni cella della scacchiera
document.getElementById('chessboard').addEventListener('click', function(event) {
    // Verifica se l'elemento cliccato è una cella
    if (event.target.classList.contains('cell')) {
        const row = event.target.dataset.row;
        const col = event.target.dataset.col;
        console.log(`Cella cliccata: [${row}, ${col}]`);
        // Gestione del click sulla cella
    }
});
```

## Implementazione dell'Interattività nel Gioco della Dama

Vediamo come applicare questi concetti per implementare l'interattività in un gioco della dama.

### Gestione dei Click sulla Scacchiera

```javascript
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');

// Stato del gioco
const gameState = {
    board: [], // Array bidimensionale che rappresenta la scacchiera
    selectedPiece: null, // Pedina selezionata
    currentPlayer: 'white', // Giocatore corrente
    possibleMoves: [] // Mosse possibili per la pedina selezionata
};

// Inizializzazione della scacchiera e delle pedine
initGame();

// Gestione del click sul canvas
canvas.addEventListener('click', handleCanvasClick);

function handleCanvasClick(event) {
    // Calcola la posizione del click rispetto al canvas
    const rect = canvas.getBoundingClientRect();
    const x = event.clientX - rect.left;
    const y = event.clientY - rect.top;
    
    // Converti le coordinate in indici di riga e colonna
    const col = Math.floor(x / SQUARE_SIZE);
    const row = Math.floor(y / SQUARE_SIZE);
    
    // Verifica se le coordinate sono valide
    if (row >= 0 && row < BOARD_SIZE && col >= 0 && col < BOARD_SIZE) {
        handleSquareClick(row, col);
    }
}

function handleSquareClick(row, col) {
    const piece = gameState.board[row][col];
    
    // Se non c'è una pedina selezionata e la casella contiene una pedina del giocatore corrente
    if (!gameState.selectedPiece && piece && piece.color === gameState.currentPlayer) {
        // Seleziona la pedina
        gameState.selectedPiece = { row, col };
        // Calcola le mosse possibili
        gameState.possibleMoves = calculatePossibleMoves(row, col);
        // Ridisegna la scacchiera con la pedina selezionata evidenziata
        drawBoard();
    }
    // Se c'è una pedina selezionata e la casella è una mossa valida
    else if (gameState.selectedPiece && isPossibleMove(row, col)) {
        // Esegui la mossa
        movePiece(gameState.selectedPiece.row, gameState.selectedPiece.col, row, col);
        // Deseleziona la pedina
        gameState.selectedPiece = null;
        gameState.possibleMoves = [];
        // Cambia il turno
        gameState.currentPlayer = gameState.currentPlayer === 'white' ? 'black' : 'white';
        // Ridisegna la scacchiera
        drawBoard();
    }
    // Se c'è una pedina selezionata ma la casella non è una mossa valida
    else if (gameState.selectedPiece) {
        // Deseleziona la pedina
        gameState.selectedPiece = null;
        gameState.possibleMoves = [];
        // Ridisegna la scacchiera
        drawBoard();
    }
}

function isPossibleMove(row, col) {
    return gameState.possibleMoves.some(move => move.row === row && move.col === col);
}

function calculatePossibleMoves(row, col) {
    const piece = gameState.board[row][col];
    const moves = [];
    
    // Logica per calcolare le mosse possibili
    // (dipende dalle regole specifiche del gioco della dama)
    
    return moves;
}

function movePiece(fromRow, fromCol, toRow, toCol) {
    // Sposta la pedina
    gameState.board[toRow][toCol] = gameState.board[fromRow][fromCol];
    gameState.board[fromRow][fromCol] = null;
    
    // Verifica se è una mossa di cattura
    if (Math.abs(toRow - fromRow) === 2) {
        // Calcola la posizione della pedina catturata
        const capturedRow = (fromRow + toRow) / 2;
        const capturedCol = (fromCol + toCol) / 2;
        // Rimuovi la pedina catturata
        gameState.board[capturedRow][capturedCol] = null;
    }
    
    // Verifica se la pedina deve essere promossa a dama
    if ((piece.color === 'white' && toRow === 0) || (piece.color === 'black' && toRow === BOARD_SIZE - 1)) {
        gameState.board[toRow][toCol].isKing = true;
    }
}
```

### Evidenziazione delle Caselle e Animazioni

Per migliorare l'esperienza utente, è possibile aggiungere effetti visivi come l'evidenziazione delle caselle selezionate e le mosse possibili:

```javascript
function drawBoard() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    
    // Disegna la scacchiera
    for (let row = 0; row < BOARD_SIZE; row++) {
        for (let col = 0; col < BOARD_SIZE; col++) {
            // Determina il colore della casella
            let squareColor = (row + col) % 2 === 0 ? '#EBECD0' : '#779556';
            
            // Evidenzia la casella selezionata
            if (gameState.selectedPiece && gameState.selectedPiece.row === row && gameState.selectedPiece.col === col) {
                squareColor = '#ffff99'; // Giallo chiaro per la casella selezionata
            }
            // Evidenzia le mosse possibili
            else if (isPossibleMove(row, col)) {
                squareColor = '#aaffaa'; // Verde chiaro per le mosse possibili
            }
            
            // Disegna la casella
            ctx.fillStyle = squareColor;
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
    ctx.fillText(`Turno: ${gameState.currentPlayer === 'white' ? 'Bianco' : 'Nero'}`, 10, canvas.height + 20);
}
```

### Animazione delle Mosse

Per rendere il gioco più dinamico, è possibile aggiungere animazioni quando le pedine si muovono:

```javascript
function animateMove(fromRow, fromCol, toRow, toCol, callback) {
    const piece = gameState.board[fromRow][fromCol];
    const startX = fromCol * SQUARE_SIZE + SQUARE_SIZE / 2;
    const startY = fromRow * SQUARE_SIZE + SQUARE_SIZE / 2;
    const endX = toCol * SQUARE_SIZE + SQUARE_SIZE / 2;
    const endY = toRow * SQUARE_SIZE + SQUARE_SIZE / 2;
    
    const duration = 500; // Durata dell'animazione in millisecondi
    const startTime = performance.now();
    
    function animate(currentTime) {
        const elapsed = currentTime - startTime;
        const progress = Math.min(elapsed / duration, 1);
        
        // Calcola la posizione corrente
        const currentX = startX + (endX - startX) * progress;
        const currentY = startY + (endY - startY) * progress;
        
        // Ridisegna la scacchiera
        drawBoard();
        
        // Disegna la pedina in movimento
        drawPieceAtPosition(piece, currentX, currentY);
        
        if (progress < 1) {
            // Continua l'animazione
            requestAnimationFrame(animate);
        } else {
            // Animazione completata, esegui il callback
            if (callback) callback();
        }
    }
    
    requestAnimationFrame(animate);
}

function drawPieceAtPosition(piece, x, y) {
    const radius = SQUARE_SIZE * 0.4;
    
    ctx.beginPath();
    ctx.arc(x, y, radius, 0, Math.PI * 2);
    ctx.fillStyle = piece.color === 'white' ? '#FFFFFF' : '#000000';
    ctx.fill();
    ctx.strokeStyle = '#333333';
    ctx.lineWidth = 2;
    ctx.stroke();
    
    // Se è una dama, disegna una corona
    if (piece.isKing) {
        drawCrown(x, y, radius * 0.6);
    }
}
```

## Gestione degli Eventi della Tastiera

Oltre agli eventi del mouse, è possibile implementare controlli da tastiera per migliorare l'accessibilità o offrire modalità alternative di interazione:

```javascript
document.addEventListener('keydown', function(event) {
    // Annulla la selezione con il tasto Escape
    if (event.key === 'Escape' && gameState.selectedPiece) {
        gameState.selectedPiece = null;
        gameState.possibleMoves = [];
        drawBoard();
    }
    
    // Implementa la navigazione con le frecce
    if (gameState.selectedPiece && ['ArrowUp', 'ArrowDown', 'ArrowLeft', 'ArrowRight'].includes(event.key)) {
        let { row, col } = gameState.selectedPiece;
        
        switch (event.key) {
            case 'ArrowUp': row = Math.max(0, row - 1); break;
            case 'ArrowDown': row = Math.min(BOARD_SIZE - 1, row + 1); break;
            case 'ArrowLeft': col = Math.max(0, col - 1); break;
            case 'ArrowRight': col = Math.min(BOARD_SIZE - 1, col + 1); break;
        }
        
        // Se la nuova posizione è diversa, simula un click su quella casella
        if (row !== gameState.selectedPiece.row || col !== gameState.selectedPiece.col) {
            handleSquareClick(row, col);
        }
    }
});
```

## Eventi Touch per Dispositivi Mobili

Per rendere il gioco utilizzabile su dispositivi mobili, è necessario gestire anche gli eventi touch:

```javascript
canvas.addEventListener('touchstart', function(event) {
    // Previeni lo zoom e altri comportamenti predefiniti del browser
    event.preventDefault();
    
    // Ottieni le coordinate del touch
    const touch = event.touches[0];
    const rect = canvas.getBoundingClientRect();
    const x = touch.clientX - rect.left;
    const y = touch.clientY - rect.top;
    
    // Converti le coordinate in indici di riga e colonna
    const col = Math.floor(x / SQUARE_SIZE);
    const row = Math.floor(y / SQUARE_SIZE);
    
    // Gestisci il touch come un click
    if (row >= 0 && row < BOARD_SIZE && col >= 0 && col < BOARD_SIZE) {
        handleSquareClick(row, col);
    }
});
```

## Eventi Personalizzati

Gli eventi personalizzati possono essere utili per gestire la logica di gioco in modo più modulare e disaccoppiato:

```javascript
// Creazione di un evento personalizzato
function createGameEvent(eventName, detail) {
    return new CustomEvent(eventName, { detail });
}

// Esempio: evento per la fine del gioco
function endGame(winner) {
    const gameOverEvent = createGameEvent('gameOver', { winner });
    document.dispatchEvent(gameOverEvent);
}

// Registrazione di un listener per l'evento personalizzato
document.addEventListener('gameOver', function(event) {
    const winner = event.detail.winner;
    alert(`Il gioco è finito! Ha vinto il giocatore ${winner === 'white' ? 'Bianco' : 'Nero'}`);
    // Resetta il gioco o mostra un menu
});
```

## Gestione dello Stato del Gioco

Un aspetto importante dell'interattività è la gestione dello stato del gioco. È possibile utilizzare un pattern di tipo "state machine" (macchina a stati) per gestire le diverse fasi del gioco:

```javascript
const GameStates = {
    WAITING_FOR_SELECTION: 'waitingForSelection',
    PIECE_SELECTED: 'pieceSelected',
    MOVING: 'moving',
    GAME_OVER: 'gameOver'
};

const gameState = {
    board: [],
    currentPlayer: 'white',
    selectedPiece: null,
    possibleMoves: [],
    currentState: GameStates.WAITING_FOR_SELECTION
};

function handleSquareClick(row, col) {
    switch (gameState.currentState) {
        case GameStates.WAITING_FOR_SELECTION:
            // Logica per la selezione di una pedina
            const piece = gameState.board[row][col];
            if (piece && piece.color === gameState.currentPlayer) {
                selectPiece(row, col);
                gameState.currentState = GameStates.PIECE_SELECTED;
            }
            break;
            
        case GameStates.PIECE_SELECTED:
            // Logica per la selezione della destinazione o cambio di selezione
            if (isPossibleMove(row, col)) {
                gameState.currentState = GameStates.MOVING;
                movePiece(gameState.selectedPiece.row, gameState.selectedPiece.col, row, col, function() {
                    // Callback dopo il completamento della mossa
                    if (isGameOver()) {
                        gameState.currentState = GameStates.GAME_OVER;
                        endGame(determineWinner());
                    } else {
                        switchPlayer();
                        gameState.currentState = GameStates.WAITING_FOR_SELECTION;
                    }
                });
            } else {
                // Se si clicca su un'altra pedina del giocatore corrente, cambia selezione
                const piece = gameState.board[row][col];
                if (piece && piece.color === gameState.currentPlayer) {
                    selectPiece(row, col);
                } else {
                    // Altrimenti, deseleziona
                    deselectPiece();
                    gameState.currentState = GameStates.WAITING_FOR_SELECTION;
                }
            }
            break;
            
        case GameStates.MOVING:
            // Ignora i click durante l'animazione di movimento
            break;
            
        case GameStates.GAME_OVER:
            // Ignora i click quando il gioco è finito
            break;
    }
    
    // Aggiorna la visualizzazione
    drawBoard();
}
```

## Conclusione

Gli eventi e l'interattività sono elementi fondamentali per creare un'esperienza di gioco coinvolgente. In JavaScript, il sistema di eventi offre strumenti potenti per gestire le interazioni dell'utente e creare applicazioni dinamiche.

Nel contesto del gioco della dama, abbiamo visto come implementare:

1. Gestione dei click sulla scacchiera
2. Selezione e movimento delle pedine
3. Evidenziazione delle caselle e animazioni
4. Controlli da tastiera e touch
5. Eventi personalizzati
6. Gestione dello stato del gioco

Questi concetti possono essere estesi e adattati per creare giochi più complessi o altre applicazioni interattive. La chiave è progettare un sistema di eventi che sia chiaro, modulare e che risponda efficacemente alle azioni dell'utente.