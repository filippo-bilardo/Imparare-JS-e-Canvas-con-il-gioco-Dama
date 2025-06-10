# Guida Didattica: Dama in JavaScript - Step 9: Gestione Errori e Debugging

## Obiettivo
In questo nono step, implementeremo tecniche di gestione degli errori e debugging nel gioco della dama. Queste tecniche sono fondamentali per creare un'applicazione robusta che gestisca correttamente situazioni impreviste e fornisca informazioni utili per la risoluzione dei problemi.

## Concetti teorici da approfondire

### 1. Gestione degli errori in JavaScript
- **Try-catch-finally**: Come utilizzare i blocchi try-catch per gestire le eccezioni
- **Tipi di errori**: Comprendere i diversi tipi di errori in JavaScript (SyntaxError, TypeError, ReferenceError, ecc.)
- **Errori personalizzati**: Come creare e lanciare errori personalizzati

### 2. Tecniche di debugging
- **Console API**: Utilizzo avanzato di console.log, console.error, console.warn, console.table, ecc.
- **Breakpoint e step-through**: Come utilizzare i breakpoint nel browser per eseguire il codice passo-passo
- **Ispezione delle variabili**: Come esaminare lo stato delle variabili durante l'esecuzione

### 3. Logging e monitoraggio
- **Sistemi di logging**: Come implementare un sistema di logging per tracciare eventi e errori
- **Livelli di log**: Come utilizzare diversi livelli di log (debug, info, warning, error)
- **Persistenza dei log**: Come salvare i log per analisi future

## Analisi del codice

### JavaScript: Implementazione di un sistema di logging

```javascript
// Sistema di logging
const Logger = {
    // Livelli di log
    LEVELS: {
        DEBUG: 0,
        INFO: 1,
        WARNING: 2,
        ERROR: 3
    },
    
    // Livello corrente (può essere modificato in base alle esigenze)
    currentLevel: 0, // DEBUG
    
    // Array per memorizzare i log
    logs: [],
    
    // Numero massimo di log da memorizzare
    maxLogs: 100,
    
    // Funzione per aggiungere un log
    log: function(level, message, data = null) {
        // Verifica se il livello è sufficiente per essere loggato
        if (level >= this.currentLevel) {
            const logEntry = {
                timestamp: new Date(),
                level: Object.keys(this.LEVELS).find(key => this.LEVELS[key] === level),
                message: message,
                data: data
            };
            
            // Aggiungi il log all'array
            this.logs.push(logEntry);
            
            // Limita il numero di log memorizzati
            if (this.logs.length > this.maxLogs) {
                this.logs.shift(); // Rimuovi il log più vecchio
            }
            
            // Stampa il log nella console con il formato appropriato
            switch (level) {
                case this.LEVELS.DEBUG:
                    console.debug(`[DEBUG] ${message}`, data);
                    break;
                case this.LEVELS.INFO:
                    console.info(`[INFO] ${message}`, data);
                    break;
                case this.LEVELS.WARNING:
                    console.warn(`[WARNING] ${message}`, data);
                    break;
                case this.LEVELS.ERROR:
                    console.error(`[ERROR] ${message}`, data);
                    break;
            }
            
            // Salva i log nel localStorage
            this.saveLogs();
        }
    },
    
    // Funzioni di convenienza per i diversi livelli di log
    debug: function(message, data = null) {
        this.log(this.LEVELS.DEBUG, message, data);
    },
    
    info: function(message, data = null) {
        this.log(this.LEVELS.INFO, message, data);
    },
    
    warning: function(message, data = null) {
        this.log(this.LEVELS.WARNING, message, data);
    },
    
    error: function(message, data = null) {
        this.log(this.LEVELS.ERROR, message, data);
    },
    
    // Funzione per salvare i log nel localStorage
    saveLogs: function() {
        try {
            localStorage.setItem('dama_logs', JSON.stringify(this.logs));
        } catch (error) {
            console.error('Errore durante il salvataggio dei log:', error);
        }
    },
    
    // Funzione per caricare i log dal localStorage
    loadLogs: function() {
        try {
            const savedLogs = localStorage.getItem('dama_logs');
            if (savedLogs) {
                this.logs = JSON.parse(savedLogs);
            }
        } catch (error) {
            console.error('Errore durante il caricamento dei log:', error);
        }
    },
    
    // Funzione per esportare i log
    exportLogs: function() {
        const logsJson = JSON.stringify(this.logs, null, 2);
        const blob = new Blob([logsJson], { type: 'application/json' });
        const url = URL.createObjectURL(blob);
        
        const a = document.createElement('a');
        a.href = url;
        a.download = `dama_logs_${new Date().toISOString().replace(/:/g, '-')}.json`;
        a.click();
        
        URL.revokeObjectURL(url);
    },
    
    // Funzione per cancellare i log
    clearLogs: function() {
        this.logs = [];
        localStorage.removeItem('dama_logs');
    }
};

// Inizializza i log all'avvio
Logger.loadLogs();
```

**Spiegazione:**
- Implementiamo un oggetto `Logger` che fornisce funzionalità di logging avanzate
- Definiamo diversi livelli di log (DEBUG, INFO, WARNING, ERROR)
- Implementiamo funzioni per aggiungere log, salvarli nel localStorage, caricarli, esportarli e cancellarli
- Questo sistema di logging ci permetterà di tracciare eventi e errori nel gioco

### JavaScript: Gestione degli errori nelle funzioni principali

```javascript
// Gestione degli errori nella funzione movePiece
function movePiece(fromRow, fromCol, toRow, toCol, captures = []) {
    try {
        // Validazione degli input
        if (!isValidPosition(fromRow, fromCol) || !isValidPosition(toRow, toCol)) {
            throw new Error('Posizione non valida');
        }
        
        const piece = gameState.board[fromRow][fromCol];
        if (!piece) {
            throw new Error('Nessuna pedina nella posizione di partenza');
        }
        
        // Salva lo stato corrente per la funzionalità di annullamento
        const currentState = {
            board: JSON.parse(JSON.stringify(gameState.board)),
            currentPlayer: gameState.currentPlayer,
            captures: [...captures]
        };
        gameState.moveHistory.push(currentState);
        
        // Sposta la pedina
        gameState.board[toRow][toCol] = piece;
        gameState.board[fromRow][fromCol] = null;
        
        // Rimuovi le pedine catturate
        for (const capture of captures) {
            if (!isValidPosition(capture.row, capture.col)) {
                Logger.warning('Posizione di cattura non valida', capture);
                continue;
            }
            gameState.board[capture.row][capture.col] = null;
        }
        
        // Controlla se la pedina deve essere promossa a dama
        if (!piece.isKing) {
            if ((piece.color === WHITE && toRow === 0) || (piece.color === BLACK && toRow === BOARD_SIZE - 1)) {
                piece.isKing = true;
                Logger.info('Pedina promossa a dama', { row: toRow, col: toCol, color: piece.color });
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
        
        // Log della mossa
        Logger.info('Mossa eseguita', {
            from: { row: fromRow, col: fromCol },
            to: { row: toRow, col: toCol },
            captures: captures,
            player: piece.color
        });
        
        // Salva automaticamente dopo ogni mossa
        autoSaveGame();
    } catch (error) {
        Logger.error('Errore durante lo spostamento della pedina', {
            error: error.message,
            from: { row: fromRow, col: fromCol },
            to: { row: toRow, col: toCol }
        });
        
        // Ripristina lo stato precedente se disponibile
        if (gameState.moveHistory.length > 0) {
            const previousState = gameState.moveHistory.pop();
            gameState.board = previousState.board;
            gameState.currentPlayer = previousState.currentPlayer;
            updateGameInfo();
            drawBoard();
        }
        
        // Mostra un messaggio di errore all'utente
        alert(`Errore: ${error.message}`);
    }
}

// Funzione per verificare se una posizione è valida
function isValidPosition(row, col) {
    return row >= 0 && row < BOARD_SIZE && col >= 0 && col < BOARD_SIZE;
}
```

**Spiegazione:**
- Aggiorniamo la funzione `movePiece` per includere la gestione degli errori con try-catch
- Implementiamo la validazione degli input per prevenire errori
- Utilizziamo il sistema di logging per tracciare eventi e errori
- In caso di errore, ripristiniamo lo stato precedente e mostriamo un messaggio all'utente

### JavaScript: Implementazione di un pannello di debug

```javascript
// Pannello di debug
function createDebugPanel() {
    // Crea il pannello di debug
    const debugPanel = document.createElement('div');
    debugPanel.id = 'debugPanel';
    debugPanel.style.position = 'fixed';
    debugPanel.style.top = '10px';
    debugPanel.style.right = '10px';
    debugPanel.style.width = '300px';
    debugPanel.style.maxHeight = '400px';
    debugPanel.style.overflowY = 'auto';
    debugPanel.style.backgroundColor = '#f8f8f8';
    debugPanel.style.border = '1px solid #ccc';
    debugPanel.style.borderRadius = '5px';
    debugPanel.style.padding = '10px';
    debugPanel.style.boxShadow = '0 0 10px rgba(0, 0, 0, 0.1)';
    debugPanel.style.zIndex = '1000';
    debugPanel.style.display = 'none'; // Nascosto di default
    
    // Crea l'intestazione del pannello
    const header = document.createElement('div');
    header.style.display = 'flex';
    header.style.justifyContent = 'space-between';
    header.style.alignItems = 'center';
    header.style.marginBottom = '10px';
    
    const title = document.createElement('h3');
    title.textContent = 'Pannello di Debug';
    title.style.margin = '0';
    
    const closeButton = document.createElement('button');
    closeButton.textContent = 'X';
    closeButton.style.border = 'none';
    closeButton.style.background = 'none';
    closeButton.style.cursor = 'pointer';
    closeButton.style.fontSize = '16px';
    closeButton.style.fontWeight = 'bold';
    closeButton.onclick = function() {
        debugPanel.style.display = 'none';
    };
    
    header.appendChild(title);
    header.appendChild(closeButton);
    debugPanel.appendChild(header);
    
    // Crea i controlli del pannello
    const controls = document.createElement('div');
    controls.style.marginBottom = '10px';
    
    // Pulsante per esportare i log
    const exportButton = document.createElement('button');
    exportButton.textContent = 'Esporta Log';
    exportButton.style.marginRight = '5px';
    exportButton.onclick = function() {
        Logger.exportLogs();
    };
    
    // Pulsante per cancellare i log
    const clearButton = document.createElement('button');
    clearButton.textContent = 'Cancella Log';
    clearButton.onclick = function() {
        Logger.clearLogs();
        updateDebugPanel();
    };
    
    controls.appendChild(exportButton);
    controls.appendChild(clearButton);
    debugPanel.appendChild(controls);
    
    // Crea il contenitore per i log
    const logsContainer = document.createElement('div');
    logsContainer.id = 'logsContainer';
    debugPanel.appendChild(logsContainer);
    
    // Aggiungi il pannello al body
    document.body.appendChild(debugPanel);
    
    // Aggiungi un pulsante per mostrare/nascondere il pannello
    const toggleButton = document.createElement('button');
    toggleButton.textContent = 'Debug';
    toggleButton.style.position = 'fixed';
    toggleButton.style.top = '10px';
    toggleButton.style.right = '10px';
    toggleButton.style.zIndex = '1001';
    toggleButton.onclick = function() {
        if (debugPanel.style.display === 'none') {
            debugPanel.style.display = 'block';
            updateDebugPanel();
        } else {
            debugPanel.style.display = 'none';
        }
    };
    
    document.body.appendChild(toggleButton);
    
    // Funzione per aggiornare il contenuto del pannello
    function updateDebugPanel() {
        const logsContainer = document.getElementById('logsContainer');
        logsContainer.innerHTML = '';
        
        // Mostra gli ultimi 20 log (dal più recente al più vecchio)
        const recentLogs = [...Logger.logs].reverse().slice(0, 20);
        
        recentLogs.forEach(log => {
            const logEntry = document.createElement('div');
            logEntry.style.marginBottom = '5px';
            logEntry.style.padding = '5px';
            logEntry.style.borderLeft = '3px solid #ccc';
            
            // Colore in base al livello di log
            switch (log.level) {
                case 'DEBUG':
                    logEntry.style.borderColor = '#6c757d';
                    break;
                case 'INFO':
                    logEntry.style.borderColor = '#28a745';
                    break;
                case 'WARNING':
                    logEntry.style.borderColor = '#ffc107';
                    break;
                case 'ERROR':
                    logEntry.style.borderColor = '#dc3545';
                    break;
            }
            
            const timestamp = new Date(log.timestamp).toLocaleTimeString();
            const header = document.createElement('div');
            header.style.fontWeight = 'bold';
            header.textContent = `[${timestamp}] [${log.level}]`;
            
            const message = document.createElement('div');
            message.textContent = log.message;
            
            logEntry.appendChild(header);
            logEntry.appendChild(message);
            
            // Se ci sono dati aggiuntivi, mostrali come JSON
            if (log.data) {
                const data = document.createElement('pre');
                data.style.fontSize = '12px';
                data.style.marginTop = '5px';
                data.style.backgroundColor = '#f0f0f0';
                data.style.padding = '5px';
                data.textContent = JSON.stringify(log.data, null, 2);
                logEntry.appendChild(data);
            }
            
            logsContainer.appendChild(logEntry);
        });
    }
    
    // Aggiorna il pannello ogni 2 secondi
    setInterval(updateDebugPanel, 2000);
    
    return {
        update: updateDebugPanel
    };
}

// Crea il pannello di debug all'avvio
const debugPanel = createDebugPanel();
```

**Spiegazione:**
- Implementiamo una funzione `createDebugPanel` che crea un pannello di debug interattivo
- Il pannello mostra i log più recenti, con colori diversi in base al livello di log
- Aggiungiamo pulsanti per esportare e cancellare i log
- Il pannello può essere mostrato/nascosto con un pulsante

### JavaScript: Monitoraggio delle prestazioni

```javascript
// Monitoraggio delle prestazioni
const Performance = {
    // Memorizza i tempi di inizio per le misurazioni
    timers: {},
    
    // Avvia una misurazione
    start: function(label) {
        this.timers[label] = performance.now();
    },
    
    // Termina una misurazione e registra il risultato
    end: function(label) {
        if (!this.timers[label]) {
            Logger.warning(`Timer '${label}' non trovato`);
            return;
        }
        
        const duration = performance.now() - this.timers[label];
        Logger.debug(`Performance '${label}': ${duration.toFixed(2)}ms`);
        
        delete this.timers[label];
        return duration;
    },
    
    // Misura il tempo di esecuzione di una funzione
    measure: function(label, func, ...args) {
        this.start(label);
        const result = func(...args);
        this.end(label);
        return result;
    }
};

// Esempio di utilizzo del monitoraggio delle prestazioni
function drawBoard() {
    Performance.start('drawBoard');
    
    // Codice esistente per disegnare la scacchiera
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    
    for (let row = 0; row < BOARD_SIZE; row++) {
        for (let col = 0; col < BOARD_SIZE; col++) {
            // Determina il colore della casella (alternando bianco e nero)
            const isLightSquare = (row + col) % 2 === 0;
            ctx.fillStyle = isLightSquare ? '#EBECD0' : '#779556';
            
            // Disegna la casella
            ctx.fillRect(col * SQUARE_SIZE, row * SQUARE_SIZE, SQUARE_SIZE, SQUARE_SIZE);
            
            // ... resto del codice per disegnare la scacchiera ...
        }
    }
    
    Performance.end('drawBoard');
}
```

**Spiegazione:**
- Implementiamo un oggetto `Performance` che fornisce funzionalità per misurare le prestazioni del codice
- Utilizziamo l'API `performance.now()` per misurare il tempo di esecuzione delle funzioni
- Registriamo i risultati delle misurazioni nel sistema di logging
- Questo ci permetterà di identificare colli di bottiglia e ottimizzare le parti critiche del codice

## Esercizi pratici

1. **Modalità di debug avanzata**: Implementa una modalità di debug che mostri informazioni aggiuntive sulla scacchiera, come le coordinate delle caselle e le mosse valide per ogni pedina.

2. **Gestione degli errori di rete**: Se il gioco utilizza funzionalità online (ad esempio, per il multiplayer), implementa una gestione robusta degli errori di rete.

3. **Test automatici**: Implementa una suite di test automatici per verificare il corretto funzionamento delle funzionalità principali del gioco.

4. **Profiling avanzato**: Utilizza gli strumenti di profiling del browser per identificare e risolvere problemi di prestazioni nel gioco.

## Approfondimenti

- **Error Boundaries in React**: Se stai utilizzando React, studia come implementare Error Boundaries per gestire gli errori in modo elegante
- **Monitoraggio remoto**: Esplora soluzioni come Sentry o LogRocket per il monitoraggio remoto degli errori in produzione
- **Test-Driven Development (TDD)**: Approfondisci l'approccio TDD per sviluppare software più robusto e manutenibile

## Conclusione

In questo nono step, abbiamo implementato tecniche di gestione degli errori e debugging nel gioco della dama. Abbiamo imparato a utilizzare try-catch per gestire le eccezioni, a implementare un sistema di logging avanzato, a creare un pannello di debug interattivo e a monitorare le prestazioni del codice. Queste tecniche sono fondamentali per creare un'applicazione robusta e manutenibile, in grado di gestire situazioni impreviste e fornire informazioni utili per la risoluzione dei problemi. Nel prossimo step, esploreremo tecniche per migliorare la sicurezza e proteggere i dati del gioco.