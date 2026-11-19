# Visualizzatore DICOM (web)

Un visualizzatore di immagini mediche DICOM (TC, RM, RX…) che funziona interamente nel browser.

**Privacy:** i file dell'esame non vengono mai caricati su internet. Il browser li legge e li elabora direttamente sul computer di chi usa la pagina, e nessun server riceve le immagini.

## Come si apre
- **Online**: con GitHub Pages attivo, all'indirizzo `https://<utente>.github.io/<nome-repository>/`.
- **In locale**: doppio clic su `index.html` (Chrome, Edge o Firefox).

In entrambi i casi serve internet per scaricare le librerie: `dicom-parser`, più i decodificatori delle immagini compresse quando servono.

## Pubblicare su GitHub Pages
1. Carica questi file in un repository GitHub.
2. Vai in *Settings → Pages*, alla voce *Source* scegli *Deploy from a branch*, poi il branch `main` e la cartella `/ (root)`.
3. Dopo circa un minuto il sito è online all'indirizzo indicato nella stessa pagina.

Non caricare mai nel repository file di esami reali. Il `.gitignore` esclude già i file `.dcm` e `DICOMDIR`.

## Uso
- **Apri cartella** / **Apri file**, oppure trascina una cartella sull'area nera.
- **Demo**: genera una TC sintetica di 60 fette per fare prove senza dati reali.
- Rotellina: scorri le fette · Ctrl+rotellina: zoom · tasto destro: sposta · doppio clic: adatta.
- Strumenti: Finestra (trascina per cambiare livello/larghezza), Sposta, Zoom, Misura (distanza in mm).
- Preset di finestra per la TC (cervello, tessuti molli, addome, polmone, osso), Inverti, Salva PNG.
- **Tag DICOM**: mostra tutti i tag del file corrente, con un filtro di ricerca.

## Cosa supporta
- DICOM non compressi: Implicit/Explicit VR Little Endian, Explicit Big Endian.
- DICOM compressi:

  | Formato | Decodificatore |
  |---|---|
  | JPEG Baseline 8 bit | il browser stesso |
  | JPEG Lossless (Process 14, SV1) | `jpeg-lossless-decoder-js` |
  | JPEG-LS (lossless e near-lossless) | CharLS in WebAssembly (`@cornerstonejs/codec-charls`) |
  | JPEG 2000 / HTJ2K | OpenJPEG in WebAssembly (`@cornerstonejs/codec-openjpeg`) |
  | RLE Lossless | codice incluso nel file |

  I decodificatori vengono scaricati da jsdelivr solo la prima volta che servono.
- 8/16/32 bit, signed/unsigned, Rescale Slope/Intercept (valori in HU per la TC), MONOCHROME1/2, RGB/YBR, multi-frame.
- Le fette vengono ordinate in base alla posizione nel paziente (Image Position/Orientation), oppure per Instance Number.

Il programma è stato verificato con i file di test di pydicom e di jpeg-lossless-decoder-js. Le versioni RLE, JPEG 2000 e JPEG-LS lossless di `MR_small` danno pixel identici all'originale non compresso.

## Limiti e prossimi passi possibili
- Non sono supportati il JPEG Extended a 12 bit (`1.2.840.10008.1.2.4.51` a 12 bit) e il Deflate: per questi file compare un messaggio.
- Mancano le viste MPR (sagittale/coronale), la misura degli angoli e le ROI con media e deviazione standard.

È uno strumento didattico: non va usato per fare diagnosi.

## Struttura del codice (`index.html`)
1. `parseFile` / `buildImages`: lettura del file e dei tag principali
2. `sortSeries`: ordinamento delle fette
3. `decodeImage` (con `decodeRLE`, `decodeJPEG`, `decodeLossless`, `decodeWasm`) e `renderOffscreen`: decodifica dei pixel e applicazione della finestra (window/level)
4. `draw`, `updateOverlays`: disegno, overlay e misure
5. `makeDemoFiles`: crea veri file DICOM sintetici, utile per capire il formato
