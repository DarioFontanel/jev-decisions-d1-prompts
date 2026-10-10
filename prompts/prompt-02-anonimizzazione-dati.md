# Prompt 02 — Anonimizzazione: trovare e oscurare dati personali in un foglio di calcolo e in un video

**Modello da usare: [SCRIVI QUI UNO DEI TRE: OpenAI Decisions, Jev, d1]**

Mi serve uno strumento da riga di comando che cerca informazioni private dentro due tipi di file e me ne restituisce una copia ripulita. A giudicare è il modello indicato qui sopra, e soltanto quello: lo strumento non contiene codice per gli altri.

- **Foglio di calcolo** (CSV o XLSX), letto come testo. Funziona con tutti e tre i modelli.
- **Registrazione dello schermo** (MP4), giudicata fotogramma per fotogramma. Funziona solo se il modello legge le immagini; in caso contrario lo strumento lo dice e si ferma lì.

Oltre al file ripulito voglio i numeri: quanto tempo ha impiegato il modello e quanto è costato. Costruirò una copia identica dello strumento per ciascun modello e confronterò i risultati dopo: per questo regole, formati e misure qui sotto vanno seguiti alla lettera.

## Le categorie

Sono le stesse per fogli e video. Il valore `none` significa "niente di privato".

| Valore | Cosa copre | Descrizione da passare al modello |
|---|---|---|
| `person_name` | Nome e cognome di una persona | `First and last name of a person` |
| `email` | Indirizzo email personale | `Personal email address` |
| `phone` | Numero di telefono | `Phone number` |
| `home_address` | Indirizzo di abitazione | `Home street address of a person; a city name alone does not count` |
| `id_number` | Codice fiscale o numero di documento | `Tax code or ID document number` |
| `bank_or_card` | IBAN o numero di carta | `IBAN or payment card number` |
| `credential` | Password, chiave API, token | `Password, API key or access token` |
| `private_message` | Messaggio di una conversazione privata | `Text of a private conversation between people` |
| `none` | Nessuna informazione privata | `No private information` |

Passa al modello i valori e le descrizioni in inglese della terza colonna, esattamente come sono scritte e in quest'ordine.

## Parte 1: il foglio di calcolo

Una richiesta per riga. Il contesto è la riga stessa come oggetto JSON, con i nomi delle colonne come chiavi. Dentro la richiesta, una domanda a scelta per ogni colonna, tutte insieme:

> What kind of private information, if any, does the field "<nome colonna>" contain?

con le categorie qui sopra come opzioni. Una cella è da oscurare quando la scelta è diversa da `none`. Se le colonne sono più di 20, dividi la riga in più richieste da 20 domande al massimo.

Produci:

- una copia del foglio in cui ogni cella da oscurare è sostituita dalla sua categoria tra parentesi quadre, per esempio `[email]`;
- un file `celle_oscurate_<modello>.csv` con intestazione `riga,colonna,categoria,confidenza`, una riga per cella oscurata; `riga` è il numero della riga di dati partendo da 1 (l'intestazione del foglio non conta).

Se insieme al foglio ricevi un file di soluzioni, calcola precisione e richiamo ed elenca le celle sbagliate. Il file è un CSV con intestazione `riga,colonna,categoria`: `riga` è il numero della riga di dati partendo da 1 (l'intestazione del foglio non conta), `colonna` è il nome della colonna, `categoria` è uno dei valori della tabella. Elenca solo le celle private: tutto ciò che non compare vale `none`.

## Parte 2: la registrazione dello schermo

1. **Estrai i fotogrammi** con ffmpeg, uno ogni mezzo secondo, ridimensionati a 1280 pixel di larghezza.
2. **Primo passaggio, fotogramma intero.** Una richiesta per fotogramma, con una domanda sì/no:
   > Does this screenshot show at least one of the following: a person's full name, an email address, a phone number, a home address, an ID number, a bank or card number, a credential, or the text of a private conversation?

   Il testo che accompagna l'immagine è `Screenshot from a screen recording.` Il fotogramma è segnalato quando la probabilità supera 0.5 (soglia modificabile).
3. **Secondo passaggio, solo sui fotogrammi segnalati.** Dividi il fotogramma in una griglia di 4 colonne per 3 righe e ritaglia i 12 riquadri. Per ogni riquadro fai una richiesta separata, con una sola immagine, il testo di accompagnamento `Part of a screenshot from a screen recording.` e una sola domanda a scelta con le categorie della tabella, `none` compresa:
   > What kind of private information, if any, is visible in this image?

   Così sai dove si trova il dato, e ne puoi trovare più d'uno nello stesso fotogramma.
4. **Costruisci gli intervalli.** Per ogni riquadro unisci i fotogrammi consecutivi in cui risulta diverso da `none`, e allarga ogni intervallo di mezzo secondo prima e dopo.
5. **Oscura** con ffmpeg ogni riquadro nel suo intervallo, allargandolo di un piccolo margine: sfocatura gaussiana abbastanza forte da rendere il testo illeggibile, senza rettangoli pieni. L'audio resta intatto. La griglia è definita in frazioni del fotogramma: i fotogrammi si analizzano rimpiccioliti, ma la sfocatura si applica al video alla sua risoluzione originale, quindi riporta i riquadri a quella scala.

Produci il video oscurato e due file:

- `fotogrammi_<modello>.csv` con intestazione `tempo,probabilita,segnalato`: una riga per fotogramma analizzato, tempo in secondi, `segnalato` vale 1 oppure 0;
- `intervalli_<modello>.csv` con intestazione `inizio,fine,colonna,riga,categoria`: una riga per intervallo oscurato, tempi in secondi già allargati, `colonna` da 1 a 4 da sinistra, `riga` da 1 a 3 dall'alto, `categoria` la più frequente nell'intervallo.

Se insieme al video ricevi un file di soluzioni, calcola quanti dati sono rimasti coperti per tutta la loro durata, quanti sono sfuggiti del tutto o in parte, e quanti fotogrammi sono stati segnalati senza motivo. Il file è un JSON di questa forma, con tempi in secondi e rettangoli in pixel alla risoluzione originale del video:

```json
{
  "larghezza": 1920,
  "altezza": 1080,
  "dati": [
    { "inizio": 12.0, "fine": 18.5, "categoria": "email",
      "rettangolo": { "x": 640, "y": 300, "larghezza": 420, "altezza": 36 } }
  ]
}
```

Le immagini vanno inviate come data URL in base64, perché le API non accettano indirizzi remoti.

## Scheda dei modelli

Usa solo la riga del modello indicato in cima.

| Modello | Chiamata | Domande | Risposte | Immagini | Prezzo input |
|---|---|---|---|---|---|
| OpenAI Decisions | `POST https://api.openai.com/v1/decisions`, `model: "gpt-6-luna"`, contesto in `input` | lista di `{type, name, instructions}`; `predicate` per sì/no, `choice` con `choices: [{value, description}]` | `answers[i].probability` oppure `answers[i].choice` e `confidence` | messaggio utente con parti `input_text` e `input_image` | $0.10 per milione di token |
| Jev (TypeSafe) | `POST https://api.typesafe.ai/v1/systemone`, `model: "jev-latest"`, contesto in `state` | oggetto `{nome: {type, instructions, criteria}}`; `noul` per sì/no, `choice` con `criteria: {valore: descrizione}` | `answers.<nome>.noul` oppure `.choice` e `.confidence` | non supportate | $0.042 per milione di token |
| d1 (Liquid AI) | `POST https://api.liquid.ai/decisions/v1/systemone`, `model: "d1"`, contesto in `state` | come Jev | come Jev | elenco `images` accanto a `state` | $0.04 per milione di token |

Autenticazione `Authorization: Bearer <chiave>`; ogni risposta riporta `usage.input_tokens`. Sono API recenti: prima di scrivere le chiamate controlla la documentazione del tuo modello.

- OpenAI Decisions: https://developers.openai.com/api/docs/guides/decisions.md
- Jev (TypeSafe): https://docs.typesafe.ai/api.md
- d1 (Liquid AI): https://docs.liquid.ai/lfm/models/d1.md

## Il resoconto

Al termine genera una pagina HTML, apribile con un doppio clic, con il nome del modello in testa:

- **Foglio:** la tabella originale con le celle oscurate evidenziate e colorate per categoria.
- **Video:** una striscia temporale con i fotogrammi controllati e quelli segnalati in evidenza; cliccando un fotogramma si vedono i riquadri individuati.
- **Numeri, per ciascuna parte:** richieste fatte, celle o riquadri oscurati per categoria, durata totale dell'analisi, tempo mediano per richiesta, token, costo. Se ci sono le soluzioni, anche i risultati del confronto con esse.

Salva inoltre un file `richieste_<modello>.csv` con una riga per ogni richiesta al modello e questa intestazione, esattamente: `modello,parte,elemento,risposta,valore,ms,token_input,costo_usd`. `parte` vale `foglio` oppure `video`; `elemento` dice che cosa è stato giudicato (la riga del foglio, il fotogramma, il riquadro); `valore` è la confidenza o la probabilità; `ms` è il tempo di risposta del modello.

Tutti i file CSV prodotti usano la virgola come separatore e il punto come separatore decimale.

### Avanzamento in tempo reale

Con l'opzione `--eventi <file>` lo strumento scrive il proprio avanzamento in un file di testo, un oggetto JSON per riga, aggiungendo ogni riga appena il fatto accade (senza aspettare la fine), così un'altra applicazione può mostrare il lavoro mentre procede. Il file viene svuotato all'avvio. Gli eventi sono questi, con esattamente questi nomi:

```json
{"evento":"inizio","parte":"foglio","modello":"d1","righe":300,"colonne":["id cliente","nome e cognome"]}
{"evento":"riga","riga":17,"celle":{"email":"email","telefono":"phone"},"ms":231.4,"token":2690,"costo":0.000269}
{"evento":"inizio","parte":"video","modello":"d1","fotogrammi":148,"durata":74.0}
{"evento":"fotogramma","tempo":12.0,"probabilita":0.98,"segnalato":true,"ms":310.2,"token":1124,"costo":0.000112}
{"evento":"riquadro","tempo":12.0,"colonna":2,"riga":1,"categoria":"email","ms":201.7,"token":304,"costo":0.00003}
{"evento":"montaggio"}
{"evento":"fine","parte":"foglio","richieste":300,"durata_analisi_ms":8900,"token":807613,"costo":0.080761,"file":{"copia":"/percorso/assoluto/foglio_oscurato.xlsx"}}
```

- `riga`: un evento per ogni riga del foglio quando tutte le sue richieste sono concluse; `riga` parte da 1, `celle` contiene solo le celle da oscurare con la loro categoria, `ms`, `token` e `costo` sono le somme delle richieste di quella riga.
- `fotogramma` e `riquadro`: un evento per ogni richiesta del primo e del secondo passaggio; `riquadro` va scritto anche quando la categoria è `none`.
- `montaggio`: quando l'analisi del video è finita e comincia la sfocatura.
- `fine`: `durata_analisi_ms` è la durata della sola analisi; in `file` il percorso assoluto della copia oscurata (`copia` per il foglio, `video` per il video).
- `modello` usa lo stesso nome dei file (`openai_decisions`, `jev`, `d1`).

## Misure e vincoli

- Scrivilo in Node.js, con meno dipendenze possibili; per i video usa ffmpeg.
- Il tempo di una richiesta è la sola attesa della risposta del modello. Estrazione dei fotogrammi, ritagli e montaggio finale si cronometrano a parte.
- Costo = `input_tokens × prezzo`; se la risposta riporta `usage.cost`, usa quel valore. Il prezzo sta in un solo punto del codice.
- 8 richieste contemporanee, valore modificabile, con un nuovo tentativo quando l'API risponde che stai andando troppo veloce.
- La chiave sta in un file `.env` (`OPENAI_API_KEY`, `TYPESAFE_API_KEY` o `LIQUID_API_KEY`, secondo il modello) e non compare mai nei file prodotti.
- I file originali non si modificano: tutto ciò che viene prodotto finisce in una cartella `risultati`.

## Prima di consegnare

Esegui lo strumento su un foglio e, se il modello legge le immagini, su un video veri; apri il video oscurato e controlla a occhio che i dati privati siano davvero illeggibili negli intervalli indicati. Riferiscimi cosa hai verificato e cosa no.
