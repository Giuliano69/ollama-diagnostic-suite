# ADR-001 — Scopo del progetto
**Status:** Proposed/Accepted/Superseded | **Data:**            | **Autore:**    |
|----------------------------------------|----------------------|----------------|
|Accepted                                |   2026-07-04         | PM             |

## 1. Il Contesto / Problema
Durante sessioni di inferenza locale con modelli LLM di grandi dimensioni (es. Gemma 2/4) eseguiti tramite
Ollama all'interno di un container LXC su Proxmox VE con passthrough della iGPU Intel (UHD 630), si sono
verificati instabilità sistemiche, degradamento prestazionale o blocchi silenziosi associati ad anomalie driver
(es. messaggi del kernel relativi a _sched_reserve_ e limitazioni di memoria).

Risulta necessario uno strumento che automatizzi la raccolta delle informazioni diagnostiche durante l’esecuzione di un test.

Gli strumenti disponibili permettono di osservare singoli aspetti (log, consumo RAM, GPU, processi), ma non forniscono una raccolta coordinata e riproducibile delle informazioni necessarie per analizzare un problema.

Prima di avviare lo sviluppo è stata condotta una verifica di soluzioni esistenti (round 3-4): sono stati esaminati tre progetti di terze parti (Xza85hrf/Ollama_monitor, ysfemreAlbyrk/ollama-monitor,
elbruno/ElBruno.OllamaMonitor). Nessuno copre il caso d'uso: sono strumenti di connectivity/load-testing HTTP o monitor grafici in tempo reale senza persistenza su disco, non fanno raccolta di RSS/smaps, log kernel o GPU.



## 2. Decisione Adottata
Si stabilisce di investire un quantitativo rigidamente circoscritto di tempo nella realizzazione di uno strumento software generalizzato di monitoraggio, isolamento e diagnostica automatizzata denominato Ollama Diagnostic Suite (ODS).


Lo strumento software deve raccogliere automaticamente, registrare ed analizzare informazioni diagnostiche durante l’esecuzione di modelli LLM locali,(inclusi futuri modelli di embedding) eseguiti sia tramite Ollama che tramite backend alternativi (es. llama-server).

La Suite è pensata per restare minimale nelle prime versioni: La versione V0.1 sarà limitata alle funzionalità essenziali di raccolta dati. L’analisi automatica verrà introdotta successivamente.


## 3. Motivazione e Alternative Scartate
Gli strumenti disponibili permettono di osservare singoli aspetti (log, consumo RAM, GPU, processi), ma non forniscono una raccolta coordinata e riproducibile delle informazioni necessarie per analizzare un problema.

- **Fix ad-hoc solo per Gemma4**: scartata, non riusabile per problemi futuri
  con altri modelli (es. modelli di embedding).
- **Adozione di un tool di terze parti**: scartata dopo verifica concreta
  (vedi Contesto) — nessuno dei tre progetti esaminati copre le esigenze
  diagnostiche a livello di processo/kernel/GPU.
- **Analisi manuale** : Scartata perchè l’analisi manuale richiede troppo tempo, è poco riproducibile e rende difficile confrontare test differenti.
- **Utilizzare esclusivamente gli strumenti già presenti nel sistema**: scartata perchè Ogni strumento osserva solamente una parte del sistema; manca una raccolta sincronizzata delle informazioni.

## 4. Conseguenze e vincoli
- Positive
    - progetto semplice da comprendere;
    - tempi di sviluppo ridotti;
    - risultati facilmente riproducibili;
    - possibilità di estendere progressivamente il sistema.
    
- Negative
    - Roadmap incrementale V0.1 → V1.0 (vedi `project_state.md`). La V0.1 non effettuerà diagnosi automatiche;
    - alcune analisi continueranno inizialmente ad essere manuali.

## 5. Stato di Revisione
Questo ADR dovrà essere rivalutato solamente se:
    - gli obiettivi del progetto cambieranno significativamente;
    - ODS evolverà da semplice suite diagnostica a piattaforma di monitoraggio permanente;
    - emergeranno casi d’uso incompatibili con lo scopo attuale.

## 6. Collegamenti
N.D.
