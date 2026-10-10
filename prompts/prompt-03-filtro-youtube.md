# Prompt 03 — Filtro YouTube: solo video sull'AI, scelti da un modello decisionale

**Modello da usare: [SCRIVI QUI UNO DEI TRE: OpenAI Decisions, Jev, d1]**

Voglio un'estensione per Chrome (Manifest V3) che ripulisce YouTube lasciando visibili soltanto i video che parlano di intelligenza artificiale. A decidere è il modello indicato qui sopra, e soltanto quello: l'estensione non deve contenere codice per gli altri. La uso anche per misurare quel modello, quindi a ogni decisione devo sapere quanto ci ha messo e quanto è costata.

La installo io caricandola in Chrome come estensione non pacchettizzata: da quel momento deve funzionare da sola mentre scorro YouTube.

## Comportamento sulla pagina

Sulla home e sui risultati di ricerca di youtube.com, per ogni scheda video che entra nello schermo:

1. Leggi titolo, nome del canale, identificativo del video e indirizzo della miniatura. Giudica i video normali e gli Shorts; ignora annunci, playlist, mix e schede dei canali, che restano visibili. Se il nome del canale non compare sulla scheda, il canale resta vuoto.
2. Poni al modello una sola domanda sì/no e leggi la probabilità del "sì".
3. Se la probabilità è sotto la soglia, rimpiazza la scheda con una riga compatta e attenuata che riporta il titolo accorciato e la probabilità ottenuta, con un pulsante "Mostra" che la ripristina. Se è sopra la soglia non toccare nulla.

La domanda va lasciata in inglese, esattamente così:

> Is artificial intelligence the main topic of this YouTube video? Count AI models, AI tools and assistants, AI news, tutorials and automations built with AI. Answer no when AI is only mentioned in passing or used as a buzzword.

Il testo da valutare ha sempre questa forma: `Title: <titolo>` e, a capo, `Channel: <canale>`.

## Scheda dei modelli

Usa solo la riga del modello indicato in cima.

| Modello | Chiamata | Come si scrive la domanda | Dove sta la risposta | Immagini | Prezzo input |
|---|---|---|---|---|---|
| OpenAI Decisions | `POST https://api.openai.com/v1/decisions`, `model: "gpt-6-luna"`, testo in `input` | `questions` è una lista: `{type: "predicate", name, instructions}` | `answers[0].probability` | sì: messaggio utente con una parte `input_text` e una `input_image` | $0.10 per milione di token |
| Jev (TypeSafe) | `POST https://api.typesafe.ai/v1/systemone`, `model: "jev-latest"`, testo in `state` | `questions` è un oggetto: `{nome: {type: "noul", instructions}}` | `answers.<nome>.noul` | no, solo testo | $0.042 per milione di token |
| d1 (Liquid AI) | `POST https://api.liquid.ai/decisions/v1/systemone`, `model: "d1"`, testo in `state` | come Jev | `answers.<nome>.noul` | sì: elenco `images` accanto a `state` | $0.04 per milione di token |

Autenticazione `Authorization: Bearer <chiave>`; ogni risposta riporta `usage.input_tokens`. Sono API recenti: prima di scrivere la chiamata controlla la documentazione del tuo modello.

- OpenAI Decisions: https://developers.openai.com/api/docs/guides/decisions.md
- Jev (TypeSafe): https://docs.typesafe.ai/api.md
- d1 (Liquid AI): https://docs.liquid.ai/lfm/models/d1.md

## Due modi di giudicare

- **Solo testo**: titolo e canale. È il modo predefinito.
- **Testo + miniatura**: aggiungi l'immagine di copertina. Esiste solo se il modello legge le immagini; in caso contrario l'opzione non compare e una nota nel pannello lo spiega.

Le API non accettano indirizzi di immagini: scarica la miniatura in formato medio (320×180) e inviala come data URL in base64.

## Pannello di controllo

Un riquadro fisso in basso a destra, richiudibile, con:

- il nome del modello;
- l'interruttore solo testo / testo + miniatura, se disponibile;
- la soglia regolabile, predefinita a 0.5;
- video giudicati, video nascosti, tempo mediano di risposta, spesa accumulata, errori.

## Registro

Salva ogni chiamata con queste colonne, in quest'ordine: istante, identificativo del video, titolo, canale, modello, modalità, probabilità, esito (mostrato o nascosto), millisecondi di risposta, millisecondi per scaricare e codificare la miniatura, token in input, costo in dollari.

Nel CSV l'intestazione è esattamente `istante,id_video,titolo,canale,modello,modalita,probabilita,esito,ms_risposta,ms_miniatura,token_input,costo_usd`, con virgola come separatore e punto decimale.

- Il tempo di risposta misura solo la chiamata al modello; la preparazione della miniatura ha la sua colonna.
- Il costo è `input_tokens × prezzo`; se la risposta contiene già `usage.cost`, usa quello. Il prezzo sta in un unico punto del codice.

Nel pannello servono un pulsante che scarica il registro in CSV, con il nome del modello nel nome del file, e uno che lo azzera.

## Regole che non si toccano

- L'estensione legge la pagina e nasconde schede, nient'altro. Niente clic, like, iscrizioni, commenti o navigazione al posto mio.
- La chiave API non compare mai nel codice né nel registro. Si inserisce nella pagina delle opzioni e resta in `chrome.storage.local`. In alternativa, se nella cartella dell'estensione esiste un file `chiave.local.json`, leggila da lì all'avvio: quel file è personale e non va mai condiviso.
- Le chiamate al modello partono dal service worker, non dalla pagina di YouTube.
- Ogni video viene giudicato una volta sola per modalità: conserva i risultati e riusali quando la scheda ricompare.
- Limita le chiamate contemporanee a 4.
- Se una chiamata fallisce, il video resta visibile e l'errore viene contato nel pannello.
- Il nome dell'estensione contiene il nome del modello, così posso tenerne installate più versioni e attivarne una per volta.

## Prima di consegnare

Caricala in un Chrome di prova, apri YouTube e verifica che le schede vengano davvero nascoste e che il CSV contenga una riga per chiamata. Dimmi cosa hai controllato e cosa no.
