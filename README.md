# Visualizzatore DICOM (web)

Un visualizzatore di immagini mediche DICOM (TC, RM, RX…) che funziona interamente nel browser.

**Privacy:** i file dell'esame non vengono mai caricati su internet. Il browser li legge e li elabora direttamente sul computer di chi usa la pagina, e nessun server riceve le immagini. Le uniche cose conservate sono le note scritte dall'utente, nel browser del suo dispositivo.

## Documentazione
- 📘 [Manuale d'uso (PDF)](documenti/Manuale%20d'uso%20-%20Visualizzatore%20DICOM.pdf): istruzioni complete per computer e smartphone
- 🔒 [Privacy e dati sanitari (PDF)](documenti/Privacy%20e%20dati%20sanitari%20-%20Visualizzatore%20DICOM.pdf): perché i dati restano sul dispositivo e come verificarlo

I PDF sono anche nella finestra di aiuto (**?**) del visualizzatore.

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
- **Apri file**: puoi scegliere uno o più file DICOM oppure un archivio **.zip**. Con **Apri cartella** apri una cartella intera, oppure puoi trascinarla sull'area nera.
  Il programma salta da solo i file che non sono immagini, per esempio il visualizzatore incluso nei CD.
- **Demo**: genera un esame sintetico per fare prove senza dati reali: una TC di 60 fette e una RX del torace in due proiezioni (PA e laterale).
- Strumenti: Scorri, Finestra (trascina per cambiare livello/larghezza), Sposta, Zoom, Misura (distanza in mm), Nota, Incrocio (MPR).

### Viste multiple e riconoscimento dell'esame
Con **Vista automatica** il programma legge modalità, parte del corpo, orientamento e proiezione dai tag DICOM e sceglie la disposizione:

| Esame | Vista |
|---|---|
| TC o RM volumetrica (fette parallele ed equidistanti) | **MPR**: assiale, coronale e sagittale ricostruite, con linee di riferimento colorate e strumento **Incrocio** per portare i piani su un punto |
| Radiografie con più proiezioni | proiezioni affiancate: frontale (PA/AP), poi laterale e oblique; mammografia CC dx/sx e MLO dx/sx |
| RM con serie su piani diversi | griglia con una serie per piano |
| Altro (ecografie, filmati, immagini singole) | una vista |

Il menu delle viste permette anche di scegliere 1 immagine, 2 affiancate, griglia 2×2 o MPR. Ogni vista ha zoom e finestra propri; quella attiva ha il bordo blu. Ai bordi compaiono le lettere di orientamento (R/L, A/P, H/F).

### Note sulle immagini
Con lo strumento **Nota** si tocca un punto e si scrive il testo. Le note:
- compaiono sull'immagine come segnaposto numerato con il testo accanto; si modificano toccandole e si spostano trascinandole;
- sono elencate nel pannello a sinistra (tocca una nota per andare all'immagine);
- vengono salvate **nel browser del dispositivo** (`localStorage`), collegate all'identificativo dell'immagine (SOP Instance UID), e si ritrovano riaprendo lo stesso esame. Non vengono mai inviate in rete e non contengono il nome del paziente;
- si possono esportare e importare come file `.json`, come copia di sicurezza o per spostarle su un altro dispositivo.
- **▶**: riproduce la serie o il filmato (multi-frame, rispettando il Frame Time).
- Preset di finestra per la TC (cervello, tessuti molli, addome, polmone, osso), Inverti, Salva PNG.
- **Tag DICOM**: mostra tutti i tag del file corrente, con un filtro di ricerca.

### Da computer
Rotellina: scorri le fette · Ctrl+rotellina: zoom · tasto destro: sposta · doppio clic: adatta.

### Da smartphone e tablet
- Gli strumenti sono nella barra in basso. Lo strumento predefinito è **Scorri**: trascina un dito su e giù per cambiare immagine.
- Due dita: allarga/stringi per lo zoom, trascina per spostare. Doppio tocco: adatta allo schermo.
- **☰** apre l'elenco delle serie. Il pannello dei tag sale dal basso.
- Sui telefoni di solito non si possono scegliere cartelle: usa **Apri** con i singoli file o con lo **.zip** dell'esame.
- Per non esaurire la memoria del telefono, le immagini compresse lontane da quella visualizzata vengono liberate e ridecodificate quando servono.

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
- L'MPR è a ricostruzione semplice (pixel più vicino), senza MIP, viste oblique o 3D. Sui telefoni, se la memoria non basta, il volume viene ricostruito a risoluzione ridotta.
- Mancano la misura degli angoli e le ROI con media e deviazione standard.

È uno strumento didattico: non va usato per fare diagnosi.

## Struttura del codice (`index.html`)
1. `parseFile` / `buildImages`: lettura del file e dei tag principali
2. `sortSeries`: ordinamento delle fette
3. `decodeImage` (con `decodeRLE`, `decodeJPEG`, `decodeLossless`, `decodeWasm`) e `renderOffscreen`: decodifica dei pixel e applicazione della finestra (window/level)
4. `planFor`, `projectionView`, `volumeInfo`: riconoscimento del tipo di esame, delle proiezioni e dei volumi
5. `Pane`, `setLayout`: le viste sullo schermo e la loro disposizione
6. `buildVolume`, `extractPlane`: volume 3D e ricostruzioni MPR
7. `drawPane`, `updateOverlays`, `drawRefLines`: disegno, overlay, linee di riferimento e misure
8. `noteRef`, `drawNotes`, `exportNotes`: note salvate nel browser
9. `makeDemoFiles`: crea veri file DICOM sintetici (TC e RX), utile per capire il formato
