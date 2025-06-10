```
# Persistenza dei Dati nel Gioco della Dama: Guida Approfondita

## Introduzione

La persistenza dei dati è un aspetto fondamentale nello sviluppo di giochi, in quanto permette di salvare lo stato del gioco e riprenderlo successivamente. Nel contesto del gioco della dama, la persistenza dei dati consente di salvare partite in corso, registrare statistiche di gioco, implementare funzionalità di replay e molto altro. In questa guida, esploreremo diverse tecniche per implementare la persistenza dei dati in un gioco della dama sviluppato in JavaScript.

## Tecniche di Persistenza dei Dati in JavaScript

JavaScript offre diverse opzioni per salvare i dati nel browser, ognuna con i propri vantaggi e limitazioni. Vediamo le principali tecniche disponibili:

### 1. Web Storage API

La Web Storage API fornisce due meccanismi per memorizzare dati nel browser:

#### localStorage

`localStorage` permette di memorizzare coppie chiave-valore che persistono anche dopo la chiusura del browser. I dati salvati in localStorage non hanno una data di scadenza.

```javascript
// Salvataggio dello stato del gioco
function saveGameState() {
    const gameState = {
        board: board,
        currentPlayer: currentPlayer,
        capturedPieces: capturedPieces,
        timestamp: new Date().toISOString()
    };
    
    localStorage.setItem('damaGameState', JSON.stringify(gameState));
    console.log('Stato del gioco salvato con successo!');
}

// Caricamento dello stato del gioco
function loadGameState() {
    const savedState = localStorage.getItem('damaGameState');
    
    if (savedState) {
        const gameState = JSON.parse(savedState);
        board = gameState.board;
        currentPlayer = gameState.currentPlayer;
        capturedPieces = gameState.capturedPieces;
        
        // Aggiorna la visualizzazione
        drawBoard();
        updateGameInfo();
        
        console.log(`Partita caricata dal salvataggio del ${new Date(gameState.timestamp).toLocaleString()}`);
        return true;
    }
    
    console.log('Nessun salvataggio trovato.');
    return false;
}
```

#### sessionStorage

`sessionStorage` funziona in modo simile a localStorage, ma i dati vengono cancellati quando la sessione del browser termina (quando l'utente chiude la scheda o il browser).

```javascript
// Salvataggio temporaneo di una mossa
function saveTemporaryMove(fromRow, fromCol, toRow, toCol) {
    const move = { fromRow, fromCol, toRow, toCol, timestamp: Date.now() };
    
    // Ottieni la cronologia delle mosse o inizializza un array vuoto
    const movesHistory = JSON.parse(sessionStorage.getItem('damaMoves') || '[]');
    
    // Aggiungi la nuova mossa
    movesHistory.push(move);
    
    // Salva la cronologia aggiornata
    sessionStorage.setItem('damaMoves', JSON.stringify(movesHistory));
}

// Recupero della cronologia delle mosse
function getMovesHistory() {
    return JSON.parse(sessionStorage.getItem('damaMoves') || '[]');
}
```

### 2. IndexedDB

IndexedDB è un database NoSQL integrato nel browser, più potente e flessibile rispetto a Web Storage, ideale per memorizzare grandi quantità di dati strutturati.

```javascript
// Inizializzazione del database
let db;
const request = indexedDB.open('DamaGameDB', 1);

request.onerror = function(event) {
    console.error('Errore nell\'apertura del database:', event.target.error);
};

request.onupgradeneeded = function(event) {
    db = event.target.result;
    
    // Crea un object store per le partite salvate
    if (!db.objectStoreNames.contains('savedGames')) {
        const objectStore = db.createObjectStore('savedGames', { keyPath: 'id', autoIncrement: true });
        
        // Crea indici per facilitare la ricerca
        objectStore.createIndex('timestamp', 'timestamp', { unique: false });
        objectStore.createIndex('playerName', 'playerName', { unique: false });
    }
};

request.onsuccess = function(event) {
    db = event.target.result;
    console.log('Database inizializzato con successo!');
};

// Salvataggio di una partita
function saveGame(playerName, description = '') {
    const transaction = db.transaction(['savedGames'], 'readwrite');
    const objectStore = transaction.objectStore('savedGames');
    
    const gameData = {
        playerName: playerName,
        description: description,
        timestamp: new Date().toISOString(),
        gameState: {
            board: board,
            currentPlayer: currentPlayer,
            capturedPieces: capturedPieces
        }
    };
    
    const request = objectStore.add(gameData);
    
    request.onsuccess = function(event) {
        console.log('Partita salvata con ID:', event.target.result);
    };
    
    request.onerror = function(event) {
        console.error('Errore nel salvataggio della partita:', event.target.error);
    };
}

// Caricamento di una partita specifica
function loadGame(gameId) {
    const transaction = db.transaction(['savedGames'], 'readonly');
    const objectStore = transaction.objectStore('savedGames');
    const request = objectStore.get(gameId);
    
    request.onsuccess = function(event) {
        const gameData = event.target.result;
        
        if (gameData) {
            // Ripristina lo stato del gioco
            board = gameData.gameState.board;
            currentPlayer = gameData.gameState.currentPlayer;
            capturedPieces = gameData.gameState.capturedPieces;
            
            // Aggiorna la visualizzazione
            drawBoard();
            updateGameInfo();
            
            console.log(`Partita di ${gameData.playerName} caricata con successo!`);
        } else {
            console.log('Partita non trovata.');
        }
    };
    
    request.onerror = function(event) {
        console.error('Errore nel caricamento della partita:', event.target.error);
    };
}

// Elenco delle partite salvate
function listSavedGames() {
    const transaction = db.transaction(['savedGames'], 'readonly');
    const objectStore = transaction.objectStore('savedGames');
    const request = objectStore.openCursor();
    
    const savedGames = [];
    
    request.onsuccess = function(event) {
        const cursor = event.target.result;
        
        if (cursor) {
            savedGames.push({
                id: cursor.key,
                playerName: cursor.value.playerName,
                description: cursor.value.description,
                timestamp: cursor.value.timestamp
            });
            
            cursor.continue();
        } else {
            // Visualizza l'elenco delle partite
            displaySavedGamesList(savedGames);
        }
    };
}
```

### 3. File API

La File API consente di generare file che l'utente può scaricare e successivamente caricare per ripristinare lo stato del gioco.

```javascript
// Esportazione dello stato del gioco in un file JSON
function exportGameToFile() {
    const gameState = {
        board: board,
        currentPlayer: currentPlayer,
        capturedPieces: capturedPieces,
        timestamp: new Date().toISOString(),
        version: '1.0' // Versione del formato di salvataggio
    };
    
    const gameStateJSON = JSON.stringify(gameState, null, 2);
    const blob = new Blob([gameStateJSON], { type: 'application/json' });
    const url = URL.createObjectURL(blob);
    
    // Crea un link per il download e simula il click
    const a = document.createElement('a');
    a.href = url;
    a.download = `dama_salvataggio_${new Date().toISOString().replace(/[:.]/g, '-')}.json`;
    document.body.appendChild(a);
    a.click();
    
    // Pulizia
    setTimeout(() => {
        document.body.removeChild(a);
        URL.revokeObjectURL(url);
    }, 0);
}

// Importazione dello stato del gioco da un file JSON
function importGameFromFile(file) {
    const reader = new FileReader();
    
    reader.onload = function(event) {
        try {
            const gameState = JSON.parse(event.target.result);
            
            // Verifica la versione del formato
            if (gameState.version !== '1.0') {
                console.warn('Il file di salvataggio potrebbe essere incompatibile.');
            }
            
            // Ripristina lo stato del gioco
            board = gameState.board;
            currentPlayer = gameState.currentPlayer;
            capturedPieces = gameState.capturedPieces;
            
            // Aggiorna la visualizzazione
            drawBoard();
            updateGameInfo();
            
            console.log(`Partita caricata dal file (salvata il ${new Date(gameState.timestamp).toLocaleString()})`);
        } catch (error) {
            console.error('Errore nel parsing del file:', error);
            alert('Il file selezionato non è un salvataggio valido.');
        }
    };
    
    reader.onerror = function() {
        console.error('Errore nella lettura del file.');
    };
    
    reader.readAsText(file);
}

// Gestione dell'input file
document.getElementById('importGameFile').addEventListener('change', function(event) {
    const file = event.target.files[0];
    if (file) {
        importGameFromFile(file);
    }
});
```

## Implementazione di un Sistema di Salvataggio Completo

Vediamo ora come implementare un sistema di salvataggio completo per il gioco della dama, che combina diverse tecniche di persistenza dei dati.

### Interfaccia Utente per la Gestione dei Salvataggi

```html
<div class="game-controls">
    <button id="newGameBtn">Nuova Partita</button>
    <button id="saveGameBtn">Salva Partita</button>
    <button id="loadGameBtn">Carica Partita</button>
    <button id="exportGameBtn">Esporta Partita</button>
    <input type="file" id="importGameFile" accept=".json" style="display: none;">
    <button id="importGameBtn">Importa Partita</button>
</div>

<div id="saveGameModal" class="modal">
    <div class="modal-content">
        <span class="close">&times;</span>
        <h2>Salva Partita</h2>
        <form id="saveGameForm">
            <div class="form-group">
                <label for="playerName">Nome Giocatore:</label>
                <input type="text" id="playerName" required>
            </div>
            <div class="form-group">
                <label for="gameDescription">Descrizione:</label>
                <textarea id="gameDescription"></textarea>
            </div>
            <button type="submit">Salva</button>
        </form>
    </div>
</div>

<div id="loadGameModal" class="modal">
    <div class="modal-content">
        <span class="close">&times;</span>
        <h2>Carica Partita</h2>
        <div id="savedGamesList"></div>
    </div>
</div>
```

### JavaScript per la Gestione dell'Interfaccia

```javascript
// Riferimenti agli elementi DOM
const saveGameBtn = document.getElementById('saveGameBtn');
const loadGameBtn = document.getElementById('loadGameBtn');
const exportGameBtn = document.getElementById('exportGameBtn');
const importGameBtn = document.getElementById('importGameBtn');
const importGameFile = document.getElementById('importGameFile');

const saveGameModal = document.getElementById('saveGameModal');
const loadGameModal = document.getElementById('loadGameModal');
const saveGameForm = document.getElementById('saveGameForm');
const savedGamesList = document.getElementById('savedGamesList');

// Gestione dei pulsanti
saveGameBtn.addEventListener('click', () => {
    saveGameModal.style.display = 'block';
});

loadGameBtn.addEventListener('click', () => {
    loadGameModal.style.display = 'block';
    listSavedGames(); // Carica l'elenco delle partite salvate
});

exportGameBtn.addEventListener('click', exportGameToFile);

importGameBtn.addEventListener('click', () => {
    importGameFile.click();
});

// Gestione del form di salvataggio
saveGameForm.addEventListener('submit', (event) => {
    event.preventDefault();
    
    const playerName = document.getElementById('playerName').value;
    const description = document.getElementById('gameDescription').value;
    
    saveGame(playerName, description);
    saveGameModal.style.display = 'none';
});

// Chiusura delle modali
document.querySelectorAll('.close').forEach(closeBtn => {
    closeBtn.addEventListener('click', () => {
        saveGameModal.style.display = 'none';
        loadGameModal.style.display = 'none';
    });
});

// Visualizzazione dell'elenco delle partite salvate
function displaySavedGamesList(games) {
    savedGamesList.innerHTML = '';
    
    if (games.length === 0) {
        savedGamesList.innerHTML = '<p>Nessuna partita salvata.</p>';
        return;
    }
    
    const table = document.createElement('table');
    table.innerHTML = `
        <thead>
            <tr>
                <th>Giocatore</th>
                <th>Descrizione</th>
                <th>Data</th>
                <th>Azioni</th>
            </tr>
        </thead>
        <tbody></tbody>
    `;
    
    const tbody = table.querySelector('tbody');
    
    games.forEach(game => {
        const row = document.createElement('tr');
        row.innerHTML = `
            <td>${game.playerName}</td>
            <td>${game.description || '-'}</td>
            <td>${new Date(game.timestamp).toLocaleString()}</td>
            <td>
                <button class="load-game-btn" data-id="${game.id}">Carica</button>
                <button class="delete-game-btn" data-id="${game.id}">Elimina</button>
            </td>
        `;
        
        tbody.appendChild(row);
    });
    
    savedGamesList.appendChild(table);
    
    // Aggiungi event listener ai pulsanti
    table.querySelectorAll('.load-game-btn').forEach(btn => {
        btn.addEventListener('click', () => {
            const gameId = parseInt(btn.getAttribute('data-id'));
            loadGame(gameId);
            loadGameModal.style.display = 'none';
        });
    });
    
    table.querySelectorAll('.delete-game-btn').forEach(btn => {
        btn.addEventListener('click', () => {
            const gameId = parseInt(btn.getAttribute('data-id'));
            deleteGame(gameId);
        });
    });
}

// Eliminazione di una partita salvata
function deleteGame(gameId) {
    const transaction = db.transaction(['savedGames'], 'readwrite');
    const objectStore = transaction.objectStore('savedGames');
    const request = objectStore.delete(gameId);
    
    request.onsuccess = function() {
        console.log('Partita eliminata con successo!');
        listSavedGames(); // Aggiorna l'elenco
    };
    
    request.onerror = function(event) {
        console.error('Errore nell\'eliminazione della partita:', event.target.error);
    };
}
```

## Salvataggio Automatico e Ripristino

Un'altra funzionalità utile è il salvataggio automatico, che permette di ripristinare lo stato del gioco in caso di chiusura accidentale del browser o di interruzione della connessione.

```javascript
// Configurazione del salvataggio automatico
const AUTO_SAVE_INTERVAL = 60000; // 1 minuto
let autoSaveTimer;

// Avvio del salvataggio automatico
function startAutoSave() {
    // Interrompi eventuali timer precedenti
    stopAutoSave();
    
    // Avvia un nuovo timer
    autoSaveTimer = setInterval(() => {
        // Salva lo stato corrente in localStorage
        const gameState = {
            board: board,
            currentPlayer: currentPlayer,
            capturedPieces: capturedPieces,
            timestamp: new Date().toISOString()
        };
        
        localStorage.setItem('damaAutoSave', JSON.stringify(gameState));
        console.log('Salvataggio automatico completato:', new Date().toLocaleString());
    }, AUTO_SAVE_INTERVAL);
    
    console.log('Salvataggio automatico attivato.');
}

// Interruzione del salvataggio automatico
function stopAutoSave() {
    if (autoSaveTimer) {
        clearInterval(autoSaveTimer);
        autoSaveTimer = null;
        console.log('Salvataggio automatico disattivato.');
    }
}

// Verifica se esiste un salvataggio automatico all'avvio del gioco
function checkForAutoSave() {
    const autoSave = localStorage.getItem('damaAutoSave');
    
    if (autoSave) {
        const gameState = JSON.parse(autoSave);
        const saveTime = new Date(gameState.timestamp);
        const now = new Date();
        const timeDiff = (now - saveTime) / (1000 * 60); // Differenza in minuti
        
        // Chiedi all'utente se vuole ripristinare il salvataggio automatico
        if (confirm(`È stato trovato un salvataggio automatico di ${timeDiff.toFixed(1)} minuti fa. Vuoi ripristinarlo?`)) {
            board = gameState.board;
            currentPlayer = gameState.currentPlayer;
            capturedPieces = gameState.capturedPieces;
            
            // Aggiorna la visualizzazione
            drawBoard();
            updateGameInfo();
            
            console.log('Salvataggio automatico ripristinato.');
        } else {
            // Se l'utente rifiuta, elimina il salvataggio automatico
            localStorage.removeItem('damaAutoSave');
            console.log('Salvataggio automatico ignorato e rimosso.');
        }
    }
}

// Avvia il controllo del salvataggio automatico all'inizializzazione del gioco
document.addEventListener('DOMContentLoaded', () => {
    initGame();
    checkForAutoSave();
    startAutoSave();
});
```

## Sincronizzazione con il Cloud

Per un'esperienza più avanzata, è possibile implementare la sincronizzazione dei salvataggi con un servizio cloud, permettendo agli utenti di accedere alle proprie partite da diversi dispositivi.

```javascript
// Esempio di sincronizzazione con un servizio backend (pseudocodice)
async function syncSavedGames() {
    // Verifica se l'utente è autenticato
    if (!isUserLoggedIn()) {
        console.log('Sincronizzazione non disponibile: utente non autenticato.');
        return;
    }
    
    try {
        // Ottieni l'elenco delle partite salvate localmente
        const localGames = await getAllSavedGames();
        
        // Invia le partite al server
        const response = await fetch('https://api.example.com/sync', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
                'Authorization': `Bearer ${getUserToken()}`
            },
            body: JSON.stringify({
                userId: getUserId(),
                games: localGames
            })
        });
        
        if (!response.ok) {
            throw new Error('Errore nella sincronizzazione con il server.');
        }
        
        const result = await response.json();
        
        // Aggiorna le partite locali con quelle dal server
        await updateLocalGames(result.games);
        
        console.log('Sincronizzazione completata con successo!');
    } catch (error) {
        console.error('Errore durante la sincronizzazione:', error);
    }
}
```

## Considerazioni sulla Sicurezza e Privacy

Quando si implementa la persistenza dei dati, è importante considerare alcuni aspetti di sicurezza e privacy:

1. **Limitazioni di spazio**: localStorage e sessionStorage hanno un limite di spazio (generalmente 5-10 MB per dominio).
2. **Sicurezza dei dati**: i dati salvati nel browser sono accessibili a JavaScript e potrebbero essere manipolati.
3. **Privacy**: informare gli utenti sui dati che vengono salvati e ottenere il consenso quando necessario.
4. **Pulizia dei dati**: implementare meccanismi per eliminare dati obsoleti o non più necessari.

## Conclusione

La persistenza dei dati è un elemento fondamentale per migliorare l'esperienza utente nei giochi web come la dama. In questa guida, abbiamo esplorato diverse tecniche per salvare e ripristinare lo stato del gioco, dalla semplice memorizzazione in localStorage all'utilizzo di database più complessi come IndexedDB, fino all'esportazione e importazione di file e alla sincronizzazione con servizi cloud.

Implementando queste funzionalità, è possibile creare un'esperienza di gioco più completa e professionale, permettendo agli utenti di salvare i propri progressi, riprendere partite interrotte e persino condividere le proprie partite tra diversi dispositivi.

## Esercizi Pratici

1. **Sistema di salvataggio base**: Implementa un sistema di salvataggio e caricamento utilizzando localStorage.
2. **Gestione dei salvataggi multipli**: Estendi il sistema per supportare più slot di salvataggio, con nomi e descrizioni personalizzati.
3. **Esportazione e importazione**: Aggiungi la possibilità di esportare e importare partite come file JSON.
4. **Replay delle partite**: Implementa un sistema per registrare tutte le mosse di una partita e riprodurle in sequenza.
5. **Sincronizzazione**: Se hai familiarità con lo sviluppo backend, crea un semplice servizio per sincronizzare i salvataggi tra diversi dispositivi.

## Risorse Aggiuntive

- [MDN Web Storage API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API)
- [MDN IndexedDB API](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API)
- [MDN File API](https://developer.mozilla.org/en-US/docs/Web/API/File_API)
- [Web Storage Specification](https://html.spec.whatwg.org/multipage/webstorage.html)
- [IndexedDB Specification](https://www.w3.org/TR/IndexedDB/)
```