# Prompt 01 — Wikipedia Race: da "Gatto" a "Bitcoin" con un modello decisionale

**Modello da usare: [SCRIVI QUI UNO DEI TRE: OpenAI Decisions, Jev, d1]**

Costruiscimi una piccola applicazione web locale in cui il modello indicato qui sopra gioca a questo gioco: partire da una voce di Wikipedia in italiano e arrivare a un'altra seguendo solo i link interni. Partenza predefinita **Gatto**, arrivo **Bitcoin**; entrambe modificabili da due campi di testo. Voglio sapere se arriva, in quanti passaggi, in quanto tempo e a che prezzo.

L'applicazione usa solo quel modello e non contiene codice per gli altri. Ne costruirò una copia identica per ciascun modello e confronterò i risultati dopo: per questo regole, formati e misure qui sotto vanno seguiti alla lettera.

## Le regole del gioco

A ogni turno:

1. Recupera i link della voce corrente verso altre voci dell'enciclopedia (it.wikipedia.org, solo voci vere: niente categorie, file, pagine di aiuto o voci inesistenti). Risolvi i reindirizzamenti, sia per partenza e arrivo sia per ogni link: "Gatto", per esempio, porta a "Felis silvestris catus". Ogni link compare con il titolo vero della voce a cui porta, senza doppioni e senza la voce corrente.
2. Togli le voci già visitate, così il modello non gira in tondo, e metti i titoli rimasti in ordine alfabetico secondo le regole dell'italiano (`Intl.Collator('it')`).
3. Se tra i link c'è la voce di arrivo, vai lì direttamente senza interpellare il modello: la corsa è finita.
4. Se resta un solo link, seguilo senza interpellare il modello. Se non ne resta nessuno, la corsa finisce come "non arrivato".
5. Altrimenti i titoli diventano le opzioni di una domanda a scelta e il modello ne indica una. Si segue quella.

La corsa termina all'arrivo oppure dopo 25 passaggi, nel qual caso l'esito è "non arrivato".

La domanda resta in inglese, esattamente così:

> Which of these Wikipedia articles is the best next click to reach the target article in as few clicks as possible?

Il contesto passato al modello contiene soltanto due righe: `Current article: <voce corrente>` e `Target article: <voce di arrivo>`. Le opzioni sono i soli titoli, senza descrizioni.

### Voci con più di 255 link

Il modello accetta al massimo 255 opzioni per domanda, e molte voci ne hanno di più (quella sul gatto quasi 500). In quel caso:

- dividi i link, già in ordine alfabetico, in gruppi consecutivi da 255 al massimo e invia tutti i gruppi in un'unica richiesta, una domanda per gruppo;
- prendi il vincitore di ciascun gruppo e fai una seconda richiesta con i soli vincitori come opzioni.

Tempo e costo delle due richieste si sommano e contano come un'unica decisione.

## Scheda dei modelli

Usa solo la riga del modello indicato in cima.

| Modello | Chiamata | Formato della domanda a scelta | Risposta | Prezzo input |
|---|---|---|---|---|
| OpenAI Decisions | `POST https://api.openai.com/v1/decisions`, `model: "gpt-6-luna"`, contesto in `input` | `questions` è una lista: `{type: "choice", name, instructions, choices: [{value}]}` | `answers[i].choice`, `answers[i].confidence` | $0.10 per milione di token |
| Jev (TypeSafe) | `POST https://api.typesafe.ai/v1/systemone`, `model: "jev-latest"`, contesto in `state` | `questions` è un oggetto: `{nome: {type: "choice", instructions, criteria: {titolo: ""}}}` | `answers.<nome>.choice`, `answers.<nome>.confidence` | $0.042 per milione di token |
| d1 (Liquid AI) | `POST https://api.liquid.ai/decisions/v1/systemone`, `model: "d1"`, contesto in `state` | come Jev | come Jev | $0.04 per milione di token |

Autenticazione `Authorization: Bearer <chiave>`; ogni risposta riporta `usage.input_tokens`. Sono API recenti: prima di scrivere la chiamata controlla la documentazione del tuo modello.

- OpenAI Decisions: https://developers.openai.com/api/docs/guides/decisions.md
- Jev (TypeSafe): https://docs.typesafe.ai/api.md
- d1 (Liquid AI): https://docs.liquid.ai/lfm/models/d1.md

## Cosa voglio vedere

Una pagina con il nome del modello in grande, i due campi partenza e arrivo e un pulsante "Via". Durante la corsa, in tempo reale:

- il percorso che si allunga, una voce per riga, ciascuna cliccabile verso Wikipedia;
- accanto a ogni passaggio: millisecondi impiegati dal modello, numero di opzioni tra cui ha scelto, confidenza dichiarata;
- in testa: passaggi fatti, tempo totale di decisione, costo totale, stato (in corsa, arrivato, non arrivato, errore).

La pagina deve restare leggibile anche in una finestra stretta, perché affiancherò sullo schermo le tre versioni. La porta su cui gira l'applicazione è impostabile, così le tre versioni possono restare aperte insieme.

## Misure

- Il tempo di una decisione è solo l'attesa della risposta del modello, dall'invio della richiesta alla risposta completa. Il tempo speso a interrogare Wikipedia va tenuto fuori e mostrato a parte, e da questo vanno tenute separate le pause imposte da Wikipedia quando risponde che stai andando troppo veloce.
- Costo = `input_tokens × prezzo`; se la risposta contiene `usage.cost`, usa quel valore. Il prezzo sta in un solo punto del codice.
- Salva su disco i link di ogni voce la prima volta che li scarichi, in una cartella `cache_wikipedia` dentro l'applicazione (percorso impostabile con `CACHE_WIKIPEDIA`), e nelle corse successive leggili da lì senza contattare Wikipedia. Lo stesso vale per i titoli risolti di partenza e arrivo. Un'opzione all'avvio (`--svuota-cache`) la cancella.
- A fine corsa salva nella cartella `risultati` un file CSV chiamato `<modello>_<partenza>_<arrivo>_<AAAAMMGG-HHMMSS>.csv`, con virgola come separatore, punto decimale e questa riga d'intestazione, esattamente:

  `modello,passaggio,da,a,opzioni,richieste,ms_decisione,ms_wikipedia,ms_pause_wikipedia,token_input,costo_usd,confidenza,esito`

  Una riga per passaggio, compresi quelli fatti senza interpellare il modello (0 richieste, 0 millisecondi di decisione, 0 token, confidenza vuota). La colonna `esito` resta vuota tranne nell'ultima riga, quella dei totali: lì `passaggio` vale `TOTALE`, `da` è la voce di partenza, `a` l'ultima voce raggiunta, le colonne numeriche sono le somme, `confidenza` è vuota ed `esito` vale `arrivato`, `non arrivato` oppure `errore`.

## Vincoli

- Scrivila in Node.js, con meno dipendenze possibili.
- La chiave API si legge da un file `.env` (`OPENAI_API_KEY`, `TYPESAFE_API_KEY` o `LIQUID_API_KEY`, secondo il modello) e resta sul lato server: non deve mai arrivare al browser né finire nei file dei risultati.
- Interroga Wikipedia tramite la sua API pubblica, senza raffiche di richieste, presentandoti con uno User-Agent che identifichi il progetto. Wikipedia chiede anche un recapito: se nel file `.env` c'è `WIKIPEDIA_CONTATTO` (un'email o un sito), mettilo nello User-Agent. Chi usa questo prompt deve scriverci il proprio.
- Se la chiave manca, la pagina lo dice chiaramente invece di partire.

## Prima di consegnare

Avvia l'applicazione, fai correre davvero Gatto → Bitcoin e controlla che il percorso sia fatto di link che esistono sul serio nelle voci attraversate. Riportami com'è andata la corsa e che cosa non sei riuscito a verificare.
