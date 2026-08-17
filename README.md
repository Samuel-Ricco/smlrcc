# sml.rcc

Sito personale di Samuel Ricco — [smlrcc.it](https://smlrcc.it)

Tutto il sito sta in **un unico file**: [`index.html`](index.html). Niente build, niente
dipendenze da installare, niente bundler. Si apre con un doppio click.

## Com'è fatto

Per entrare nel sito c'è da fare buca.

**Il gate.** Una scena low-poly in three.js: green, bunker, buca con bandierina. Il
cursore *è* la mazza: la testa segue il puntatore sul piano del green, e la pallina si
colpisce passandoci sopra. La potenza nasce dalla velocità reale del movimento, non da
una barra da caricare.

- la velocità dello swing è misurata su una finestra fissa di 70 ms usando i timestamp
  dei pointer event, quindi non dipende dal framerate né dalla frequenza di polling del
  mouse;
- la collisione è un test su segmento (posizione precedente → attuale) per evitare il
  tunneling quando si muove veloce;
- la direzione la decide lo swing; la deviazione è proporzionale a quanto il colpo è
  preso di striscio, così centrare bene premia;
- rotolamento con attrito, rimbalzi, cattura in buca e *lip out* sopra i 6,2 m/s.

**La transizione.** Quando la palla entra, la camera si tuffa dentro la buca fino a
riempire lo schermo di nero: quel nero diventa lo sfondo del sito. A quel punto il loop
3D va in pausa, così la home non paga il costo del rendering.

**La home.** Sfondo nero e una griglia di pixel che prende colore dove passa il cursore,
poi si spegne. La palette è la stessa della scena 3D (verdi, ocra, crema).

## Sviluppo

Basta aprire `index.html` nel browser. Serve una connessione: three.js arriva da CDN
(con due mirror di fallback).

Se preferisci un server locale:

```bash
python -m http.server 8000
```

## Deploy

GitHub Pages serve il file dalla radice del repo.

1. Settings → Pages → Source: `Deploy from a branch`, branch `main`, cartella `/ (root)`.
2. Custom domain: `smlrcc.it` (crea il file `CNAME` con dentro il dominio).
3. DNS del dominio:
   - apex `smlrcc.it` → record **A** verso `185.199.108.153`, `185.199.109.153`,
     `185.199.110.153`, `185.199.111.153`;
   - `www` → record **CNAME** verso `samuel-ricco.github.io`.
4. Spuntare *Enforce HTTPS* quando il certificato è stato emesso.

## Struttura del file

| Blocco | Cosa contiene |
| --- | --- |
| `<style>` | palette, HUD del minigame, transizione, layout del sito |
| `#world` | canvas 3D + interfaccia del gate |
| `#site` | la home, con il canvas dell'effetto pixel |
| `<script>` | scena, fisica, transizione, effetto pixel |
