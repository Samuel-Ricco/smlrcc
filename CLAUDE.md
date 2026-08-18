# smlrcc — note di progetto

Portfolio personale di Samuel Ricco. **Un solo file** (`index.html`, ~2300 righe) più gli
asset grafici in `src/`. Niente build, niente bundler, niente dipendenze da installare.
Online su **smlrcc.it** via GitHub Pages (branch `main`, cartella root).

```
index.html   stile, markup, scena 3D del gioco, decorazioni 3D della home, i18n
src/         32 SVG in pixel art (sprite, icone, cursori, bandiere)
CNAME        smlrcc.it
README.md    documentazione per chi apre il repo
```

## Modo di lavorare

- **Push a ogni step completato**, senza chiedere: commit descrittivo e `git push` su
  `github.com/Samuel-Ricco/smlrcc`. Autorizzazione permanente data dall'utente.
- Messaggi di commit in inglese, discorsivi: cosa è cambiato e **perché**, comprese le
  cause dei bug corretti.
- L'utente scrive in italiano. Commenti nel codice e testi del sito in italiano.
- **Verificare sempre su un server locale**, non aprendo il file: il pannello di anteprima
  serve la pagina come `data:` URL, dove i percorsi relativi a `src/` non risolvono e
  *tutte* le immagini risultano rotte. Basta `python -m http.server 8123` in un
  `.claude/launch.json` temporaneo (da cancellare prima del commit).

### Trappole dell'ambiente di anteprima

Non sono bug del sito, ma falsano ogni misura:

- il pannello a volte mostra uno **snapshot vecchio** mentre il JS gira su un documento
  nuovo: se i numeri non tornano, controllare prima di credere a un test;
- con la pagina non visibile **`requestAnimationFrame` è sospeso** (zero frame) e i
  `setTimeout` sono limitati a ~850 ms: uno swing sintetico veloce è impossibile;
- `innerWidth`/`innerHeight` possono valere **0** se il pannello non è disposto.

## Le due metà

**1. Gate (minigame).** Si apre direttamente sul gioco, senza schermata di istruzioni. Il
cursore *è* la mazza. Mandando la palla in buca la camera si tuffa nel foro finché il nero
riempie lo schermo — quel nero (`--void`) è lo stesso sfondo del sito, così la transizione
non ha stacco.

**2. Home.** Sfondo nero, tipografia pixel, decorazioni **3D vive** disegnate dallo stesso
renderer, griglia di pixel che prende colore sotto al cursore.

Fasi in `state.phase`: `play` → `sunk` → `site`. Il loop di rendering non si ferma mai:
entrando nel sito cambia scena (`updateSite`) invece di andare in pausa.

## Fisica del colpo (non toccare senza motivo)

Ogni costante è stata tarata e verificata a schermo:

| costante | valore | ruolo |
|---|---|---|
| `SWING_WIN` | 70 ms | finestra su cui si misura la velocità dello swing |
| `POWER_K` | 0.42 | velocità mazza → velocità pallina |
| `POW_MAX` | 38 | fondo scala della barra |
| `FRICTION` | 4.2 | decelerazione di rotolamento |
| `HIT_R` | 0.55 | raggio di contatto (mezza testa + pallina) |
| `CLUB_Y` | 0.28 | quota del centro testa |
| `CAPTURE_V` | 6.2 | oltre questa velocità la palla gira sul bordo |
| `DIVE_T` | 1.7 s | durata del tuffo nella buca |

Scelte importanti, tutte nate da bug reali:

- **La velocità si misura sui timestamp dei pointer event**, su finestra fissa di 70 ms,
  non sui frame: misurandola per frame, con event-rate diverso dal framerate la potenza
  risultava sottostimata.
- **Collisione su segmento** (posizione precedente → attuale): a mano veloce la testa
  supera la pallina in un frame e senza questo la attraversa.
- **`off` è la distanza *laterale* dalla linea di swing**, non quella di avvicinamento:
  con la seconda anche un colpo perfettamente centrato perdeva potenza.
- **Guardia sui salti del puntatore**: uno spostamento oltre 2.6 unità in un evento azzera
  la finestra e blocca il colpo per 3 frame. Senza, rientrando col mouse nella finestra la
  palla partiva a razzo.
- **`#world` ha `touch-action:none`**: senza, il browser legge il trascinamento come scroll,
  annulla i pointer event e su mobile la mazza smette di seguire il dito.

## Filtro pixel art

La scena non è renderizzata a risoluzione piena: il buffer è alto `PIX_H` px e il canvas
viene stirato con `image-rendering: pixelated`. `PIX_H_GAME = 300`, `PIX_H_SITE = 520` (le
decorazioni della home sono piccole e a 300 diventavano poltiglia). Antialias **off** e
`BasicShadowMap`, altrimenti il filtro ingrandisce bordi già sfumati. Cambiando modalità
serve `curW = 0` per forzare il ridimensionamento.

## Decorazioni 3D della home

Camera **ortografica tarata in pixel CSS**: ogni modello legge il `getBoundingClientRect()`
dello sprite SVG che sostituisce, e quello sprite resta nel DOM invisibile (`body.d3`) a
fare da ancora. Così scroll, resize e breakpoint restano competenza del CSS, e senza WebGL
gli SVG piatti tornano visibili da soli (`buildSite` è in `try/catch`).

Tabella `SITE_DECOS`: `[selettore, costruttore, altezza, poggiaAterra]`.
Altezza `0` significa "riempi esattamente il rettangolo", misurata dal bounding box; un
valore esplicito serve a chi deve essere **più grande** del proprio ancoraggio (pianeta,
nuvole, uccelli).

Cose imparate a caro prezzo:

- **Sotto proiezione ortografica inclinare un piano non serve a niente**: senza prospettiva
  si proietta identico a uno piatto e legge come un muro. I prati sono **trapezi** con
  strisce che rastremano e colore che si scurisce in fondo (`k = 0.72`).
- **Niente ombre proiettate**: su un piano frontale leggono come ombra sul muro. C'è solo
  l'**ellisse di contatto** sotto ogni modello, dimensionata sull'impronta reale
  (`userData.foot`: la buca per le bandiere, le ruote per il kart) e non sull'ingombro.
- L'ellisse **non deve ereditare la rotazione** del modello (le si applica il quaternione
  inverso ogni frame) né **saltare** con lui durante l'animazione da clic.
- I modelli vanno **ricentrati sul bounding box**: un golfista è asimmetrico per mazza e
  scarpe e usciva dal suo rettangolo.
- Il trapezio si restringe in alto: un oggetto troppo in alto o troppo ai bordi **resta
  fuori dal prato** e sembra fluttuare.

**Animazioni**: il cielo ha idle (pianeta che ruota, uccelli che battono le ali e tornano
indietro a fine corsa, nuvole alla deriva); tutto ciò che poggia sull'erba è **fermo** e
reagisce al **clic** con un salto e un giro a otto scatti. Il colpo si rileva sui rettangoli
delle ancore, non con un raycast.

## Effetto pixel del cursore

Griglia da 22 px su canvas 2D **dietro** al canvas 3D (`#px` z0, `#scene` z1, contenuti z2).
Il colore avanza col movimento su un'**onda triangolare** (150→350→150): con un modulo, a
fine giro saltava da magenta a menta. Premendo parte un'onda concentrica che **spazza via**
il colore dietro al fronte, come la scia di un sasso in acqua.

Su **touch la scia è disattivata** (`pointerType === 'touch'`): i pointermove arrivano solo
a dito premuto, quindi tra due tap l'interpolazione tirava una riga di colore da un punto
all'altro. Sul touch resta solo lo splash.

Il suono dello splash ha doppia protezione dallo spam: **niente sotto i 70 ms** e **massimo
5 voci sovrapposte**, contatore liberato a fine oscillatore.

## Estetica (vincoli fissi)

- Palette **vaporwave**: `--void #120a1f`, `--fg #f0e6ff`, `--pink #ff5f9e`,
  `--cyan #49f2e0`, `--violet #b98cf7`, `--mint #7cf0a8`. Il verde del gioco è turchese,
  i bunker magenta, gli alberi viola.
- Stile **pixel/8-bit**: spigoli vivi, bordi pieni da 2px, ombre solide sfalsate,
  transizioni a `steps()`. **Niente** raggi di curvatura, blur o gradienti sfumati.
- Font: **Pixelify Sans** per titoli e UI, **VT323** per il testo. Silkscreen è stato
  scartato perché M e W erano illeggibili. Gli SVG usano `shape-rendering="crispEdges"` e
  vanno scalati per **fattori interi**, altrimenti i pixel si sfocano.
- Il ciano è il colore degli elementi interattivi (header, bottoni, hover), il rosa quello
  decorativo (etichette, bandiere, punti dei titoli).
- I titoli grandi finiscono con una **pallina da golf** (`<i class="gball">`) al posto del punto.

## Lingue

Dizionario `I18N` con chiavi `data-i18n`, IT/EN, selettore in alto a destra in **entrambe**
le metà, scelta salvata in `localStorage`, lingua iniziale dedotta dal browser. Vive **fuori
da `init()`**, così funziona anche se three.js non carica.

Attenzione: le stringhe finiscono in `innerHTML` e contengono apostrofi. Una volta
`all'estetica` è finito non escapato in una stringa a singoli apici e ha rotto l'intero
script. Usare doppi apici quando il testo contiene apostrofi.

## Stato attuale

Da riempire con contenuti veri:

- le **tre card dei progetti** (titolo, due righe, tag) sono segnaposto;
- **Codice** e **Altrove** nei contatti (`github.com/Samuel-Ricco` e un LinkedIn troncato);
- l'email `ciao@smlrcc.it` è un segnaposto da confermare.

Deploy: HTTPS attivo. Il DNS ha **un solo record A** dei quattro di GitHub — gli altri tre
(`185.199.109/110/111.153`) restano da aggiungere per la ridondanza.
