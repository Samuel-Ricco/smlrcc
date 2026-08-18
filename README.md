# smlrcc

Sito personale di Samuel Ricco — [smlrcc.it](https://smlrcc.it)

Il sito sta in **un unico file** — [`index.html`](index.html) — con gli asset grafici in
[`src/`](src) e la copia di three.js in [`vendor/`](vendor). Niente build, niente
dipendenze da installare, niente bundler: basta che `src/` e `vendor/` restino accanto
al file.

## Com'è fatto

Per entrare nel sito c'è da fare buca.

**Il gate.** Si apre direttamente sul minigame, senza schermata di istruzioni. Una scena
low-poly in three.js: green, bunker, buca con bandierina. Il cursore *è* la mazza: la
testa segue il puntatore sul piano del green, e la pallina si colpisce passandoci sopra.
La potenza nasce dalla velocità reale del movimento, non da una barra da caricare.

- la velocità dello swing è misurata su una finestra fissa di 70 ms usando i timestamp
  dei pointer event, quindi non dipende dal framerate né dalla frequenza di polling del
  mouse;
- la collisione è un test su segmento (posizione precedente → attuale) per evitare il
  tunneling quando si muove veloce;
- la direzione la decide lo swing; la deviazione è proporzionale a quanto il colpo è
  preso di striscio, così centrare bene premia;
- rotolamento con attrito, rimbalzi, cattura in buca e *lip out* sopra i 6,2 m/s.

**Il filtro pixel art.** La scena non è renderizzata a risoluzione piena: il buffer è
alto circa 210 px e il canvas viene stirato a schermo intero con
`image-rendering: pixelated`, cioè nearest-neighbor. Antialias disattivato e ombre a
bordo netto, altrimenti il filtro arriverebbe sfumato e ingrandito. Costa meno di un
render pieno. Il cielo è un gradiente CSS a bande dietro al canvas trasparente.

**La transizione.** Quando la palla entra, la camera si tuffa dentro la buca fino a
riempire lo schermo di nero: quel nero diventa lo sfondo del sito. A quel punto il loop
3D va in pausa, così la home non paga il costo del rendering.

**La home.** Sfondo nero e una griglia di pixel che prende colore dove passa il cursore,
poi si spegne. La tipografia è pixel: Pixelify Sans per titoli e interfaccia, VT323 per
il testo.

**Le decorazioni della home sono 3D.** Sole, nuvole, uccellini, kart, bandiere e golfisti
non sono immagini ma modelli three.js, disegnati dallo stesso renderer del minigame e
quindi con la stessa pixelatura. Vivono in una scena a parte con camera ortografica
tarata in pixel CSS: ogni modello si aggancia al rettangolo dello sprite SVG che
sostituisce, che resta nel DOM invisibile a fare da ancora. Così scroll, resize e
breakpoint restano competenza del CSS, e senza WebGL gli SVG tornano visibili da soli.

## Sviluppo

three.js sta nel repo (`vendor/three.min.js`, r152), quindi il 3D funziona anche
offline; se quel file manca, la pagina ripiega da sola su unpkg e jsdelivr. L'unica
risorsa che resta esterna sono i due font di Google Fonts: senza rete il testo cade su
un monospace di sistema, tutto il resto funziona.

Meglio un server locale che il doppio click, così i percorsi relativi risolvono:

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

## Struttura

```
index.html      pagina intera: stile, markup, scena 3D, effetto pixel
src/            asset grafici in pixel art, un SVG per elemento
vendor/         three.min.js r152, copia locale della libreria 3D
CNAME           dominio per GitHub Pages
```

Dentro `index.html`:

| Blocco | Cosa contiene |
| --- | --- |
| `<style>` | palette, HUD del minigame, transizione, layout del sito |
| `#world` | canvas 3D + interfaccia del gate |
| `#site` | la home, con il canvas dell'effetto pixel |
| `<script>` | scena, fisica, transizione, effetto pixel, lingue |

Gli SVG in `src/` sono disegnati su griglia intera con `shape-rendering="crispEdges"`,
quindi si possono modificare a mano o in un editor pixel senza perdere i bordi netti.
Quelli animati (`golfer-*.svg`, `bird-*.svg`) portano l'animazione dentro al file: sono
usati come `<img>`, e il CSS della pagina non arriva dentro un SVG referenziato così.
