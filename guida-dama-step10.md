# Guida Didattica: Dama in JavaScript - Step 10: Sicurezza e Protezione Dati

## Obiettivo
In questo decimo e ultimo step, implementeremo misure di sicurezza e protezione dei dati nel gioco della dama. Queste misure sono fondamentali per proteggere i dati degli utenti, prevenire manipolazioni non autorizzate e garantire un'esperienza di gioco sicura e affidabile.

## Concetti teorici da approfondire

### 1. Sicurezza dei dati nel browser
- **Same-Origin Policy**: Come funziona la politica di sicurezza dello stesso origine nei browser
- **Content Security Policy (CSP)**: Come implementare politiche di sicurezza dei contenuti per prevenire attacchi XSS
- **Secure Cookies**: Come utilizzare cookie sicuri per memorizzare dati sensibili

### 2. Protezione contro la manipolazione dei dati
- **Validazione dei dati**: Come verificare l'integrità e la validità dei dati ricevuti
- **Sanitizzazione degli input**: Come prevenire attacchi di injection e XSS
- **Firma digitale**: Come utilizzare firme digitali per verificare l'autenticità dei dati

### 3. Crittografia dei dati
- **Web Crypto API**: Come utilizzare l'API di crittografia del browser
- **Crittografia simmetrica e asimmetrica**: Differenze e casi d'uso
- **Hashing**: Come utilizzare funzioni di hash per proteggere dati sensibili

## Analisi del codice

### JavaScript: Implementazione della validazione dei dati

```javascript
// Validazione dei dati
const Validator = {
    // Verifica se un oggetto ha la struttura attesa
    validateStructure: function(obj, schema) {
        if (typeof obj !== 'object' || obj === null) {
            return false;
        }
        
        for (const key in schema) {
            if (schema.hasOwnProperty(key)) {
                // Verifica se la proprietà esiste
                if (!obj.hasOwnProperty(key)) {
                    return false;
                }
                
                // Verifica il tipo della proprietà
                const expectedType = schema[key];
                if (typeof expectedType === 'string') {
                    // Tipo semplice
                    if (typeof obj[key] !== expectedType) {
                        return false;
                    }
                } else if (Array.isArray(expectedType)) {
                    // Array
                    if (!Array.isArray(obj[key])) {
                        return false;
                    }
                    
                    // Se è specificato un tipo per gli elementi dell'array
                    if (expectedType.length > 0) {
                        const elementType = expectedType[0];
                        for (const element of obj[key]) {
                            if (typeof elementType === 'string') {
                                if (typeof element !== elementType) {
                                    return false;
                                }
                            } else if (typeof elementType === 'object') {
                                if (!this.validateStructure(element, elementType)) {
                                    return false;
                                }
                            }
                        }
                    }
                } else if (typeof expectedType === 'object') {
                    // Oggetto annidato
                    if (!this.validateStructure(obj[key], expectedType)) {
                        return false;
                    }
                }
            }
        }
        
        return true;
    },
    
    // Schema per lo stato del gioco
    gameStateSchema: {
        board: [],
        currentPlayer: 'string',
        moveHistory: [],
        selectedPiece: 'object',
        validMoves: []
    },
    
    // Schema per una pedina
    pieceSchema: {
        color: 'string',
        isKing: 'boolean'
    },
    
    // Schema per una mossa
    moveSchema: {
        row: 'number',
        col: 'number',
        captures: []
    },
    
    // Verifica se lo stato del gioco è valido
    validateGameState: function(gameState) {
        // Verifica la struttura di base
        if (!this.validateStructure(gameState, this.gameStateSchema)) {
            return false;
        }
        
        // Verifica la scacchiera
        if (!Array.isArray(gameState.board) || gameState.board.length !== BOARD_SIZE) {
            return false;
        }
        
        for (const row of gameState.board) {
            if (!Array.isArray(row) || row.length !== BOARD_SIZE) {
                return false;
            }
            
            for (const cell of row) {
                if (cell !== null && !this.validateStructure(cell, this.pieceSchema)) {
                    return false;
                }
            }
        }
        
        // Verifica il giocatore corrente
        if (gameState.currentPlayer !== WHITE && gameState.currentPlayer !== BLACK) {
            return false;
        }
        
        // Verifica la cronologia delle mosse
        if (!Array.isArray(gameState.moveHistory)) {
            return false;
        }
        
        // Verifica le mosse valide
        if (!Array.isArray(gameState.validMoves)) {
            return false;
        }
        
        for (const move of gameState.validMoves) {
            if (!this.validateStructure(move, this.moveSchema)) {
                return false;
            }
        }
        
        return true;
    },
    
    // Sanitizza una stringa per prevenire XSS
    sanitizeString: function(str) {
        if (typeof str !== 'string') {
            return '';
        }
        
        return str
            .replace(/&/g, '&amp;')
            .replace(/</g, '&lt;')
            .replace(/>/g, '&gt;')
            .replace(/"/g, '&quot;')
            .replace(/'/g, '&#x27;')
            .replace(/\//g, '&#x2F;');
    }
};
```

**Spiegazione:**
- Implementiamo un oggetto `Validator` che fornisce funzionalità per validare la struttura e l'integrità dei dati
- Definiamo schemi per lo stato del gioco, le pedine e le mosse
- Implementiamo funzioni per verificare se i dati rispettano gli schemi definiti
- Aggiungiamo una funzione per sanitizzare le stringhe e prevenire attacchi XSS

### JavaScript: Implementazione della crittografia dei dati

```javascript
// Crittografia dei dati
const Crypto = {
    // Chiave segreta (in un'applicazione reale, questa dovrebbe essere generata in modo sicuro)
    secretKey: null,
    
    // Inizializza la chiave segreta
    async init() {
        try {
            // Genera una chiave casuale se non esiste già
            if (!localStorage.getItem('dama_crypto_key')) {
                const array = new Uint8Array(16);
                window.crypto.getRandomValues(array);
                const key = Array.from(array, byte => byte.toString(16).padStart(2, '0')).join('');
                localStorage.setItem('dama_crypto_key', key);
            }
            
            this.secretKey = localStorage.getItem('dama_crypto_key');
            Logger.info('Chiave di crittografia inizializzata');
        } catch (error) {
            Logger.error('Errore durante l\'inizializzazione della chiave di crittografia', error);
            // Fallback a una chiave predefinita (meno sicuro)
            this.secretKey = 'dama_default_key_2023';
        }
    },
    
    // Calcola l'hash di una stringa
    async hash(data) {
        try {
            const encoder = new TextEncoder();
            const dataBuffer = encoder.encode(data);
            const hashBuffer = await window.crypto.subtle.digest('SHA-256', dataBuffer);
            const hashArray = Array.from(new Uint8Array(hashBuffer));
            const hashHex = hashArray.map(b => b.toString(16).padStart(2, '0')).join('');
            return hashHex;
        } catch (error) {
            Logger.error('Errore durante il calcolo dell\'hash', error);
            // Fallback a un hash semplice (meno sicuro)
            return this.simpleHash(data);
        }
    },
    
    // Hash semplice (fallback)
    simpleHash(data) {
        let hash = 0;
        for (let i = 0; i < data.length; i++) {
            const char = data.charCodeAt(i);
            hash = ((hash << 5) - hash) + char;
            hash = hash & hash; // Converti in intero a 32 bit
        }
        return hash.toString(16);
    },
    
    // Cripta una stringa
    encrypt(data) {
        if (!this.secretKey) {
            Logger.warning('Chiave di crittografia non inizializzata');
            return data;
        }
        
        try {
            // Implementazione semplice di XOR (in un'applicazione reale, utilizzare algoritmi più robusti)
            const encrypted = [];
            for (let i = 0; i < data.length; i++) {
                const charCode = data.charCodeAt(i);
                const keyChar = this.secretKey.charCodeAt(i % this.secretKey.length);
                encrypted.push(String.fromCharCode(charCode ^ keyChar));
            }
            return btoa(encrypted.join('')); // Codifica in base64
        } catch (error) {
            Logger.error('Errore durante la crittografia', error);
            return data;
        }
    },
    
    // Decripta una stringa
    decrypt(encryptedData) {
        if (!this.secretKey) {
            Logger.warning('Chiave di crittografia non inizializzata');
            return encryptedData;
        }
        
        try {
            const data = atob(encryptedData); // Decodifica da base64
            const decrypted = [];
            for (let i = 0; i < data.length; i++) {
                const charCode = data.charCodeAt(i);
                const keyChar = this.secretKey.charCodeAt(i % this.secretKey.length);
                decrypted.push(String.fromCharCode(charCode ^ keyChar));
            }
            return decrypted.join('');
        } catch (error) {
            Logger.error('Errore durante la decrittografia', error);
            return encryptedData;
        }
    },
    
    // Firma un oggetto
    async sign(data) {
        const dataString = JSON.stringify(data);
        const hash = await this.hash(dataString + this.secretKey);
        return {
            data: data,
            signature: hash
        };
    },
    
    // Verifica la firma di un oggetto
    async verify(signedData) {
        if (!signedData || !signedData.data || !signedData.signature) {
            return false;
        }
        
        const dataString = JSON.stringify(signedData.data);
        const hash = await this.hash(dataString + this.secretKey);
        return hash === signedData.signature;
    }
};

// Inizializza la crittografia all'avvio
Crypto.init();
```

**Spiegazione:**
- Implementiamo un oggetto `Crypto` che fornisce funzionalità di crittografia e firma digitale
- Utilizziamo l'API Web Crypto per generare chiavi casuali e calcolare hash
- Implementiamo funzioni per criptare e decriptare dati
- Aggiungiamo funzioni per firmare e verificare l'autenticità dei dati

### JavaScript: Aggiornamento delle funzioni di salvataggio e caricamento

```javascript
// Salva lo stato del gioco nel localStorage in modo sicuro
async function saveGame() {
    try {
        // Valida lo stato del gioco prima di salvarlo
        if (!Validator.validateGameState(gameState)) {
            throw new Error('Stato del gioco non valido');
        }
        
        // Crea un oggetto con lo stato corrente del gioco
        const saveData = {
            board: gameState.board,
            currentPlayer: gameState.currentPlayer,
            moveHistory: gameState.moveHistory,
            timestamp: new Date().toISOString()
        };
        
        // Firma i dati
        const signedData = await Crypto.sign(saveData);
        
        // Cripta i dati firmati
        const encryptedData = Crypto.encrypt(JSON.stringify(signedData));
        
        // Genera un ID univoco per il salvataggio
        const saveId = 'dama_save_' + new Date().getTime();
        
        // Salva i dati nel localStorage
        localStorage.setItem(saveId, encryptedData);
        
        // Aggiorna la lista dei salvataggi
        updateSavesList();
        
        // Feedback all'utente
        alert('Partita salvata con successo!');
        Logger.info('Partita salvata', { saveId });
    } catch (error) {
        Logger.error('Errore durante il salvataggio', error);
        alert('Errore durante il salvataggio della partita: ' + error.message);
    }
}

// Carica uno stato del gioco dal localStorage in modo sicuro
async function loadGame() {
    try {
        // Ottieni la lista dei salvataggi disponibili
        const saves = await getSavedGames();
        
        if (saves.length === 0) {
            alert('Nessuna partita salvata disponibile.');
            return;
        }
        
        // Crea una lista di salvataggi da mostrare all'utente
        let savesList = 'Seleziona una partita da caricare:\n';
        saves.forEach((save, index) => {
            const date = new Date(save.data.timestamp);
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
        const signedData = saves[index];
        
        // Verifica la firma
        const isValid = await Crypto.verify(signedData);
        if (!isValid) {
            throw new Error('La firma del salvataggio non è valida. Il salvataggio potrebbe essere stato manomesso.');
        }
        
        const saveData = signedData.data;
        
        // Valida i dati caricati
        if (!Validator.validateGameState({
            board: saveData.board,
            currentPlayer: saveData.currentPlayer,
            moveHistory: saveData.moveHistory,
            selectedPiece: null,
            validMoves: []
        })) {
            throw new Error('I dati del salvataggio non sono validi.');
        }
        
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
        Logger.info('Partita caricata', { timestamp: saveData.timestamp });
    } catch (error) {
        Logger.error('Errore durante il caricamento', error);
        alert('Errore durante il caricamento della partita: ' + error.message);
    }
}

// Ottieni la lista dei salvataggi disponibili in modo sicuro
async function getSavedGames() {
    const saves = [];
    
    // Itera su tutti gli elementi nel localStorage
    for (let i = 0; i < localStorage.length; i++) {
        const key = localStorage.key(i);
        
        // Verifica se la chiave corrisponde a un salvataggio del gioco
        if (key.startsWith('dama_save_')) {
            try {
                const encryptedData = localStorage.getItem(key);
                const decryptedData = Crypto.decrypt(encryptedData);
                const signedData = JSON.parse(decryptedData);
                
                // Verifica la firma
                const isValid = await Crypto.verify(signedData);
                if (isValid) {
                    signedData.id = key; // Aggiungi l'ID del salvataggio
                    saves.push(signedData);
                } else {
                    Logger.warning('Salvataggio con firma non valida', { key });
                }
            } catch (error) {
                Logger.error('Errore durante il parsing del salvataggio', { key, error });
            }
        }
    }
    
    // Ordina i salvataggi per data (dal più recente al più vecchio)
    saves.sort((a, b) => new Date(b.data.timestamp) - new Date(a.data.timestamp));
    
    return saves;
}
```

**Spiegazione:**
- Aggiorniamo le funzioni `saveGame`, `loadGame` e `getSavedGames` per includere misure di sicurezza
- Utilizziamo la validazione dei dati per verificare l'integrità dello stato del gioco
- Utilizziamo la crittografia per proteggere i dati salvati
- Utilizziamo firme digitali per verificare l'autenticità dei dati

### HTML: Implementazione di Content Security Policy

```html
<!-- Aggiungi questa meta tag nell'head del documento HTML -->
<meta http-equiv="Content-Security-Policy" content="default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'; img-src 'self' data:; connect-src 'none'">
```

**Spiegazione:**
- Implementiamo una Content Security Policy (CSP) per prevenire attacchi XSS
- Limitiamo le fonti da cui possono essere caricati script, stili, immagini e connessioni
- Questo aiuta a prevenire l'esecuzione di codice malevolo iniettato nel gioco

## Esercizi pratici

1. **Autenticazione utente**: Implementa un sistema di autenticazione che permetta agli utenti di registrarsi e accedere al gioco con username e password.

2. **Protezione contro il cheating**: Implementa misure per prevenire la manipolazione del gioco da parte degli utenti (ad esempio, modificando direttamente lo stato del gioco nel localStorage).

3. **Backup e ripristino**: Implementa funzionalità per esportare i salvataggi in file crittografati e importarli successivamente.

4. **Modalità privata**: Implementa una modalità che non salva dati nel localStorage, utile per gli utenti che desiderano maggiore privacy.

## Approfondimenti

- **OAuth e OpenID Connect**: Studia questi protocolli per implementare l'autenticazione tramite provider esterni (Google, Facebook, ecc.)
- **JWT (JSON Web Tokens)**: Approfondisci come utilizzare i JWT per l'autenticazione e l'autorizzazione
- **HTTPS e TLS**: Comprendi l'importanza di utilizzare HTTPS per proteggere le comunicazioni tra client e server

## Conclusione

In questo decimo e ultimo step, abbiamo implementato misure di sicurezza e protezione dei dati nel gioco della dama. Abbiamo imparato a validare e sanitizzare i dati, a utilizzare la crittografia per proteggere le informazioni sensibili e a implementare firme digitali per verificare l'autenticità dei dati. Abbiamo anche esplorato l'implementazione di Content Security Policy per prevenire attacchi XSS. Queste misure sono fondamentali per proteggere i dati degli utenti, prevenire manipolazioni non autorizzate e garantire un'esperienza di gioco sicura e affidabile.

Con questo step, completiamo il nostro percorso di sviluppo del gioco della dama in JavaScript. Abbiamo coperto tutti gli aspetti fondamentali dello sviluppo di un gioco web, dalla creazione della scacchiera all'implementazione delle regole del gioco, dalla gestione dell'interfaccia utente alla sicurezza e protezione dei dati. Speriamo che questo percorso ti abbia fornito le conoscenze e le competenze necessarie per sviluppare giochi web complessi e sicuri.