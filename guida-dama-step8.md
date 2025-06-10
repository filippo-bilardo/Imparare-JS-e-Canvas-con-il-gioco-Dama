# Guida Didattica: Dama in JavaScript - Step 8: Persistenza dei Dati

## Obiettivo
In questo ottavo step, implementeremo la persistenza dei dati nel gioco della dama. Questo permetterà di salvare lo stato del gioco e riprenderlo in un secondo momento, anche dopo aver chiuso il browser. Utilizzeremo il localStorage per memorizzare lo stato della partita e implementeremo funzionalità per salvare e caricare le partite.

## Concetti teorici da approfondire

### 1. Persistenza dei dati nel browser
- **Web Storage API**: Come utilizzare localStorage e sessionStorage per memorizzare dati nel browser
- **Serializzazione e deserializzazione**: Come convertire oggetti JavaScript in stringhe JSON e viceversa
- **Limitazioni del localStorage**: Comprendere i limiti di spazio e le restrizioni di sicurezza

### 2. Gestione dello stato dell'applicazione
- **Salvataggio dello stato**: Come memorizzare lo stato corrente del gioco
- **Ripristino dello stato**: Come ricaricare uno stato salvato precedentemente
- **Gestione di più salvataggi**: Come implementare un sistema per gestire più partite salvate

### 3. Interfaccia utente per la gestione dei salvataggi
- **UI per il salvataggio**: Come implementare controlli per salvare e caricare partite
- **Feedback all'utente**: Come comunicare all'utente lo stato delle operazioni di salvataggio e caricamento

## Analisi del codice

### Aggiornamento della struttura HTML

```html
<div class="controls">
    <button id="newGameBtn">Nuova Partita</button>
    <button id="undoBtn">Annulla Mossa</button>
    <button id="saveGameBtn">Salva Partita</button>
    <button id="loadGameBtn">Carica Partita</button>
</div>
```

**Spiegazione:**
- Aggiungiamo due nuovi pulsanti: "Salva Partita" e "Carica Partita"
- Questi pulsanti permetteranno all'utente di salvare lo stato corrente del gioco e di caricarlo in un secondo momento

### JavaScript: Funzioni per la persistenza dei dati

```javascript
// Salva lo stato del gioco nel localStorage
function saveGame() {
    try {
        // Crea un oggetto con lo stato corrente del gioco
        const saveData = {
            board: gameState.board,
            currentPlayer: gameState.currentPlayer,
            moveHistory: gameState.moveHistory,
            timestamp: new Date().toISOString()
        };
        
        // Genera un ID univoco per il salvataggio
        const saveId = 'dama_save_' + new Date().getTime();
        
        // Salva i dati nel localStorage
        localStorage.setItem(saveId, JSON.stringify(saveData));
        
        // Aggiorna la lista dei salvataggi
        updateSavesList();
        
        // Feedback all'utente
        alert('Partita salvata con successo!');
    } catch (error) {
        console.error('Errore durante il salvataggio:', error);
        alert('Errore durante il salvataggio della partita.');
    }
}

// Carica uno stato del gioco dal localStorage
function loadGame() {
    try {
        // Ottieni la lista dei salvataggi disponibili
        const saves = getSavedGames();
        
        if (saves.length === 0) {
            alert('Nessuna partita salvata disponibile.');
            return;
        }
        
        // Crea una lista di salvataggi da mostrare all'utente
        let savesList = 'Seleziona una partita da caricare:\n';
        saves.forEach((save, index) => {
            const date = new Date(save.timestamp);
            savesList += `${index + 1}. ${date.toLocaleString()}\n`;
        });
        
        // Chiedi all'utente quale salvataggio caricare
        const selection = prompt(savesList);
        const index = parseInt(selection) - 1;
        
        if (isNaN(index) || index < 0 || index >= saves.length) {
            alert('Selezione non valida.');
            return;
        }
        
        // Carica il salvataggio selezionato
        const saveData = saves[index];
        
        // Ripristina lo stato del gioco
        gameState.board = saveData.board;
        gameState.currentPlayer = saveData.currentPlayer;
        gameState.moveHistory = saveData.moveHistory;
        gameState.selectedPiece = null;
        gameState.validMoves = [];
        
        // Aggiorna le informazioni di gioco e ridisegna la scacchiera
        updateGameInfo();
        drawBoard();
        
        // Feedback all'utente
        alert('Partita caricata con successo!');
    } catch (error) {
        console.error('Errore durante il caricamento:', error);
        alert('Errore durante il caricamento della partita.');
    }
}

// Ottieni la lista dei salvataggi disponibili
function getSavedGames() {
    const saves = [];
    
    // Itera su tutti gli elementi nel localStorage
    for (let i = 0; i < localStorage.length; i++) {
        const key = localStorage.key(i);
        
        // Verifica se la chiave corrisponde a un salvataggio del gioco
        if (key.startsWith('dama_save_')) {
            try {
                const saveData = JSON.parse(localStorage.getItem(key));
                saveData.id = key; // Aggiungi l'ID del salvataggio
                saves.push(saveData);
            } catch (error) {
                console.error('Errore durante il parsing del salvataggio:', error);
            }
        }
    }
    
    // Ordina i salvataggi per data (dal più recente al più vecchio)
    saves.sort((a, b) => new Date(b.timestamp) - new Date(a.timestamp));
    
    return saves;
}

// Aggiorna la lista dei salvataggi (per un'eventuale UI dedicata)
function updateSavesList() {
    // Questa funzione potrebbe essere utilizzata per aggiornare una UI dedicata ai salvataggi
    // Per ora, stampiamo solo i salvataggi nella console
    const saves = getSavedGames();
    console.log('Salvataggi disponibili:', saves.length);
    saves.forEach((save, index) => {
        console.log(`${index + 1}. ${new Date(save.timestamp).toLocaleString()}`);
    });
}
```

**Spiegazione:**
- Implementiamo la funzione `saveGame` che salva lo stato corrente del gioco nel localStorage
- Implementiamo la funzione `loadGame` che carica uno stato salvato dal localStorage
- Implementiamo la funzione `getSavedGames` che ottiene la lista dei salvataggi disponibili
- Implementiamo la funzione `updateSavesList` che aggiorna la lista dei salvataggi (per un'eventuale UI dedicata)

### JavaScript: Gestione degli eventi per i nuovi pulsanti

```javascript
// Riferimenti agli elementi DOM
const saveGameBtn = document.getElementById('saveGameBtn');
const loadGameBtn = document.getElementById('loadGameBtn');

// Aggiungi event listener per i pulsanti di salvataggio e caricamento
saveGameBtn.addEventListener('click', saveGame);
loadGameBtn.addEventListener('click', loadGame);
```

**Spiegazione:**
- Otteniamo riferimenti ai nuovi pulsanti di salvataggio e caricamento
- Aggiungiamo event listener per gestire i click su questi pulsanti

### JavaScript: Serializzazione e deserializzazione dello stato del gioco

```javascript
// Serializza lo stato del gioco per il salvataggio
function serializeGameState() {
    // Crea una copia profonda dello stato del gioco
    const serializedState = JSON.stringify({
        board: gameState.board,
        currentPlayer: gameState.currentPlayer,
        moveHistory: gameState.moveHistory
    });
    
    return serializedState;
}

// Deserializza lo stato del gioco dal salvataggio
function deserializeGameState(serializedState) {
    try {
        const parsedState = JSON.parse(serializedState);
        
        // Verifica che lo stato deserializzato sia valido
        if (!parsedState.board || !parsedState.currentPlayer) {
            throw new Error('Stato del gioco non valido');
        }
        
        return parsedState;
    } catch (error) {
        console.error('Errore durante la deserializzazione:', error);
        throw error;
    }
}
```

**Spiegazione:**
- Implementiamo la funzione `serializeGameState` che converte lo stato del gioco in una stringa JSON
- Implementiamo la funzione `deserializeGameState` che converte una stringa JSON in uno stato del gioco
- Queste funzioni sono utili per garantire che lo stato del gioco sia correttamente salvato e caricato

### JavaScript: Gestione automatica del salvataggio

```javascript
// Salva automaticamente lo stato del gioco dopo ogni mossa
function autoSaveGame() {
    try {
        // Salva lo stato corrente nel localStorage con una chiave fissa
        localStorage.setItem('dama_autosave', serializeGameState());
        console.log('Partita salvata automaticamente');
    } catch (error) {
        console.error('Errore durante il salvataggio automatico:', error);
    }
}

// Carica l'ultimo stato salvato automaticamente
function loadAutoSave() {
    try {
        const autoSave = localStorage.getItem('dama_autosave');
        
        if (!autoSave) {
            console.log('Nessun salvataggio automatico disponibile');
            return false;
        }
        
        const savedState = deserializeGameState(autoSave);
        
        // Ripristina lo stato del gioco
        gameState.board = savedState.board;
        gameState.currentPlayer = savedState.currentPlayer;
        gameState.moveHistory = savedState.moveHistory;
        gameState.selectedPiece = null;
        gameState.validMoves = [];
        
        // Aggiorna le informazioni di gioco e ridisegna la scacchiera
        updateGameInfo();
        drawBoard();
        
        console.log('Partita caricata automaticamente');
        return true;
    } catch (error) {
        console.error('Errore durante il caricamento automatico:', error);
        return false;
    }
}

// Aggiorna la funzione movePiece per includere il salvataggio automatico
function movePiece(fromRow, fromCol, toRow, toCol, captures = []) {
    // ... codice esistente ...
    
    // Salva automaticamente dopo ogni mossa
    autoSaveGame();
    
    // ... resto del codice esistente ...
}

// Verifica se c'è un salvataggio automatico all'avvio del gioco
window.addEventListener('load', () => {
    // Chiedi all'utente se vuole caricare l'ultimo salvataggio automatico
    if (localStorage.getItem('dama_autosave')) {
        if (confirm('È stata trovata una partita salvata automaticamente. Vuoi caricarla?')) {
            loadAutoSave();
        } else {
            // Se l'utente rifiuta, inizia una nuova partita
            newGame();
        }
    } else {
        // Se non c'è un salvataggio automatico, inizia una nuova partita
        newGame();
    }
});
```

**Spiegazione:**
- Implementiamo la funzione `autoSaveGame` che salva automaticamente lo stato del gioco dopo ogni mossa
- Implementiamo la funzione `loadAutoSave` che carica l'ultimo stato salvato automaticamente
- Aggiorniamo la funzione `movePiece` per includere il salvataggio automatico
- Aggiungiamo un event listener che verifica se c'è un salvataggio automatico all'avvio del gioco

## Esercizi pratici

1. **Gestione di più profili**: Implementa un sistema che permetta di salvare partite associate a diversi profili utente.

2. **Esportazione e importazione**: Aggiungi la possibilità di esportare i salvataggi in file JSON e di importarli successivamente.

3. **Cronologia delle partite**: Implementa una funzionalità che permetta di visualizzare la cronologia delle partite giocate, con statistiche come il numero di mosse, il vincitore, ecc.

4. **Replay delle partite**: Implementa una funzionalità che permetta di rivedere una partita mossa per mossa, come un replay.

## Approfondimenti

- **IndexedDB**: Esplora l'utilizzo di IndexedDB per memorizzare grandi quantità di dati strutturati nel browser
- **Firebase**: Studia come utilizzare Firebase per implementare un sistema di salvataggio cloud-based
- **Compressione dei dati**: Approfondisci tecniche di compressione per ridurre la dimensione dei dati salvati

## Conclusione

In questo ottavo step, abbiamo implementato la persistenza dei dati nel gioco della dama. Abbiamo imparato a utilizzare il localStorage per salvare e caricare lo stato del gioco, a gestire più salvataggi e a implementare un sistema di salvataggio automatico. Queste funzionalità migliorano significativamente l'esperienza utente, permettendo di interrompere e riprendere una partita in qualsiasi momento. Nel prossimo step, esploreremo tecniche avanzate per migliorare l'interfaccia utente e l'esperienza di gioco.