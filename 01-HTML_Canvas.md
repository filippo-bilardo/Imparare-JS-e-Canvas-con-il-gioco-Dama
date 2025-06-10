# HTML Canvas: Guida Approfondita

## Introduzione

L'elemento HTML Canvas è una delle caratteristiche più potenti introdotte con HTML5, che permette di disegnare grafica 2D e 3D dinamicamente tramite JavaScript. A differenza di altri elementi HTML che hanno un comportamento predefinito, Canvas è essenzialmente una tela bianca che richiede JavaScript per disegnare qualsiasi cosa al suo interno.

## Cos'è Canvas

Canvas è un elemento HTML che fornisce un'area di disegno bitmap controllabile tramite JavaScript. È rappresentato dal tag `<canvas>` e può essere manipolato utilizzando un'API di disegno specifica. Canvas è particolarmente utile per:

- Creare grafici e visualizzazioni di dati
- Sviluppare giochi 2D
- Generare animazioni
- Manipolare immagini in tempo reale
- Creare applicazioni di disegno

### Sintassi di base

```html
<canvas id="myCanvas" width="500" height="500"></canvas>
```

È importante notare che Canvas ha due set di dimensioni:

1. **Dimensioni CSS**: controllano quanto spazio occupa nel layout della pagina
2. **Dimensioni dell'area di disegno**: controllano la risoluzione effettiva del canvas

Se non specifichi le dimensioni nell'attributo `width` e `height`, Canvas avrà una dimensione predefinita di 300x150 pixel.

## Contesto di rendering

Per disegnare su un Canvas, è necessario ottenere un "contesto di rendering". Questo è un oggetto che fornisce i metodi e le proprietà necessari per disegnare sul canvas.

### Ottenere il contesto 2D

```javascript
const canvas = document.getElementById('myCanvas');
const ctx = canvas.getContext('2d');
```

Il metodo `getContext()` può accettare diversi parametri a seconda del tipo di contesto che si desidera ottenere:

- `'2d'`: per disegno 2D (il più comune)
- `'webgl'` o `'experimental-webgl'`: per rendering 3D
- `'webgl2'`: per WebGL versione 2
- `'bitmaprenderer'`: per rendering di immagini bitmap

### Proprietà principali del contesto 2D

- `fillStyle`: imposta il colore o lo stile per riempire le forme
- `strokeStyle`: imposta il colore o lo stile per i contorni delle forme
- `lineWidth`: imposta la larghezza delle linee
- `font`: imposta il font per il testo
- `globalAlpha`: imposta la trasparenza globale
- `shadowColor`, `shadowBlur`, `shadowOffsetX`, `shadowOffsetY`: controllano le ombre

### Metodi principali del contesto 2D

#### Disegno di rettangoli

- `fillRect(x, y, width, height)`: disegna un rettangolo pieno
- `strokeRect(x, y, width, height)`: disegna solo il contorno di un rettangolo
- `clearRect(x, y, width, height)`: cancella un'area rettangolare

#### Disegno di percorsi

- `beginPath()`: inizia un nuovo percorso
- `moveTo(x, y)`: sposta il punto di disegno a una nuova posizione
- `lineTo(x, y)`: aggiunge una linea al percorso
- `arc(x, y, radius, startAngle, endAngle, anticlockwise)`: aggiunge un arco al percorso
- `closePath()`: chiude il percorso
- `fill()`: riempie il percorso
- `stroke()`: disegna il contorno del percorso

#### Disegno di testo

- `fillText(text, x, y, maxWidth)`: disegna testo pieno
- `strokeText(text, x, y, maxWidth)`: disegna il contorno del testo
- `measureText(text)`: restituisce un oggetto con informazioni sulla larghezza del testo

#### Manipolazione delle immagini

- `drawImage()`: disegna un'immagine, un canvas o un video sul canvas

#### Trasformazioni

- `translate(x, y)`: sposta l'origine del canvas
- `rotate(angle)`: ruota il canvas
- `scale(x, y)`: scala il canvas
- `transform()` e `setTransform()`: applicano trasformazioni personalizzate

## Sistema di coordinate

Il sistema di coordinate in Canvas ha l'origine (0,0) nell'angolo in alto a sinistra. Le coordinate x aumentano verso destra e le coordinate y aumentano verso il basso.

```
(0,0) → x aumenta →
   ↓
   y
   aumenta
   ↓
```

Questo è diverso dal sistema di coordinate cartesiano tradizionale, dove l'origine è in basso a sinistra e le coordinate y aumentano verso l'alto.

### Esempio pratico: disegnare una griglia

```javascript
function drawGrid(ctx, width, height, cellSize) {
    ctx.beginPath();
    
    // Linee verticali
    for (let x = 0; x <= width; x += cellSize) {
        ctx.moveTo(x, 0);
        ctx.lineTo(x, height);
    }
    
    // Linee orizzontali
    for (let y = 0; y <= height; y += cellSize) {
        ctx.moveTo(0, y);
        ctx.lineTo(width, y);
    }
    
    ctx.strokeStyle = '#ddd';
    ctx.stroke();
}
```

## Animazioni con Canvas

Per creare animazioni con Canvas, è necessario:

1. Cancellare il canvas (o parte di esso)
2. Disegnare la nuova frame
3. Ripetere il processo

Questo viene tipicamente fatto utilizzando `requestAnimationFrame()` per sincronizzare l'animazione con il refresh rate del browser.

```javascript
let x = 0;
const canvas = document.getElementById('myCanvas');
const ctx = canvas.getContext('2d');

function animate() {
    // Cancella il canvas
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    
    // Disegna un rettangolo che si muove
    ctx.fillStyle = 'blue';
    ctx.fillRect(x, 50, 50, 50);
    
    // Aggiorna la posizione
    x = (x + 2) % canvas.width;
    
    // Richiama la funzione per la prossima frame
    requestAnimationFrame(animate);
}

// Avvia l'animazione
animate();
```

## Gestione degli eventi

Per interagire con il Canvas, è possibile aggiungere event listener per gestire eventi come click del mouse, movimento del mouse, pressione di tasti, ecc.

```javascript
canvas.addEventListener('click', function(event) {
    // Ottieni le coordinate del click relative al canvas
    const rect = canvas.getBoundingClientRect();
    const x = event.clientX - rect.left;
    const y = event.clientY - rect.top;
    
    // Fai qualcosa con le coordinate
    ctx.beginPath();
    ctx.arc(x, y, 10, 0, Math.PI * 2);
    ctx.fill();
});
```

## Ottimizzazione delle prestazioni

Le operazioni su Canvas possono essere costose in termini di prestazioni, specialmente per animazioni complesse. Ecco alcuni suggerimenti per ottimizzare le prestazioni:

1. **Limita l'area di ridisegno**: invece di cancellare l'intero canvas, cancella solo le aree che cambiano
2. **Usa più canvas sovrapposti**: uno per elementi statici e uno per elementi dinamici
3. **Riduci le chiamate al contesto**: raggruppa le operazioni di disegno simili
4. **Usa `requestAnimationFrame()` invece di `setTimeout()` o `setInterval()`**
5. **Disattiva la trasparenza** se non necessaria: `canvas.getContext('2d', { alpha: false })`
6. **Arrotonda le coordinate dei pixel** per evitare l'anti-aliasing

## Canvas vs SVG

Canvas e SVG sono entrambi utilizzati per creare grafica sul web, ma hanno differenze fondamentali:

| Canvas | SVG |
|--------|-----|
| Basato sui pixel (bitmap) | Basato sui vettori |
| Risoluzione dipendente | Risoluzione indipendente |
| Nessun DOM per gli elementi disegnati | Ogni elemento è nel DOM |
| Prestazioni migliori con molti elementi | Prestazioni migliori con pochi elementi |
| Manipolazione pixel per pixel | Manipolazione basata su oggetti |
| Richiede JavaScript per disegnare | Può essere creato con HTML/XML |
| Difficile da rendere accessibile | Più accessibile |

## Esempi pratici

### Esempio 1: Disegnare una scacchiera (come nel gioco della dama)

```javascript
function drawCheckerboard(ctx, boardSize, squareSize) {
    for (let row = 0; row < boardSize; row++) {
        for (let col = 0; col < boardSize; col++) {
            // Determina il colore della casella
            const isLightSquare = (row + col) % 2 === 0;
            ctx.fillStyle = isLightSquare ? '#EBECD0' : '#779556';
            
            // Disegna la casella
            ctx.fillRect(col * squareSize, row * squareSize, squareSize, squareSize);
        }
    }
}

const canvas = document.getElementById('myCanvas');
const ctx = canvas.getContext('2d');
const BOARD_SIZE = 8;
const SQUARE_SIZE = canvas.width / BOARD_SIZE;

drawCheckerboard(ctx, BOARD_SIZE, SQUARE_SIZE);
```

### Esempio 2: Disegnare un grafico a barre semplice

```javascript
function drawBarChart(ctx, data, width, height) {
    const barWidth = width / data.length;
    const maxValue = Math.max(...data);
    
    // Disegna gli assi
    ctx.beginPath();
    ctx.moveTo(0, height);
    ctx.lineTo(width, height);
    ctx.stroke();
    
    // Disegna le barre
    data.forEach((value, index) => {
        const barHeight = (value / maxValue) * height;
        const x = index * barWidth;
        const y = height - barHeight;
        
        ctx.fillStyle = 'steelblue';
        ctx.fillRect(x, y, barWidth - 2, barHeight);
    });
}

const data = [5, 10, 15, 20, 25, 30, 35, 40];
drawBarChart(ctx, data, canvas.width, canvas.height - 20);
```

## Conclusione

HTML Canvas è uno strumento potente per creare grafica dinamica sul web. Con la sua API di disegno flessibile, è possibile creare qualsiasi cosa, da semplici forme a complessi giochi 2D o visualizzazioni di dati. La comprensione del sistema di coordinate, del contesto di rendering e delle varie tecniche di disegno è fondamentale per sfruttare appieno le potenzialità di Canvas.

Per approfondire ulteriormente, è consigliabile esplorare la documentazione completa dell'API Canvas su [MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API) e sperimentare con esempi pratici per acquisire familiarità con le diverse tecniche di disegno e animazione.