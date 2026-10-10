# jev-decisions-d1-prompts

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](./LICENSE)
[![Decision models](https://img.shields.io/badge/Decision_models-Jev_%7C_OpenAI_Decisions_%7C_d1-10a37f)](#i-tre-prompt)
[![Claude Code](https://img.shields.io/badge/Claude_Code-d97757)](https://docs.anthropic.com/en/docs/claude-code)

**jev-decisions-d1-prompts** è una raccolta di tre prompt che ti permette di replicare il confronto fra i modelli decisionali Jev (TypeSafe), OpenAI Decisions API e d1 (Liquid AI), oppure di riutilizzarli nei tuoi progetti. Ogni prompt fa costruire a un agente di coding un'applicazione completa attorno a un solo modello, con le stesse regole e le stesse misure di tempo e costo per tutti e tre.

---

## I tre prompt

**1. 🏁 [Wikipedia Race](./prompts/prompt-01-wikipedia-race.md)** — un'applicazione web locale in cui il modello parte dalla voce "Gatto" di Wikipedia in italiano e deve raggiungere "Bitcoin" seguendo solo i link interni. A ogni passaggio i link della voce diventano le opzioni di una domanda a scelta.

**2. 🔒 [Anonimizzazione dei dati](./prompts/prompt-02-anonimizzazione-dati.md)** — uno strumento da riga di comando che individua e oscura le informazioni personali in un foglio di calcolo (CSV o XLSX) e in una registrazione dello schermo (MP4), con resoconto HTML e confronto con un file di soluzioni.

**3. 📺 [Filtro YouTube](./prompts/prompt-03-filtro-youtube.md)** — un'estensione Chrome (Manifest V3) che lascia visibili su YouTube soltanto i video che parlano di intelligenza artificiale, giudicando titolo e canale oppure titolo, canale e miniatura.

---

## Prerequisiti

- **Agente di coding** — i prompt sono stati eseguiti con Claude Code; funzionano con qualsiasi agente in grado di scrivere ed eseguire codice.
- **Chiave API del modello** — una per ciascun modello che intendi provare: OpenAI (`OPENAI_API_KEY`), TypeSafe (`TYPESAFE_API_KEY`) e Liquid AI (`LIQUID_API_KEY`).
- **Node.js** — i prompt 1 e 2 producono applicazioni Node.js.
- **ffmpeg** — richiesto dal prompt 2 per estrarre i fotogrammi e sfocare il video.
- **Google Chrome** — richiesto dal prompt 3, che produce un'estensione da caricare come non pacchettizzata.

Jev legge solo testo: nel prompt 2 la parte sul video e nel prompt 3 la modalità con miniatura sono disponibili soltanto con OpenAI Decisions e d1.

---

## Utilizzo

1. Crea una cartella vuota per ogni coppia modello-prompt, così gli output restano separati.
2. Apri il prompt e sostituisci il segnaposto `[SCRIVI QUI UNO DEI TRE: OpenAI Decisions, Jev, d1]` con il nome del modello.
3. Avvia l'agente nella cartella e incolla il testo del prompt senza altre modifiche.
4. Inserisci la chiave API dove indica il prompt: file `.env` per i prompt 1 e 2, pagina delle opzioni dell'estensione per il prompt 3.

Per un confronto valido usa lo stesso prompt per tutti i modelli e cambia solo il segnaposto. Ogni prompt contiene la scheda dei tre modelli con endpoint, formato delle domande e prezzo: l'agente usa soltanto la riga del modello indicato. Le tre API sono recenti, quindi nomi dei modelli e prezzi nella scheda possono cambiare: verifica la documentazione collegata nel prompt.

---

## Risultati del confronto

Valori misurati nelle esecuzioni mostrate nel video, a ottobre 2026.

**Wikipedia Race** — da "Gatto" a "Bitcoin":

- **OpenAI Decisions** — 4 passaggi, 1,42 s di decisione, 0,00069 $.
- **Jev** — 4 passaggi, 1,47 s, 0,00042 $.
- **d1** — 3 passaggi, 1,75 s, 0,00020 $.

**Anonimizzazione del foglio** — 300 righe per 12 colonne:

- **OpenAI Decisions** — 9,6 s, 0,0808 $, accuratezza 99,86%.
- **Jev** — 10,5 s, 0,0355 $, accuratezza 99,92%.
- **d1** — 14,4 s, 0,0359 $, accuratezza 99,78%.

**Anonimizzazione del video** — 148 fotogrammi, senza Jev:

- **OpenAI Decisions** — 30,8 s, 0,0414 $, accuratezza 95%, con circa il 10% di fotogrammi segnalati oltre il necessario.
- **d1** — 37,4 s, 0,0154 $, accuratezza 70%.

**Filtro YouTube** — accordo con il giudizio di Opus 5.5 sugli stessi video:

- **Solo testo, 61 video** — OpenAI Decisions 96,7% (mediana 211 ms), Jev 95,1% (266 ms), d1 94,9% (293 ms).
- **Testo e miniatura, 52 video** — OpenAI Decisions 100%, d1 98%.

**Sintesi** — OpenAI Decisions è risultato il più veloce e, con le immagini, il più accurato, ma anche il più costoso. Jev ha un'accuratezza paragonabile sul testo a meno della metà del prezzo. d1 è il più economico, ma più lento e meno accurato sulle immagini.

---

## Struttura del progetto

```
prompts/
  prompt-01-wikipedia-race.md
  prompt-02-anonimizzazione-dati.md
  prompt-03-filtro-youtube.md
```

---

Designed by **[Dario Fontanel, PhD](https://dariofontanel.com/)**

*Aiuto PMI italiane ad integrare l'intelligenza artificiale per automatizzare i lavori ripetitivi, abbattere i costi e guadagnare tempo per crescere.*

[![Sito](https://img.shields.io/badge/Sito-dariofontanel.com-4285F4?style=flat&logo=googlechrome&logoColor=white)](https://dariofontanel.com/)
[![YouTube](https://img.shields.io/badge/YouTube-FF0000?style=flat&logo=youtube&logoColor=white)](https://www.youtube.com/@dariofontanel)
[![Instagram](https://img.shields.io/badge/Instagram-E4405F?style=flat&logo=instagram&logoColor=white)](https://www.instagram.com/dariofontanel.ai/)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-0A66C2?style=flat&logo=data%3Aimage%2Fsvg%2Bxml%3Bbase64%2CPHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAyNCAyNCI%2BPHBhdGggZmlsbD0id2hpdGUiIGQ9Ik0yMC40NDcgMjAuNDUyaC0zLjU1NHYtNS41NjljMC0xLjMyOC0uMDI3LTMuMDM3LTEuODUyLTMuMDM3LTEuODUzIDAtMi4xMzYgMS40NDUtMi4xMzYgMi45Mzl2NS42NjdIOS4zNTFWOWgzLjQxNHYxLjU2MWguMDQ2Yy40NzctLjkgMS42MzctMS44NSAzLjM3LTEuODUgMy42MDEgMCA0LjI2NyAyLjM3IDQuMjY3IDUuNDU1djYuMjg2ek01LjMzNyA3LjQzM2MtMS4xNDQgMC0yLjA2My0uOTI2LTIuMDYzLTIuMDY1IDAtMS4xMzguOTItMi4wNjMgMi4wNjMtMi4wNjMgMS4xNCAwIDIuMDY0LjkyNSAyLjA2NCAyLjA2MyAwIDEuMTM5LS45MjUgMi4wNjUtMi4wNjQgMi4wNjV6bTEuNzgyIDEzLjAxOUgzLjU1NVY5aDMuNTY0djExLjQ1MnpNMjIuMjI1IDBIMS43NzFDLjc5MiAwIDAgLjc3NCAwIDEuNzI5djIwLjU0MkMwIDIzLjIyNy43OTIgMjQgMS43NzEgMjRoMjAuNDUxQzIzLjIgMjQgMjQgMjMuMjI3IDI0IDIyLjI3MVYxLjcyOUMyNCAuNzc0IDIzLjIgMCAyMi4yMjUgMHoiLz48L3N2Zz4%3D)](https://www.linkedin.com/in/dario-fontanel/)
[![TikTok](https://img.shields.io/badge/TikTok-000000?style=flat&logo=tiktok&logoColor=white)](https://www.tiktok.com/@dario.fontanel)
[![AI Academy](https://img.shields.io/badge/AI_Academy-E7514F?style=flat&logo=data%3Aimage%2Fsvg%2Bxml%3Bbase64%2CPHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAyNCAyNCI%2BPHBhdGggZmlsbD0id2hpdGUiIGQ9Ik0xMiAzIDEgOWwxMSA2IDktNC45MVYxN2gyVjlMMTIgM3pNNSAxMy4xOFYxN2MwIDEuNjYgMy4xMyAzIDcgM3M3LTEuMzQgNy0zdi0zLjgybC03IDMuODItNy0zLjgyeiIvPjwvc3ZnPg%3D%3D)](https://www.skool.com/ai-academy-2306)

Licenza [MIT](./LICENSE).
